📌 Microservices – Clean Notes
🔹 What is Microservices Architecture?

Microservices architecture is a way of building applications as a collection of small, independent services.

Key Characteristics

Each service is independent

Each service has one clear responsibility

Each service exposes its own API

Services communicate using APIs or events

One service can call another service’s API

Services can be developed, deployed, and scaled independently

🔹 Example Service Flow
Auth Service → User Service → Order Service


Each service does only its own job and communicates with others when needed.

🔹 Example: Service-to-Service Communication
✅ User Service
// GET /users/1
app.get("/users/:id", (req, res) => {
  res.json({ id: 1, name: "Abhishek" });
});

✅ Order Service (Calling User Service)
const axios = require("axios");

app.get("/orders/:id", async (req, res) => {
  const user = await axios.get("http://user-service/users/1");

  res.json({
    orderId: 101,
    user: user.data
  });
});


📌 Order Service does NOT access User DB directly
It communicates only via User Service API.

🔹 How Microservices Communicate
1️⃣ Synchronous Communication

REST (HTTP)

gRPC

Caller waits for response

Service A → Service B → Response

2️⃣ Asynchronous Communication

Message Queues (Kafka, RabbitMQ)

Event-driven

Caller does NOT wait for response

Service A → Event → Service B

🔹 Typical Request Flow
Frontend
   ↓
API Gateway
   ↓
Service A → Service B → Service C

🔹 Handling Timeouts in Microservices
Example Scenario

Search Service → returns blog posts

Analytics Service → returns likes, comments, views

When a user searches a blog:

Search Service → Analytics Service

🔹 Possible Failure Cases

Request never reaches Analytics Service

Analytics Service response times out

Analytics Service is slow due to heavy computation

🔹 How to Handle These Failures
✅ 1. Default / Fallback Values

If Analytics Service fails:

{
  "likes": 0,
  "comments": 0
}


UI continues working with partial data.

✅ 2. Retry with Exponential Backoff

Retry only when required, with increasing delays:

2s → 4s → 8s → 16s


📌 Avoid infinite retries

⚠️ Retry Concerns

Request may be non-idempotent

Duplicate operations may occur

Downstream services may already be overloaded

Example

💥 Payment of ₹10 executed twice

✅ 3. Retry Only Idempotent Requests
Request Type	Safe to Retry
GET	✅ Yes
POST (payment/order)	❌ No

📌 Example Problem:
Retrying POST /create-post → Duplicate posts created

✅ 4. Graceful Degradation

Ignore Analytics Service failure

Show partial data instead of breaking UI

📌 Better user experience than full failure

### Sharing Database in Microservices
# What is Shared Database?
Multiple microservices use the same database

User Service  ┐
Order Service ├──► Shared Database
Payment Service┘

## Advantages of Sharing Database
# Easy to Implement
No inter-service API calls

Example:
Order Service directly reads users table
(No call to User Service)

# Faster Initial Development
Simple architecture

Suitable for MVPs

Example:
Startup with 3 services and 1 shared DB

### Disadvantages of Sharing Database
# Tight Coupling (Biggest Problem)
Schema changes affect all services

Example:
User Service renames:

email → user_email
Order Service breaks ❌

# Scalability Issues
All services hit the same DB

Example:
High traffic on Order Service slows User Service

# Technology Lock-in
All services must use same DB type

Example:
User Service → MongoDB
Order Service → PostgreSQL
❌ Not possible with shared DB

# Data Ownership Confusion
No clear table owner

Example:
Who owns users table?
User Service?
Order Service?
Both? 😕
Leads to bugs and conflicts

⭐ Best Practice (Recommended)
❌ Shared Database
✅ Database per Microservice

User Service  → User DB
Order Service → Order DB
Payment Service → Payment DB
Services communicate via APIs or Events


# 📌 Microservices Communication – Complete Notes

---

## 🔹 How Microservices Communicate

In microservices architecture, services are **independent and **do not share memory or code**.  
They communicate using **network-based communication**.

