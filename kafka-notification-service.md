## 🚀 **Kafka Topics & Event Design for OMS Notifications**

### 🎯 **Primary Flow**

Every significant action in the OMS emits an event to Kafka, consumed by your Notification Service for delivery.

---

### 📌 **Example Kafka Topics**

You might structure Kafka topics like this:

| Kafka Topic                | Description                                                                    |
| -------------------------- | ------------------------------------------------------------------------------ |
| `order.created`            | Fired when a new order is placed.                                              |
| `order.approval.requested` | Fired when an order moves to the next approval level (e.g., manager, finance). |
| `order.approval.completed` | Fired when approval is granted at a specific level.                            |
| `order.approval.rejected`  | Fired when an approval is denied.                                              |
| `order.cancelled`          | Fired when an order is cancelled.                                              |
| `order.fulfilled`          | Fired when an order is fulfilled or shipped.                                   |
| `order.failed`             | Fired on processing failure, to trigger compensating actions or alerts.        |

---

### 📌 **Kafka Message Payload Example**

Each event message sent to Kafka could look like:

```json
{
  "orderId": "ORD12345",
  "eventType": "ORDER_APPROVAL_REQUESTED",
  "timestamp": "2025-06-17T10:30:00Z",
  "triggeredBy": {
    "userId": "USER789",
    "role": "SALES_EXECUTIVE"
  },
  "approvalLevel": "MANAGER",
  "notifiers": [
    { "userId": "MGR001", "role": "MANAGER" },
    { "userId": "ADMIN001", "role": "ADMIN" }
  ],
  "message": "Order ORD12345 requires your approval."
}
```

👉 This payload enables the **notification consumer** to build customized messages for Firebase / email / SMS, depending on recipient and role.

---

### 📌 **How the Notification Service Uses Kafka**

✅ **Producers**:

* Your **OMS service** publishes events at key points (create, approval requested, approved, rejected, cancelled, fulfilled).

✅ **Kafka as the broker**:

* Topics partitioned (if needed) by order ID hash to scale across consumers.
* Guarantees ordering within a partition so no approval stage is notified before the previous one.

✅ **Consumers (Notification Service)**:

* Subscribes to these topics.
* Applies logic like:

  * Send FCM push to **customer users** on `order.created`, `order.fulfilled`, `order.cancelled`.
  * Notify **approvers** on `order.approval.requested`.
  * Notify **admin + order creator** on `order.approval.completed` or `order.approval.rejected`.

✅ **Additional Features**:

* Can batch low-priority notifications (e.g., periodic summaries to admins).
* Can support retry queues for failed deliveries (e.g., `order.notifications.retry`).

---

## 📝 **Resume-ready Bullet Example**

> Designed a Kafka-based event-driven notification framework for an Order Management System, publishing domain events (e.g., `order.created`, `order.approval.requested`, `order.fulfilled`) and consuming them for real-time user/admin notifications via Firebase, achieving scalable decoupling between business logic and delivery channels.

---

## 💡 **Architecture Diagram (Described in words)**

👉 Imagine this:

```
[ OMS Service ] 
  └── emits events → [ Kafka Topics ]
                        └── consumed by → [ Notification Service ]
                                                ├── Push to FCM
                                                ├── Send Email/SMS
                                                └── Log & Retry if failed
```

---

✅ **Awesome! Let’s do this cleanly so you can use it in your project or resume.**

---

# 🚀 **Kafka Producer + Consumer for OMS Notifications**

We’ll use **Node.js (kafkajs)** for the example.

---

## 1️⃣ **Kafka Topic Naming + Partitioning Convention**

👉 **Topic naming pattern**:

```
order.<event_type>
```

Examples:

* `order.created`
* `order.approval.requested`
* `order.approval.completed`
* `order.approval.rejected`
* `order.cancelled`
* `order.fulfilled`

👉 **Partitioning strategy**:

* Partition key: `orderId`
* Ensures all events of the same order go to the same partition → guarantees ordering.

---

## 2️⃣ **Kafka Producer Example**

```javascript
const { Kafka } = require('kafkajs');

const kafka = new Kafka({
  clientId: 'oms-service',
  brokers: ['localhost:9092'],
});

const producer = kafka.producer();

async function sendOrderEvent(orderId, eventType, payload) {
  await producer.connect();

  const topic = `order.${eventType.toLowerCase()}`;
  
  await producer.send({
    topic,
    messages: [
      {
        key: orderId, // for partitioning
        value: JSON.stringify({
          orderId,
          eventType,
          timestamp: new Date().toISOString(),
          ...payload,
        }),
      },
    ],
  });

  console.log(`Event sent: ${topic} for order ${orderId}`);
  
  await producer.disconnect();
}

// Example usage:
sendOrderEvent('ORD12345', 'CREATED', {
  triggeredBy: { userId: 'USER789', role: 'SALES_EXECUTIVE' },
  message: 'Order created successfully',
});
```

👉 **Tip:** In production, keep the producer connection open rather than connect/disconnect on each send.

---

## 3️⃣ **Kafka Consumer Example (Notification Service)**

```javascript
const kafka = new Kafka({
  clientId: 'notification-service',
  brokers: ['localhost:9092'],
});

const consumer = kafka.consumer({ groupId: 'notification-group' });

async function runConsumer() {
  await consumer.connect();
  
  // Subscribe to wildcard topics
  await consumer.subscribe({ topic: /^order\..*/, fromBeginning: false });

  await consumer.run({
    eachMessage: async ({ topic, partition, message }) => {
      const event = JSON.parse(message.value.toString());
      console.log(`Received ${topic} event for Order: ${event.orderId}`);

      switch (topic) {
        case 'order.created':
          // Notify customer users
          notifyUser(event, 'customer');
          break;
        case 'order.approval.requested':
          // Notify approvers
          notifyUser(event, 'approver');
          break;
        case 'order.approval.completed':
        case 'order.approval.rejected':
          // Notify admin + creator
          notifyUser(event, 'admin');
          break;
        case 'order.fulfilled':
        case 'order.cancelled':
          notifyUser(event, 'customer');
          break;
        default:
          console.warn(`Unhandled topic: ${topic}`);
      }
    },
  });
}

function notifyUser(event, type) {
  // This is where you'd trigger Firebase / Email / SMS
  console.log(`🔔 Notifying ${type}:`, event.message);
}

// Start consumer
runConsumer();
```

---

## 4️⃣ **Partitions & Key Design**

👉 **Why partition by orderId?**

```plaintext
- Guarantees that all events of one order go to the same partition.
- Allows ordered processing: approval requested → approved → fulfilled.
- Helps in scaling: if you have 10 partitions, orders will auto-distribute.
```

Example:

```javascript
producer.send({
  topic: 'order.created',
  messages: [
    { key: orderId, value: JSON.stringify(payload) }
  ]
});
```

---

## 📝 **Resume-ready line**

> Built Kafka producers/consumers with `order.<event_type>` topic naming and `orderId`-based partitioning to ensure ordered processing of approval workflows in an Order Management System, enabling real-time, scalable notifications to customers, admins, and approvers.

---

