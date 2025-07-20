AWS IoT Core Mastery Roadmap
==========================================

**SECTION 1: Core Concepts & Architecture**

1.  What is AWS IoT Core? (overview, typical use cases)

2.  Key building blocks:

    -   IoT Message Broker (MQTT, HTTPS, WebSockets)

    -   Device Shadow

    -   Rules Engine

    -   Security & Authentication (X.509, IAM, custom authorizers)

    -   Device Registry & Thing Types

3.  How AWS IoT Core fits into real-world solutions (architecture diagrams)

    -   Data flow: Device → AWS IoT Core → AWS services (Lambda, DynamoDB, S3, SNS, etc.)

* * * * *

**SECTION 2: Hands-on Setup & First Device Connection**

1.  Setting up your AWS IoT Core environment (console, CLI)

2.  Registering your first IoT Thing (device) in AWS

3.  Creating and attaching certificates and policies

4.  Connecting a device using MQTT (with sample code and mosquitto CLI)

5.  Using the IoT Test Client to send/receive messages

* * * * *

**SECTION 3: Device Shadow (State Management)**

1.  Understanding Device Shadow (purpose, JSON document structure: desired, reported, delta)

2.  Creating and updating shadows (practical demo using CLI or AWS Console)

3.  Sample code: updating and retrieving shadow state

4.  Shadow use cases and best practices

* * * * *

**SECTION 4: IoT Rules Engine (Data Routing)**

1.  Writing SQL-based IoT Rules (basic and advanced filtering)

2.  Routing messages to:

    -   Lambda (event-driven processing)

    -   DynamoDB (storage)

    -   S3 (archiving)

    -   SNS/SQS (notifications)

3.  Transforming and enriching data in rules

4.  Hands-on: Build a rule that stores high-temp readings in DynamoDB

* * * * *

**SECTION 5: Security Best Practices**

1.  X.509 certificate lifecycle (provisioning, activation, rotation, revocation)

2.  IoT policies vs. IAM policies (and when to use each)

3.  Using custom authorizers (Lambda-based authentication/authorization)

4.  Principle of least privilege for devices

5.  Troubleshooting security (logs, denied actions, certificate issues)

* * * * *

**SECTION 6: Scaling, Provisioning, and Fleet Management**

1.  Device registry and Thing Groups

2.  Bulk provisioning:

    -   Just-In-Time Registration (JITR)

    -   Just-In-Time Provisioning (JITP)

    -   Fleet provisioning templates

3.  Device Management and Device Defender (monitoring, audit, alerting)

* * * * *

**SECTION 7: Monitoring, Logging, and Troubleshooting**

1.  CloudWatch metrics and logs for IoT Core

2.  Enabling detailed logs for message broker and rules

3.  Debugging common issues (connection problems, message loss, rule failures)

4.  Device Defender for fleet-wide security monitoring

* * * * *

**SECTION 8: Real-World Mini-Projects & Use Cases**

1.  Project 1: Temperature Sensor Data Pipeline (IoT Core → Lambda → DynamoDB)

2.  Project 2: Remote Device Control (update device shadow via web app)

3.  Project 3: Scalable onboarding with JITP

4.  Project 4: Custom authentication with Lambda authorizer

* * * * *

**SECTION 9: Exam-Style Practice & Memory Boosters**

1.  Multiple-choice and scenario-based questions after each topic

2.  Immediate explanations for correct and incorrect answers

3.  Quick "power-up" exam tips and gotchas to watch for

* * * * *

**SECTION 10: Review & Mastery Checklist**

1.  Summary notes for each core topic

2.  Personal checklist: can you explain, configure, and troubleshoot each component?

3.  Mock exam session: 10--20 questions with full breakdown


---



### AWS IoT Core messaging services

The AWS IoT Core connectivity services provide secure communication with the IoT devices and manage the messages that pass between them and AWS IoT.

### Device gateway

