# แบบฝึกหัดที่ 13: การใช้งาน Cloud Pub/Sub
# Exercise 13: Using Cloud Pub/Sub

## โจทย์ | Problem
พัฒนาระบบ event-driven โดยใช้ Cloud Pub/Sub โดยมีความต้องการดังนี้:

1. สร้าง:
   - Topics และ subscriptions
   - Dead letter topics
   - Push และ pull subscriptions
   - Message filtering

2. จัดการ:
   - Message retention
   - Delivery retry
   - Monitoring
   - Access control

## เฉลย | Solution

### 1. สร้าง Topics และ Subscriptions:
```bash
# Create main topic
gcloud pubsub topics create orders-topic

# Create dead letter topic
gcloud pubsub topics create orders-dlq

# Create pull subscription
gcloud pubsub subscriptions create orders-sub \
    --topic=orders-topic \
    --message-retention-duration=7d \
    --ack-deadline=60 \
    --dead-letter-topic=orders-dlq \
    --max-delivery-attempts=5

# Create push subscription
gcloud pubsub subscriptions create orders-push-sub \
    --topic=orders-topic \
    --push-endpoint=https://api.example.com/orders \
    --message-retention-duration=7d \
    --ack-deadline=60
```

### 2. Python Publisher Code (publisher.py):
```python
from google.cloud import pubsub_v1
import json
import time

class OrderPublisher:
    def __init__(self, project_id, topic_id):
        self.publisher = pubsub_v1.PublisherClient()
        self.topic_path = self.publisher.topic_path(project_id, topic_id)

    def publish_order(self, order_data):
        data = json.dumps(order_data).encode('utf-8')
        
        # Add attributes for filtering
        attributes = {
            'order_type': order_data.get('type', 'standard'),
            'priority': str(order_data.get('priority', 'normal'))
        }
        
        future = self.publisher.publish(
            self.topic_path, 
            data,
            **attributes
        )
        
        try:
            message_id = future.result()
            print(f"Published message ID: {message_id}")
            return message_id
        except Exception as e:
            print(f"Error publishing message: {e}")
            raise

    def publish_batch(self, orders, batch_size=100):
        batch = self.publisher.batch_settings(
            max_messages=batch_size,
            max_bytes=1024 * 1024  # 1MB
        )
        
        with batch:
            for order in orders:
                data = json.dumps(order).encode('utf-8')
                self.publisher.publish(self.topic_path, data)
```

### 3. Python Subscriber Code (subscriber.py):
```python
from google.cloud import pubsub_v1
import json
import time
from concurrent.futures import TimeoutError

class OrderSubscriber:
    def __init__(self, project_id, subscription_id):
        self.subscriber = pubsub_v1.SubscriberClient()
        self.subscription_path = self.subscriber.subscription_path(
            project_id, 
            subscription_id
        )
        self.streaming_pull_future = None

    def callback(self, message):
        try:
            data = json.loads(message.data.decode('utf-8'))
            print(f"Received order: {data}")
            print(f"Attributes: {message.attributes}")
            
            # Process the order
            self.process_order(data)
            
            # Acknowledge the message
            message.ack()
            
        except Exception as e:
            print(f"Error processing message: {e}")
            # Nack the message to retry
            message.nack()

    def process_order(self, order_data):
        # Add your order processing logic here
        time.sleep(1)  # Simulate processing
        print(f"Processed order: {order_data}")

    def start_receiving(self, timeout=None):
        streaming_pull_future = self.subscriber.subscribe(
            self.subscription_path,
            callback=self.callback
        )
        print(f"Listening for messages on {self.subscription_path}")
        
        try:
            streaming_pull_future.result(timeout=timeout)
        except TimeoutError:
            streaming_pull_future.cancel()
            streaming_pull_future.result()
        except Exception as e:
            streaming_pull_future.cancel()
            print(f"Error in subscription: {e}")
```

### 4. สร้าง Message Filter:
```bash
# Create filtered subscription
gcloud pubsub subscriptions create high-priority-sub \
    --topic=orders-topic \
    --filter="attributes.priority = 'high'"

gcloud pubsub subscriptions create international-orders-sub \
    --topic=orders-topic \
    --filter="attributes.order_type = 'international'"
```

### 5. ตั้งค่า IAM:
```bash
# Create service account
gcloud iam service-accounts create pubsub-service \
    --display-name="Pub/Sub Service Account"

# Grant publisher permissions
gcloud pubsub topics add-iam-policy-binding orders-topic \
    --member="serviceAccount:pubsub-service@$PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/pubsub.publisher"

# Grant subscriber permissions
gcloud pubsub subscriptions add-iam-policy-binding orders-sub \
    --member="serviceAccount:pubsub-service@$PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/pubsub.subscriber"
```

### 6. ตั้งค่า Monitoring:
```bash
# Create alert for undelivered messages
gcloud alpha monitoring policies create \
    --display-name="Undelivered Messages Alert" \
    --condition-display-name="High undelivered count" \
    --condition-filter='resource.type="pubsub_subscription" AND metric.type="pubsub.googleapis.com/subscription/num_undelivered_messages"' \
    --condition-threshold-value=1000 \
    --condition-threshold-duration=300s

# Create alert for publish latency
gcloud alpha monitoring policies create \
    --display-name="Publish Latency Alert" \
    --condition-display-name="High publish latency" \
    --condition-filter='resource.type="pubsub_topic" AND metric.type="pubsub.googleapis.com/topic/publish_latency"' \
    --condition-threshold-value=1000 \
    --condition-threshold-duration=300s
```

## การทดสอบ | Testing
```python
# Test publisher
publisher = OrderPublisher('your-project-id', 'orders-topic')

# Publish single message
order = {
    'order_id': '12345',
    'type': 'international',
    'priority': 'high',
    'items': ['item1', 'item2']
}
publisher.publish_order(order)

# Publish batch
orders = [
    {'order_id': str(i), 'type': 'standard'} 
    for i in range(100)
]
publisher.publish_batch(orders)

# Test subscriber
subscriber = OrderSubscriber('your-project-id', 'orders-sub')
subscriber.start_receiving(timeout=30)

# Test message filtering
high_priority_subscriber = OrderSubscriber(
    'your-project-id', 
    'high-priority-sub'
)
high_priority_subscriber.start_receiving(timeout=30)
```

## เพิ่มเติม | Additional Notes
- ใช้ exponential backoff สำหรับ retries
- ตั้งค่า monitoring สำหรับ dead letter queue
- ใช้ ordering keys สำหรับ ordered delivery
- พิจารณาใช้ Cloud Functions สำหรับ event processing
- ทำ load testing ก่อน production deployment
