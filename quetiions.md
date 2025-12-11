# 1. What is conditional flow In APIGEE

ANSWER
* * *

1\. Very simple meaning
-----------------------

In **Apigee**, a **conditional flow** means:

> “**Run these steps only if this condition is true.**”

Like saying in English:

*   **If** request URL is `/login` → do **Flow A**
*   **If** request URL is `/payment` → do **Flow B**
*   **Otherwise** → do **default flow**

So it’s nothing but **IF–ELSE** for your API proxy.

* * *

2\. Real-life example (non-technical)
-------------------------------------

Imagine a **reception desk** in a company:

*   If person is **Employee** → let them go directly inside.
*   If person is **Visitor** → give visitor pass, ask details.
*   If person is **Delivery boy** → send to security gate.

Here:

*   “Employee / Visitor / Delivery boy” = **condition**
*   “Let inside / give pass / send to security” = **different flows**

Apigee conditional flow works **exactly like this**, but for **API requests**.

* * *

3\. Where does it sit in Apigee?
--------------------------------

In an Apigee API Proxy, you have **flows** in:

*   **ProxyEndpoint** (request coming from client)
*   **TargetEndpoint** (request/response between Apigee and backend)

Inside ProxyEndpoint you can define:

*   **PreFlow** – always runs (no condition)
*   **Flows (Conditional Flows)** – run **only if condition is true**
*   **PostFlow** – always runs after main flows

* * *

4\. Basic structure in simple words
-----------------------------------

A **conditional flow** has 2 main parts:

1.  **Condition** → “WHEN should this run?”
2.  **Policies / Steps** → “WHAT to do when it runs?”

### Example (in English)

> **Condition:** request path is `/login`  
> **Steps:**
> 
> *   Validate username/password format
> *   Add extra security header
> *   Log login attempt
>     

So only **/login** calls will go through this special path.

* * *

5\. Simple Apigee example (just to see)
---------------------------------------

Don’t worry about XML too much. Just observe the **idea**:

```xml
<Flow name="Login Flow">
  <Condition>(proxy.pathsuffix = "/login")</Condition>
  <Request>
    <Step><Name>Security-Policy</Name></Step>
  </Request>
  <Response>
    <Step><Name>Login-Response-Transform</Name></Step>
  </Response>
</Flow>
```

Read like a sentence:

*   **If** `proxy.pathsuffix = "/login"`  
    → apply **Security-Policy** on request  
    → apply **Login-Response-Transform** on response

If the request is `/orders` or `/users`, this flow is **skipped**.

* * *

6\. What can you check in a condition?
--------------------------------------

You can write conditions using variables like:

*   **URL path** → `proxy.pathsuffix`
    *   e.g. `/v1/users`, `/v1/orders`
*   **HTTP method** → `request.verb`
    *   `GET`, `POST`, `PUT`, etc.
*   **Query param** → `request.queryparam.plan`
    *   `?plan=gold`
*   **Header** → `request.header.User-Type`
*   **Status code** → `response.status.code`
*   And many more…

### Examples in Apigee-style condition (read like English):

1.  **For GET /orders only**
    ```text
    (request.verb = "GET") AND (proxy.pathsuffix = "/orders")
    ```
2.  **For premium users only**
    ```text
    (request.header.userType = "premium")
    ```
3.  **Error handling (5xx status)**
    ```text
    (response.status.code >= 500)
    ```

* * *

7\. Very simple scenarios
-------------------------

### Scenario 1: Different rules for GET vs POST

**Problem:**  
Same endpoint `/users`, but:

*   For **GET /users** → only read data.
*   For **POST /users** → create user, need stronger checks.

**Solution with conditional flows:**

*   Flow 1: Condition → `request.verb = "GET"`
    *   Apply: light rate limit, log read-only access.
*   Flow 2: Condition → `request.verb = "POST"`
    *   Apply: strict quota, extra security, validation.

So you handle **read** and **write** differently using conditions.

* * *

### Scenario 2: Free vs Paid plan

**Problem:**

*   Free users → 100 calls/day
*   Paid users → 10,000 calls/day