Enables devices to securely and efficiently communicate with AWS IoT. Device communication is secured by secure protocols that use X.509 certificates.

### Message broker

Provides a secure mechanism for devices and AWS IoT applications to publish and receive messages from each other. You can use either the MQTT protocol directly or MQTT over WebSocket to publish and subscribe. For more information about the protocols that AWS IoT supports, see [Device communication protocols](https://docs.aws.amazon.com/iot/latest/developerguide/protocols.html). Devices and clients can also use the HTTP REST interface to publish data to the message broker.

The message broker distributes device data to devices that have subscribed to it and to other AWS IoT Core services, such as the Device Shadow service and the rules engine.

### AWS IoT Core for LoRaWAN

AWS IoT Core for LoRaWAN makes it possible to set up a private LoRaWAN network by connecting your LoRaWAN devices and gateways to AWS without the need to develop and operate a LoRaWAN Network Server (LNS). Messages received from LoRaWAN devices are sent to the rules engine where they can be formatted and sent to other AWS IoT services.

### Rules engine

The Rules engine connects data from the message broker to other AWS IoT services for storage and additional processing. For example, you can insert, update, or query a DynamoDB table or invoke a Lambda function based on an expression that you defined in the Rules engine. You can use an SQL-based language to select data from message payloads, and then process and send the data to other services, such as Amazon Simple Storage Service (Amazon S3), Amazon DynamoDB, and AWS Lambda. You can also create rules that republish messages to the message broker and on to other subscribers. For more information, see [Rules for AWS IoT](https://docs.aws.amazon.com/iot/latest/developerguide/iot-rules.html).



Great! Let’s begin with **Section 1: Core Concepts & Architecture** for AWS IoT Core.

---

### 1. What is AWS IoT Core?

**AWS IoT Core** is a managed cloud service that lets you connect devices to AWS securely and easily. It handles real-time message exchange, state tracking, data routing, and device management for massive IoT fleets, without the need to manage infrastructure yourself.

**Key Purposes:**

* Securely connect millions or billions of devices.
* Route device data to AWS services for storage, processing, and analytics.
* Control and update device state remotely (even when the device is offline).
* Build scalable, reliable IoT applications—like sensor monitoring, industrial controls, or smart home solutions.

---

### 2. Key Building Blocks

Let’s briefly introduce the main parts of AWS IoT Core. We’ll dive deeper into each later.

1. **IoT Message Broker**

   * Supports MQTT (main protocol), HTTP, and WebSockets.
   * Acts as the central hub for all device messages.
   * Uses **topics** for publishing and subscribing.

2. **Device Shadow**

   * A virtual, persistent JSON document in the cloud representing each device’s last known and desired state.
   * Lets you interact with devices even if they’re offline.

3. **Rules Engine**

   * Lets you filter, transform, and route data from devices to 20+ AWS services (e.g., Lambda, DynamoDB, S3).
   * Uses SQL-like syntax for message filtering and selection.

4. **Security & Authentication**

   * Uses X.509 certificates for mutual authentication.
   * Supports IAM policies and Lambda-based custom authorizers.
   * Fine-grained permissions for secure device access and actions.

5. **Device Registry & Thing Types**

   * Organize and manage your device fleet.
   * Attach metadata to “things” for easier management and automation.

---

### 3. Real-World Architecture Diagram

Here’s a classic AWS IoT Core data flow (imagine a fleet of temperature sensors):

```
[Temperature Sensor Devices]
        │
   (MQTT, HTTPS)
        │
        ▼
[IoT Core Message Broker] ──> [Rules Engine] ──> [AWS Service: Lambda, DynamoDB, S3, SNS, etc.]
        │
        ▼
[Device Shadow (state)]
```

* Data flows from the device to the message broker, then to other AWS services via the Rules Engine.
* Device shadow keeps track of state for remote control and status queries.

---

### Practical Example

Suppose you want to:

* Connect a Raspberry Pi as a temperature sensor.
* Send JSON data (`{"temp":28.7, "humidity":54}`) every minute.
* Route readings >30°C to a DynamoDB table for monitoring.

You would:

* Register your Pi as a “thing” in AWS IoT.
* Attach a certificate and policy.
* Set up an IoT Rule with SQL to filter high-temp readings and send them to DynamoDB.
* Optionally, use the Device Shadow to track the Pi’s current status (e.g., online/offline, last reading).

---

### Hands-on (Optional for Now)

If you want to try this yourself:

1. Go to AWS IoT Core Console.
2. Explore the “Manage” (Things), “Test” (MQTT), and “Act” (Rules) sections.
3. Try publishing a message using the MQTT Test Client to the topic `sensors/test/data`.

---

### Practice Question #1

Which protocol is most commonly used by devices to send and receive messages with AWS IoT Core?

A) SMTP
B) MQTT
C) FTP
D) SSH