### Communication happens via:
- APIs (HTTP / REST / gRPC)
- Messages (Queues, Events)

---

## 🔹 Two Types of Communication

1️⃣ **Synchronous Communication**  
2️⃣ **Asynchronous Communication**

---

# 1️⃣ Synchronous Communication

## 🔹 What is Synchronous Communication?

- Service A **calls Service B**
- Service A **waits** for response
- Blocking communication

example 

User clicks "View Order"

Client (Browser / App)
        │
        │ 1️⃣ GET /orders/101
        ▼
Order Service
        │
        │ 2️⃣ Needs user details to build response
        │    → calls User Service (await)
        │
        │ ──────────────── BLOCKED ────────────────
        │   ❌ Cannot respond to client
        │   ❌ Cannot finish order logic
        │
        │                     User Service
        │                     │
        │                     │ 3️⃣ Query DB
        │                     │    Heavy computation
        │                     │    (5 seconds delay)
        │                     ▼
        │               User DB / Processing
        │
        │ ◄────────────── response after 5 sec
        │
        │ 4️⃣ Order Service resumes execution
        │    Combines order + user data
        ▼
Client receives response (after 5 sec)


 # Advantages of synchronous application 
 
1️⃣ Simple to understand

Request → Response model

2️⃣ Immediate response

Client gets real-time data

Disadvantages of Synchronous Communication

1️⃣ Tight runtime dependency

If Service B is down → Service A fails

2️⃣ Performance issues

Slow service slows entire chain
Service A → Service B → Service C
If Service C is slow → everything is slow
Cascading failures
One failure causes multiple failures

3-> caller is blocked 
4-> when there is spike there will be request drop or data loss because too many request then er nedd to drop the request

# when to use synchronous commmunication 

when you need result before you move on
when you want real time response
when computation is very less  

# Asynchronous Communication
🔹 What is Asynchronous Communication?

Service A sends a message
Service A does NOT wait
Non-blocking communication

Service A ──event/message──► Queue
Service B ◄──consumes── Queue

🔹 Technologies Used

Message Queues (Kafka, RabbitMQ)
Event-driven systems

# Advantages of asynchronous communication
services don't need to wait for response
system can hanlde surge and spike better 
in case of a serge the message queeue or broker  will be filled with. lots of msg but the user will have no diff in experince howeve the notififcation ssytem will have to scale up to clear the backlog 
No data loss or request drop
better control ver failure because you can always try as message is present  the queeue 

# Disadvantages of asynchronous application
1-> Broker is the single point of failure
2->you system will be litle inconsistent  as asyncrhounus operation will be there so there will be some delayes 

# When to use asynchronous operation 
1-> when delayes in system is okay like noification, analytics 
2-> when you have mutliple services which needs to react on the same event




Problem in Microservices
In microservices:
One business task = multiple services
Example: Order Placement
Order Placement involves
Order Service
Payment Service
Inventory Service
Notification Service
These services must work together in a sequence.
👉 This sequence = Workflow

❌ Without workflow design (problem)

Who calls whom?

What if payment fails?

What if inventory is empty?

How to rollback previous steps?

👉 This creates chaos

So we need Workflow Management

Two ways to design workflows

There are ONLY 2 ways:

Orchestration

Choreography

Think of it like:

Orchestration → One boss controls everything

Choreography → Everyone knows their steps, no boss


What is Orchestration?

👉 One central service (Orchestrator) controls the workflow

It decides:

Who to call

In what order

What to do on failure

Example: Order Placement (Orchestration)
Flow

Order Service receives request

Order Service calls Payment Service

Payment Service responds

Order Service calls Inventory Service

Order Service calls Notification Service

👉 Order Service = Orchestrator

Drawbacks of Orchestration
Single point of failure
Tight coupling


What is Choreography?

👉 No central controller

Services communicate via events

Each service:

Listens to events

Reacts on its own
Example: Order Placement (Choreography)
Flow (Event-driven)

Order Service → emits OrderCreated