**Solution:**

*   Flow 1: Condition → `request.header.plan = "free"`
    *   Apply: Quota policy → 100 / day
*   Flow 2: Condition → `request.header.plan = "paid"`
    *   Apply: Quota policy → 10,000 / day

One proxy, but **different treatment** depending on header.

* * *

### Scenario 3: Error handling flow

**Problem:**  
When backend gives 500 (server error), you want:

*   Custom error message
*   Extra logging
*   Send alert maybe

**Solution:**

*   Conditional flow in **Response**:
    *   Condition: `response.status.code >= 500`
    *   Steps: transform error, log details.

Normal 200 responses won’t trigger this flow.

* * *

8\. Default flow vs Conditional flows
-------------------------------------

In Apigee:

*   **PreFlow** → always runs (no condition)
*   **Flows** (conditional) → run only when condition true
*   **PostFlow** → always runs

If **no conditional flow** matches, only PreFlow + PostFlow run.

Think like:

```text
Always do: basic checks (PreFlow)

If path is /login → run Login Flow
Else if path is /orders → run Orders Flow

Always do: final logging, header add (PostFlow)
```

* * *

9\. How to answer in an interview (short & clear)
-------------------------------------------------

You can say:

> “In Apigee, a **conditional flow** is a part of the proxy flow that runs **only when a certain condition is true**, like specific path, method, header, or status code.  
> For example, I can create one flow for `/login` and another flow for `/payment`, each with different policies. Apigee checks the condition, and only the matching flow executes.”

If they ask example, you can add:

> “For instance, I can have:
> 
> *   one conditional flow for `GET /users` with simple quota
> *   and another for `POST /users` with strict quota and validation.  
>     Both are within the same proxy, but executed conditionally.”
>     

* * *

If you want, next I can show:

*   How to write **2–3 real conditional flows** for a sample API (like `/login`, `/orders`)
*   Or help you prepare **2–3 scenario-based answers** only for interview.


# Q2. What is policies in APIGEE and what common diffrent types of policies availble in APIGEE ?

* * *

1\. Super simple definition
---------------------------

In **Apigee**, a **policy** is:

> 🧩 **A ready-made rule or feature that you attach to your API, without writing code.**

Like:

*   “Check API key”
*   “Limit calls”
*   “Convert XML to JSON”
*   “Log this request”

You **don’t** write Java/Node code – you just **add a policy** in the proxy.

* * *

2\. Real-life example (non-technical)
-------------------------------------

Imagine a **main gate** of an office:

At the gate, security can apply different **rules**:

1.  Check ID card
2.  Check bag
3.  Record entry in register
4.  Limit visitor count

Each rule is like an **Apigee policy**:

*   “Check ID card” → **Security policy**
*   “Record entry” → **Logging policy**
*   “Limit visitors” → **Quota / Rate limit policy**

You decide:  
👉 “At this gate, apply rules 1, 3, and 4”  
In Apigee:  
👉 “On this API, apply policies A, B, C”

* * *

3\. Where are policies used?
----------------------------

In an Apigee **API Proxy**, you have flows:

*   **PreFlow** – always runs
*   **Conditional Flows** – run only when condition true
*   **PostFlow** – always runs later

Inside each flow, you attach **policies**.

So you can think:

> **Flow = when to run**  
> **Policy = what to do**

* * *

4\. Common types of policies (with simple examples)
---------------------------------------------------

### 1️⃣ Security policies 🔐

Used to **secure** your API.

*   **VerifyAPIKey** – check if client sent a valid API key
*   **OAuthV2** – validate access token
*   **VerifyJWT** – validate JWT token

🟢 Example:  
For `/payments` API, you add:

*   `VerifyAPIKey` → only registered apps can call
*   `OAuthV2` → only users with valid token can access

* * *

### 2️⃣ Traffic control policies 🚦

Used to **control how many requests** come.

*   **SpikeArrest** – stop sudden burst of traffic
    *   e.g. `10 requests/second`
*   **Quota** – limit total requests
    *   e.g. `1000 requests/day` per app