What’s your answer? (Pick A, B, C, or D—then I’ll explain and move to the next concept.)

Answer: B
---

Awesome! First, here’s the answer to the previous MCQ:

**Question:**
Which protocol is most commonly used by devices to send and receive messages with AWS IoT Core?
A) SMTP
B) MQTT
C) FTP
D) SSH

**Correct Answer:** **B) MQTT**

**Explanation:**
MQTT (Message Queuing Telemetry Transport) is the most common and recommended protocol for device communication in AWS IoT Core. It’s lightweight, ideal for devices with limited resources, and supports publish/subscribe patterns. The other options (SMTP, FTP, SSH) are not used for IoT messaging.

**Exam Tip:**
If you see MQTT as an option for IoT Core device messaging, it’s almost always correct for DVA-C02!

---

## Next Concept: IoT Message Broker in Detail

### What is the Message Broker?

The **Message Broker** in AWS IoT Core is the managed service that enables devices and cloud applications to communicate using **topics**.

* Supports **MQTT** (main protocol), **HTTP**, and **WebSockets**.
* Devices “publish” messages to topics (e.g., `sensors/raspi-01/data`).
* Other devices/services “subscribe” to those topics to receive messages.

**Key points:**

* **Topics** are just named channels, like “chat rooms” for device data.
* Supports wildcards in topics (e.g., `sensors/+/data` subscribes to all devices).
* Highly scalable—no need to manage your own broker server!

#### Example

* Device publishes:
  Topic: `home/thermostat/temp`
  Message: `{ "temp": 27.3 }`

* Cloud app subscribes to `home/thermostat/temp` and gets the latest reading.

#### MQTT Quality of Service (QoS) Levels

* **QoS 0**: At most once delivery (fire and forget)
* **QoS 1**: At least once delivery (acknowledged, may be delivered more than once)
* **QoS 2**: *NOT supported by AWS IoT Core*

**Exam Trick:** If a question asks about QoS 2, remember it’s **not supported**.

---

### Mini-Exercise (Optional)

**Try:**

* Use the AWS IoT Console “Test” tab to subscribe to a topic (e.g., `test/topic`).
* In another tab, publish a message to that topic.
* Watch the message appear instantly.

---

### so are topics dedicated to devices?

Great question, Rahman! Here’s a clear answer:

**Topics in AWS IoT Core are NOT dedicated to individual devices by default.**

### How topics work in AWS IoT Core:

* **Topics** are just named channels (like folders or paths) that devices can publish or subscribe to.
* You design your topic structure however you like, and it often makes sense to use a device ID or type in the topic name for clarity.

#### Common Patterns:

* **Device-specific topics:**
  Example: `sensors/device123/data`
  Only device123 publishes or subscribes here.
* **Group/broadcast topics:**
  Example: `sensors/all/devices`
  Many devices can publish or subscribe here for group-wide messages.