Payment Service → listens → emits PaymentCompleted

Inventory Service → listens → emits InventoryReserved

Notification Service → listens → sends email

👉 No service directly calls another

Drawbacks of Choreography





What is Remote Procedure Call (RPC)?
RPC = Calling a function that runs on another computer as if it is a local function

In simple words:
Your code calls a method
But that method actually runs on another service / server
You don’t care about network details

Problem without RPC

Imagine Service A wants data from Service B

Without RPC:

You manually write:

HTTP requests

URLs

JSON parsing

Error handling

Serialization / deserialization

👉 Too much boilerplate code

```javascript 
response = http.post(
    "http://userservice/getUser",
    json={"id": 1}
)

user = response.json()["name"]
```

Simple RPC Example 
Service B (Server)
def add(a, b):
    return a + b

Service A (Client)
result = add(5, 3)   # Looks local
print(result)
add() runs on Service B
Result comes via network

What is Database-per-Service Pattern?
👉 Each microservice owns its own database.
No other service can directly access that database.
Rule:
One service → One database → Private access


Why this pattern is needed (Problem it solves)
❌ Problem in monolithic / shared DB systems

In old systems:

Many services share one database

All services read/write same tables

Issues:
Tight coupling
One change breaks multiple services
Hard to scale independently
Hard to change database schema
One slow query affects everyone

Example  :Social media platform 
chat services: its own database;
auth service : its own databses
vedio services : its own databses
profile services: its own databses

Advantages of Database-per-Service:
1. Loose Coupling

Services independent

Changes don’t break others

✅ 2. Independent Scaling

Scale DB only where needed

✅ 3. Freedom of Database Choice

Order → MySQL

Payment → PostgreSQL

Logs → MongoDB

✅ 4. Better Security

Disadvantages of Database-per-Service:

No Cross-Service Joins
You cannot do
JOIN users u JOIN orders o
👉 Data must be fetched via APIs
Eventual Consistency requires broken for conveying updates cross services
Data updates not immediate everywhere
multiinfra component needs to be managed and monitored

What is API Composition Pattern?

👉 API Composition means:

One service (Aggregator / Composer)

Calls multiple microservices

Combines their responses

Returns one final response to the client

Client calls ONE API instead of many

Why API Composition is needed (Problem it solves)
❌ Problem without API Composition

Frontend has to:

Call Order Service

Call User Service

Call Payment Service

Merge responses

Issues:

Too many network calls

Complex frontend logic

Tight coupling with backend services

✅ Solution
Create an API Composer service


Imagine a REAL product: Amazon – Order Details Page

When you open “My Order → Order Details”, the page shows:

Order info (items, price)

Payment status

Delivery status

User address

All this data comes from different microservices. and on combining all these we send it to frontend

SEQUENTIAL (ONE AFTER ANOTHER)
next service needs data from previous service.
Flow

Order Service → gives order_id and user_id

Using user_id, call User Service (this will wait till order service return the reponse)

Using order_id, call Delivery Service (this will also wait till order service return the reponse)
```python 
def get_order_details(order_id):
    order = order_service.get(order_id)        # Step 1 (wait)
    user = user_service.get(order["user_id"])  # Step 2 (wait)
    delivery = delivery_service.get(order_id)  # Step 3 (wait)

    return { "order": order, "user": user, "delivery": delivery }
```
PARALLEL (SAME TIME)
```python 
async def get_order_details(order_id):
    order = await order_service.get(order_id)

    payment_task = payment_service.get_async(order_id)
    delivery_task = delivery_service.get_async(order_id)
    user_task = user_service.get_async(order["user_id"])

    payment, delivery, user = await payment_task, delivery_task, user_task

    return {
        "order": order,
        "payment": payment,
        "delivery": delivery,
        "user": user
    }
```

*** the most important part of api composer is it improves the end user experince (beacuse now frontend has to make just one api call rather than making 3-4 diff diff api calls of diff diff services)

Disadvantages of Api composition