🟢 Example:  
Free plan:

*   `Quota` → 1000 requests/day
*   `SpikeArrest` → 5 requests/second

Paid plan:

*   `Quota` → 10,000 requests/day
*   `SpikeArrest` → 50 requests/second

These are all **policies**, just attached with different values.

* * *

### 3️⃣ Transform / Message policies 🔁

Used to **change the request or response**.

*   **AssignMessage** – change URL, headers, body etc.
*   **JSONToXML** / **XMLToJSON** – convert formats
*   **ExtractVariables** – read data from path, header, body into variables

🟢 Example:

Client sends:

```json
{ "userId": 123 }
```

Backend needs:

```json
{ "id": 123 }
```

You can use:

*   `ExtractVariables` → read `userId`
*   `AssignMessage` → create new body with `"id"`

No code in backend changed. Apigee handled it with **policies**.

* * *

### 4️⃣ Error handling policies ⚠️

*   **RaiseFault** – send custom error message
*   You can show friendly error instead of ugly backend error.

🟢 Example:

If backend is down, instead of:

```json
{ "error": "java.net.SocketTimeoutException" }
```

Use `RaiseFault` to send:

```json
{
  "code": "SERVICE_UNAVAILABLE",
  "message": "Please try again after some time."
}
```

* * *

### 5️⃣ Logging / Analytics policies 📋

*   **MessageLogging** – send logs to log system (like syslog, log server)
*   You can log headers, paths, IDs etc.

🟢 Example:

Log every `/login` call with:

*   User ID
*   Client IP
*   Time

You just attach a `MessageLogging` policy in that flow.

* * *

5\. How policy looks (just to feel it)
--------------------------------------

Don’t worry about XML; read it like English:

```xml
<SpikeArrest name="Limit-Per-Second">
  <Rate>10ps</Rate>
</SpikeArrest>
```

Meaning:

> Add a **SpikeArrest policy** named `Limit-Per-Second`  
> allow only **10 calls per second**.

Then you attach this in a Request flow:

```xml
<Step>
  <Name>Limit-Per-Second</Name>
</Step>
```

So:

*   **Policy definition** → what it does
*   **Step** → where it runs (PreFlow / conditional flow / PostFlow)

* * *

6\. One small full example in plain English
-------------------------------------------

Let’s say you have an API:  
`GET /v1/users`

You decide in Apigee:

1.  Check API key
2.  Limit requests to 1000 per day
3.  Log each request

So you attach:

*   `VerifyAPIKey` policy
*   `Quota` policy
*   `MessageLogging` policy

No backend change, no Java code.  
Just **add 3 policies** in the proxy.

* * *

7\. How to answer in interview (simple version)
-----------------------------------------------

You can say something like:

> “In Apigee, **policies are ready-made building blocks** that you attach to your API proxy to handle common tasks like security, rate limiting, transformation, logging, and error handling—without writing custom code.  
> For example, I can use `VerifyAPIKey` to validate keys, `SpikeArrest` to prevent traffic spikes, `Quota` to control daily usage, and `AssignMessage` to modify the request or response.”

If they ask example, add:

> “For a payment API, I might add `VerifyAPIKey` and `OAuthV2` for security, `Quota` for rate limiting, and `MessageLogging` to log all payment calls.”

* * *

If you want, next we can do:

*   A small “**list of 10 must-know Apigee policies**” with one-line explanation each,
*   Or mock interview Q&A only about **policies + conditional flows + pre/post flows**.

# Q3. What is no target proxy ?

* * *

1\. First, what happens in a _normal_ Apigee proxy?
---------------------------------------------------

Normally the flow is:

**Client → Apigee → Backend service**

In Apigee words:

*   **ProxyEndpoint** = front door (client calls here)
*   **TargetEndpoint** = backend (Apigee forwards request here)

So a normal proxy is like:

> “I received the request, I apply policies, then I send it to **some backend URL** and return the backend’s response.”

* * *

2\. What is a **No Target Proxy**?
----------------------------------

A **No Target Proxy** (also called _targetless proxy_) is:

