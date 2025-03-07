import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.ConsumerRecords;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.common.TopicPartition;
import org.apache.kafka.common.header.Header;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

public class KafkaTopicBrowser {

    public static void main(String[] args) {
        final String topic = "TopicA";          // your topic name
        final int totalPartitions = 100;          // total partitions in the topic

        // Set up consumer properties
        Properties props = new Properties();
        props.put("bootstrap.servers", "localhost:9092"); // change as needed
        // Even though we're using assign(), Kafka still requires a group id.
        props.put("group.id", "temporary-group-" + System.currentTimeMillis());
        props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
        // Start at the beginning if no offset is available (though we explicitly seek to beginning)
        props.put("auto.offset.reset", "earliest");
        // Disable auto commit since we're manually controlling offsets
        props.put("enable.auto.commit", "false");

        // For each partition, create and start a consumer thread
        for (int partition = 0; partition < totalPartitions; partition++) {
            Thread consumerThread = new Thread(new PartitionBrowser(topic, partition, props));
            consumerThread.setName("Consumer-Partition-" + partition);
            consumerThread.start();
        }
    }
}

class PartitionBrowser implements Runnable {
    private final String topic;
    private final int partition;
    private final Properties props;

    public PartitionBrowser(String topic, int partition, Properties props) {
        this.topic = topic;
        this.partition = partition;
        this.props = props;
    }

    @Override
    public void run() {
        // Create a KafkaConsumer instance for this partition
        try (KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props)) {
            // Create a TopicPartition object for the specified partition
            TopicPartition tp = new TopicPartition(topic, partition);
            // Directly assign the consumer to this single partition (bypassing subscribe and group coordination)
            consumer.assign(Collections.singleton(tp));
            // Seek to the beginning so we can "browse" all messages
            consumer.seekToBeginning(Collections.singleton(tp));

            System.out.println("Started consumer for partition " + partition);

            // Continuously poll for records
            while (true) {
                ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(1000));
                for (ConsumerRecord<String, String> record : records) {
                    System.out.println("Partition: " + partition + ", offset: " + record.offset());
                    // Process only headers (ignoring the payload)
                    for (Header header : record.headers()) {
                        String headerKey = header.key();
                        String headerValue = new String(header.value());
                        System.out.println("  Header: " + headerKey + " = " + headerValue);
                    }
                }
                // Optionally: add logic to break out of the loop after a certain condition
            }
        } catch (Exception e) {
            System.err.println("Error in consumer for partition " + partition);
            e.printStackTrace();
        }
    }
}