* **Wildcard topics:**
  Use `+` or `#` to subscribe to patterns:

  * `sensors/+/data` subscribes to all devices’ data under `sensors/`
  * `sensors/#` subscribes to everything under `sensors/`

#### Best Practice:

* Most IoT architectures use device-specific topics for data uplink (device to cloud), so you can manage access, security, and filtering per device.
* But, you can create topics for commands, broadcasts, or grouped devices as needed.

---

**Short answer:**

* Topics are not “dedicated” by AWS.
* *You* define how devices use them—device-specific, group, or broadcast—based on your application’s needs.

---

### Example

| Device  | Publishes to topic   | Subscribes to topic      |
| ------- | -------------------- | ------------------------ |
| deviceA | sensors/deviceA/data | sensors/deviceA/commands |
| deviceB | sensors/deviceB/data | sensors/deviceB/commands |

Cloud apps can subscribe to `sensors/+/data` to get data from all devices.

---

## are topics unique, like what if two AWS users has the same topic?

Excellent question, Rahman! This is important for both the exam and real-world work.

### Are Topics Unique in AWS IoT Core?

**Topics are only unique within your own AWS account’s IoT Core service.**

* **Every AWS account’s IoT Core is logically separate from every other AWS account.**
* You and another AWS user can both have a topic called `sensors/device123/data`—but your topics, devices, and messages are completely isolated and can never interact with each other.
* There is **no global “topic registry”**—topics are just strings used for routing inside your AWS account’s IoT Core.

---

#### Scenario

Suppose:

* You, in your AWS account, have a device publishing to `sensors/mydevice/data`.
* Someone else, in their AWS account, uses exactly the same topic name.

**Result:**

* Your device messages only exist and are visible in your AWS account’s IoT Core.
* Their device messages only exist and are visible in their account.
* There is no overlap or conflict; topics do not collide between accounts.

---

#### Security & Access

* **Devices, users, and apps can only access topics in the AWS account where their certificates/policies are registered.**
* If you want cross-account IoT data sharing, you must set up explicit integrations (for example, with cross-account SNS, S3, Lambda, or IoT rules).

---

#### Exam Tip

If an exam question mentions multiple AWS users/devices with the same topic name, always assume **logical isolation per AWS account**. There is no way for devices in different AWS accounts to interact via topics unless you set up specific cross-account sharing.

---

**Summary:**

* Topics are not globally unique—just unique within each AWS account’s IoT Core.
* No risk of collision or message leakage across AWS accounts.

---

## What is MQTT?

**MQTT** stands for **Message Queuing Telemetry Transport**.
It’s a lightweight, publish-subscribe network protocol designed for fast, reliable messaging between devices—especially those with limited resources or unreliable connections (e.g., sensors, IoT devices, embedded systems).

* **Why MQTT?**

  * Low bandwidth use
  * Reliable in spotty networks
  * Simple to implement
  * Designed for IoT

---

### Core Concepts

1. **Broker**

   * Central server that receives all messages, then distributes them to interested clients (subscribers).
   * In AWS IoT Core, the “Message Broker” is fully managed.

2. **Client**

   * Anything that connects to the broker: device, application, or cloud service.
   * Each client can **publish** (send) and/or **subscribe** (receive) messages.

3. **Topic**

   * Hierarchical UTF-8 strings, used as routing keys (e.g., `home/livingroom/temp`).
   * Clients subscribe to topics to receive relevant messages.

4. **Publish/Subscribe**

   * **Publish:** Client sends message to a topic (`publish` action).
   * **Subscribe:** Client listens for messages on one or more topics.

5. **Quality of Service (QoS)**

   * **QoS 0:** At most once (fire-and-forget; no acknowledgment).
   * **QoS 1:** At least once (acknowledged; can be delivered multiple times).
   * **QoS 2:** Exactly once (not supported by AWS IoT Core).

6. **Retained Messages**

   * Broker saves the last message sent to a topic and delivers it to new subscribers. (AWS IoT Core does **not** support retained messages.)