> An Apigee proxy **that has no backend (no TargetEndpoint)**.  
> Apigee itself handles the request and sends the response.

So the flow becomes:

**Client → Apigee → (no backend) → Apigee sends response**

There is **no call to any external service**.

*   You **still have ProxyEndpoint**
*   But you **do NOT have TargetEndpoint**

* * *

3\. Simple real-life example
----------------------------

Imagine:

*   In a normal case:  
    Customer calls **reception** → receptionist forwards call to **some department** → answer comes back.
*   In a **No Target** case:  
    Customer calls reception  
    Receptionist already knows the answer  
    👉 She answers directly, **no need to transfer** to any department.

That’s exactly **No Target Proxy**:

*   Request comes to Apigee
*   Apigee replies directly (using policies)
*   No other backend server is involved

* * *

4\. When do we use No Target Proxy?
-----------------------------------

### ✅ 1) Mock / Dummy API (for testing)

You want to give frontend team **some API URL** to develop UI,  
but the real backend is not ready yet.

So you create a **No Target Proxy**:

*   Any call to `/users` returns a **fixed sample JSON** from Apigee itself.

Example response from Apigee (no backend):

```json
{
  "userId": 123,
  "name": "Test User",
  "status": "mock-data"
}
```

Frontend can start development,  
backend team can work in parallel.

* * *

### ✅ 2) OAuth token endpoint / security endpoint

Often, **/oauth/token** API in Apigee is a **No Target Proxy**.

*   Client calls `/oauth/token`
*   Apigee uses `OAuthV2` policy to:
    *   verify client
    *   create token
*   Apigee sends back access token directly

No external backend needed —  
all work is done by Apigee policies.

* * *

### ✅ 3) Simple logic APIs inside Apigee

Sometimes you want:

*   Small APIs that do only:
    *   header handling
    *   variable calculations
    *   maybe call _another_ API via ServiceCallout

Example:

*   Client calls `/status`
*   Apigee:
    *   just returns `"OK"` with some metadata
    *   or checks something and then replies

If you don’t have a real backend server for this,  
you can use a **No Target Proxy**.

* * *

### ✅ 4) Aggregator / Orchestrator APIs (advanced, later)

Apigee can call **other APIs** itself using `ServiceCallout` policy.

Flow:

*   Client → No Target Proxy
*   Inside Apigee:
    *   call API A
    *   call API B
    *   combine their responses
*   Apigee returns **combined response** to client

Still: there is **no TargetEndpoint**.  
External calls happen via **policies**, not via TargetEndpoint.

(This is more advanced, but good to know for later/interview.)

* * *

5\. How does it look technically?
---------------------------------

In a **normal** proxy, you have something like:

```xml
<TargetEndpoint name="default">
  <HTTPTargetConnection>
    <URL>https://backend.mycompany.com/service</URL>
  </HTTPTargetConnection>
</TargetEndpoint>
```

In a **No Target Proxy**:

*   This **TargetEndpoint section does not exist**
*   Or there is a `<NoTargetEndpoint/>` style configuration (depending on UI/implementation)

Instead, in `ProxyEndpoint` flows you might have policies like:

*   `AssignMessage` → set response body and status code
*   `RaiseFault` → send custom error
*   `OAuthV2` → issue token
*   `MessageLogging` → log things

So Apigee builds the response **using policies only**.

Example idea (in words, not exact code):

> In PreFlow Response:
> 
> *   AssignMessage → set body = `{ "status": "ok" }`, status code = 200
>     

No backend call.

* * *

6\. Summary in one line (for interview)
---------------------------------------

You can say:

> “A **No Target Proxy** in Apigee is an API proxy that has **no TargetEndpoint** — Apigee itself handles the request and responds directly, without calling any backend. It’s used for mock APIs, OAuth token generation, simple internal utilities, or orchestration done entirely through policies.”

If they ask for an example:

> “For example, we can build `/oauth/token` as a no-target proxy where the OAuthV2 policy validates the client and generates tokens, and Apigee sends the response directly without any backend service.”

* * *

