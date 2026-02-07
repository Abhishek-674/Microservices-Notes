# 📌 Microservices – Clean Notes (README)

---

## 🔹 What is Microservices Architecture?

- Each service is **independent**
- Each service has its **own responsibility**
- Each service exposes its **own API**
- Services communicate using **APIs or events**
- One service can call another service’s API

---

## 🔹 Example Service Flow

Auth Service → User Service → Order Service


---

## 🔹 Example: Service-to-Service Communication

### ✅ User Service
```js
// GET /users/1
app.get("/users/:id", (req, res) => {
  res.json({ id: 1, name: "Abhishek" });
});
✅ Order Service (calling User Service)
const axios = require("axios");

app.get("/orders/:id", async (req, res) => {
  const user = await axios.get("http://user-service/users/1");

  res.json({
    orderId: 101,
    user: user.data
  });
});
```
### How Microservices Communicate
# Synchronous Communication
REST (HTTP)
gRPC
Service waits for response
# Asynchronous Communication
Message Queues (Kafka, RabbitMQ)
Events
📌 Service does not wait for response

### Typical Request Flow
Frontend
   ↓
API Gateway
   ↓
Service A → Service B → Service C
### Handling Timeouts in Microservices
# Example Scenario
Search Service → returns blog posts
Analytics Service → returns likes, comments, views

When a user searches a blog:
Search Service calls Analytics Service to fetch likes & comments

# Possible Failure Cases
Request never reaches Analytics Service
Analytics response times out before reaching Search Service
Analytics Service takes too long due to heavy computation

# How to Handle These Cases
1. Use Default / Fallback Values
If Analytics Service fails:

{
  "likes": 0,
  "comments": 0
}
2. Retry with Exponential Backoff
Retry only when required
Use increasing delay:

2s → 4s → 8s → 16s
 Avoid infinite retries
Retry Concerns
Request may be non-idempotent
Example: Payment of ₹10 executed twice
Request may be expensive
Downstream services may also be overloaded

👉 Retry only when safe

3. Retry Only Idempotent Requests
GET → Safe to retry

POST (create payment, order) → Dangerous to retry

Example problem:
Accidentally creating two same posts

4. Graceful Degradation
Ignore analytics failure

Show partial / broken data instead of failing UI

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