---

### How MQTT Works (Step by Step)

**Example scenario:**

* Three temperature sensors: `sensor1`, `sensor2`, `sensor3`
* All publish temperature to topic `factory/temperature`

#### 1. Devices connect to the broker (AWS IoT endpoint) via MQTT.

#### 2. Each device publishes:

```json
// sensor1 publishes
Topic: factory/temperature
Payload: { "deviceId": "sensor1", "temp": 24.1 }

// sensor2 publishes
Topic: factory/temperature
Payload: { "deviceId": "sensor2", "temp": 25.5 }
```

#### 3. Application subscribes to `factory/temperature`:

* It receives all messages sent by all sensors on that topic.

---

#### Topic Structure Example

* **Device-specific:** `sensors/sensor1/data`
* **Grouped:** `sensors/+/data` (wildcard: matches all sensors)
* **All:** `sensors/#` (matches all topics starting with `sensors/`)

**Wildcard rules:**

* `+` matches exactly one level: `sensors/+/data` matches `sensors/sensor1/data` and `sensors/sensor2/data`
* `#` matches all levels after: `sensors/#` matches `sensors/sensor1/data`, `sensors/sensor1/alerts`, etc.

---

### Real-World Use Cases

#### 1. Smart Home

* Sensors publish temperature, humidity, motion, door status to topics like `home/livingroom/temp`.
* Mobile apps subscribe to these topics to update dashboards or trigger automation.

#### 2. Industrial Monitoring

* Machines publish their health/status to topics like `plant/line1/motor3/status`.
* Central system monitors for faults, triggers alerts on `plant/+/+/status` topics.

#### 3. Asset Tracking

* GPS devices publish location updates to `vehicles/{vehicleId}/location`.
* Tracking dashboard subscribes to all location topics.

#### 4. Remote Control

* Devices subscribe to a command topic (e.g., `devices/device123/cmd`).
* Cloud app publishes commands (e.g., "turn on") to specific device topics.

---

### MQTT Example with AWS IoT Core (Command Line)

**Publishing a message (using mosquitto\_pub):**

```bash
mosquitto_pub \
  -h <AWS_IOT_ENDPOINT> \
  -p 8883 \
  --cafile AmazonRootCA1.pem \
  --cert device.pem.crt \
  --key private.pem.key \
  -t sensors/sensor1/data \
  -m '{"temp": 22.5, "humidity": 60}'
```

**Subscribing to a topic (using mosquitto\_sub):**

```bash
mosquitto_sub \
  -h <AWS_IOT_ENDPOINT> \
  -p 8883 \
  --cafile AmazonRootCA1.pem \
  --cert device.pem.crt \
  --key private.pem.key \
  -t sensors/+/data
```

This will print any messages published by any sensor on the `sensors/+/data` topics.

---

### Exam Tips and Common Mistakes

* **AWS IoT Core supports MQTT over TLS (port 8883), MQTT over WebSockets, and HTTP.**
* **QoS 2 (exactly once) is NOT supported by AWS IoT Core—watch for this in tricky exam questions.**
* **Retained messages** are NOT supported.
* Topics are case-sensitive and must match exactly unless wildcards are used.
* Only authenticated and authorized clients can publish/subscribe (enforced by IoT policies and certificates).

---

### Quick Diagram

```
[Sensor Devices]
     |
(Publish to topics)
     v
 [AWS IoT MQTT Broker]
     |
(Subscribe to topics)
     v
[Cloud App, Lambda, Rules Engine, etc.]
```

---

### Memory Power-Up

* MQTT = **M**inimal, **Q**uick, **T**ransfer for IoT!
* If it’s a small, unreliable device or a huge fleet—**MQTT** is the answer.
* “Broker, Topic, QoS, Wildcard” = 4 MQTT pillars for AWS IoT exam questions.

---

