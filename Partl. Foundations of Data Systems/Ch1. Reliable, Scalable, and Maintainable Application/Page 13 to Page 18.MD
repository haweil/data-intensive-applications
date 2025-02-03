
# Chapter 1: Page 13 to Page 18
## Understanding System Performance and Response Time Metrics
## 📊 Performance Under Load

This section of *Designing Data-Intensive Applications* discusses how to evaluate a system’s performance when the load changes. The author presents two key perspectives:

1. **Performance Under Fixed Resources** – If system resources (CPU, memory, network bandwidth, etc.) remain the same but the load increases, how does performance degrade?
2. **Scaling Resources to Maintain Performance** – If the load increases, how much do we need to scale up resources to keep the performance stable?

To answer these questions, performance metrics are needed. The book introduces two key metrics based on the system type:

- **Batch Processing Systems (e.g., Hadoop):** The most relevant metric is *throughput*, which refers to the number of records processed per second or the total time taken to complete a job on a dataset.
- **Online Systems:** The focus is on *response time*, which is the time taken from when a client sends a request to when they receive a response.
---
## 🕒 Latency vs. Response Time

Although "latency" and "response time" are often used interchangeably, they have distinct meanings:

### 📝 Response Time:
The total time a client waits for a response after sending a request. It includes:
- **Service Time:** The actual time needed to process the request.
- **Network Delays:** The time taken for data to travel between the client and server.
- **Queueing Delays:** The waiting time before the request gets processed if the system is busy.

### ⚡ Latency:
The time a request spends waiting before it starts being processed. It does not include processing time—just the waiting period.


### 💡 Why Response Time Varies

Even if you send the same request multiple times, response time can fluctuate. This variation becomes even more noticeable in systems handling different types of requests. Instead of thinking of response time as a fixed value, we should consider it as a *distribution of values*.

#### Causes of Response Time Variation

- Some requests are inherently slower because they process more data.
- Even requests that should take the same time can experience delays due to:
  - Context switches (CPU switching to a background process).
  - Network issues (lost packets requiring retransmission).
  - Garbage collection pauses in memory management.
  - Page faults (data being loaded from disk).
  - Physical factors like vibrations affecting server hardware.

---
## Understanding Response Time Metrics

When measuring the performance of a system, it's common to report the *average response time*. However, using the *average (mean)* is not always the best way to understand how long users actually wait. Let's break this down step by step.

### 1. Why is the Average (Mean) Not Always Useful?

The **average (mean)** response time is calculated as:

\[
\text{Mean} = \frac{\text{Sum of all response times}}{\text{Number of requests (n)}}
\]

For example, if you have five response times:

\[
[100ms, 150ms, 200ms, 10,000ms, 50ms]
\]

The mean (average) response time would be:

\[
\frac{100 + 150 + 200 + 10,000 + 50}{5} = \frac{10,500}{5} = 2,100ms
\]

This result of **2,100ms** doesn't represent the typical user experience because one extremely slow request (10,000ms) has heavily skewed the average.

### Why is this misleading?

- The **mean** can be **distorted by outliers** (extremely slow or fast requests).
- The **average** does not show **what most users experience**. Most users may experience much lower response times, but the slowest requests create a higher average.

In this case, even though the average response time is 2,100ms, most users actually experienced much faster responses, closer to the 100ms - 200ms range.

This number (2,100 ms) does not represent what most users experience! Most users actually wait much less than 2,100 ms, but one extremely slow request (10,000 ms) made the average very high.

Thus, the mean is not a good indicator of typical response time because:

- It is affected by outliers (very slow requests).
- It does not tell us how many users actually experienced a delay.

### 2. Using Percentiles Instead of the Mean

A better way to measure response time is using *percentiles*, which help us understand how response times are distributed.

#### What is a Percentile?

- If you sort all response times from fastest to slowest, the **median (50th percentile, p50)** is the middle value.
- The **p50 response time** means that *50% of requests were faster than this value, and 50% were slower.*

##### Example:

If the response times are: `[50ms, 100ms, 150ms, 200ms, 10,000ms]`

- The **median (p50)** is `150ms` because two requests were faster, two were slower.
- This means half the users waited less than `150ms`, and half waited more than `150ms`.

#### Higher Percentiles (p90, p95, p99)

- **90th percentile (p90):** 90% of requests are faster than this value, and 10% are slower.
- **95th percentile (p95):** 95% of requests are faster, and only 5% are slower.
- **99th percentile (p99):** Only 1% of requests are slower than this.

👉 **Why are these useful?**

- **p50 (median)** tells us what most users experience.
- **p90/p95** tells us what experience the slowest 10% or 5% of users have.
- **p99** shows the worst-case scenario for the slowest 1% of users.

##### Example:

Imagine a website with the following response times (sorted):

`[100ms, 120ms, 150ms, 180ms, 200ms, 500ms, 1000ms, 5000ms, 10000ms]`

- **p50 (median) = 180ms** → Half of users get responses in less than `180ms`.
- **p90 = 5000ms** → 90% of users get responses in less than `5000ms`.
- **p99 = 10000ms** → 99% of users get responses in less than `10,000ms`, but 1% wait this long!

### 3. The Median (p50) for Single Requests vs. Multiple Requests

The **median (p50)** applies to a single request. But in reality, a user may make multiple requests (e.g., loading a webpage requires multiple API calls).

If a user makes several requests per session, there is a higher chance that at least one of them is slower than the median.

For example:

- If the **p50** is `200ms`, it means that **50% of requests are slower than 200ms**.
- If a webpage loads 10 resources, the chance that at least one resource takes longer than `200ms` is much higher than 50%.

Thus, **users may perceive performance worse than the median suggests** because they often load multiple elements at once.

### Summary

✅ The **mean (average) response time is misleading** because outliers (very slow requests) distort it.

✅ **Percentiles are better for understanding real user experience:**

- **p50 (median)** → Typical response time for most users.
- **p90/p95** → How bad it is for the slowest users.
- **p99** → Worst-case scenario.

✅ **Users making multiple requests per session are more likely to experience delays**, even if the median is low.

---

## Understanding High Percentiles and Their Impact on Performance

In system performance analysis, high percentiles (p95, p99, p999) help identify outliers—the slowest requests that impact user experience. Let's break this down step by step.

### 1. What Do High Percentiles Represent?

A **percentile** represents the percentage of requests that are faster than a given response time.

- **p95 (95th percentile):** 95% of requests are faster than this value, while 5% are slower.
- **p99 (99th percentile):** 99% of requests are faster, while 1% are slower.
- **p999 (99.9th percentile):** 99.9% of requests are faster, while only 0.1% (1 in 1,000) are slower.

#### 👉 Example Interpretation:

If the **p95** response time is **1.5 seconds**, this means:

- 95 out of 100 requests take less than **1.5 seconds**.
- 5 out of 100 requests take **1.5 seconds or more**.

These high percentiles help analyze the worst-performing requests, often referred to as **tail latencies**.

### 2. Why Are High Percentiles (Tail Latencies) Important?

Even though only a small percentage of users experience high latency, they can be some of the most valuable customers.

#### 🔹 Example: Amazon's 99.9th Percentile Focus

Amazon monitors the **99.9th percentile (p999)** because customers with the slowest requests are often those who buy the most (large accounts with more data). 

- Keeping these customers happy prevents revenue loss and maintains customer loyalty.

#### 📌 Real-World Impact of Slow Response Times:

- **Amazon observed:** A **100 ms delay** in response time reduces sales by **1%**.
- **Other studies:** A **1-second slowdown** reduces customer satisfaction by **16%**.

#### 🔹 Why Not Optimize for p9999 (99.99th Percentile)?

The slowest **1 in 10,000 requests** are often affected by random, uncontrollable factors (e.g., network issues, system spikes).

- Optimizing these extreme cases is **too expensive** and provides little benefit.

### 3. Summary & Key Takeaways

- ✅ **p95, p99, and p999** are useful for detecting slow requests (outliers).
- ✅ Improving **high percentiles** directly improves **user experience**.
- ✅ Companies like **Amazon** prioritize the **99.9th percentile** because slow responses impact their highest-value customers.
- ✅ Beyond a certain point (e.g., **p9999**), optimization is **costly** and has diminishing returns.


---
## Understanding Service Level Objectives (SLOs) and Service Level Agreements (SLAs)

### 1. What Are SLAs and SLOs?

- **Service Level Agreement (SLA):** A contract that defines performance expectations between a service provider and its clients.
- **Service Level Objective (SLO):** A specific performance goal within the SLA that defines what "acceptable" performance is.

#### 💡 Example of an SLA:
A company guarantees that:
- ✅ **Median response time** must be under **200 ms**.
- ✅ **99th percentile response time** must be under **1 second**.
- ✅ The service must be **available 99.9%** of the time.

📌 **If these conditions aren’t met, customers may demand a refund.**

---

### 2. Impact of Queueing Delays on High Percentiles

#### 🔹 What Causes Queueing Delays?
- A server has limited processing power (e.g., limited CPU cores).
- If multiple slow requests are processed first, **faster requests** get stuck waiting.
- This issue is known as **Head-of-Line Blocking**—even fast requests appear slow because they wait behind slow ones.

#### 🔹 Why Is Measuring Response Time on the Client Side Important?
- The server might complete requests quickly, but queueing effects make the **actual response time** seen by users much worse.
- **Measuring on the client side** gives a true picture of what users experience.

---

### 3. How Load Testing Can Skew Measurements

When testing system scalability, we often generate fake load to see how the system behaves. However:

- ❌ If the test client waits for each request to finish before sending the next, it **artificially shortens the queue**.
- ✅ A realistic test client should send requests **continuously**, regardless of whether previous ones have finished.

#### 💡 Why?
- In the real world, users don’t wait for others to finish before making new requests.
- This ensures **accurate load testing** and helps predict real-world performance.

---

### 4. Summary & Key Takeaways

- ✅ **SLAs & SLOs** define performance expectations (e.g., max response times, uptime).
- ✅ **Queueing delays** and **head-of-line blocking** slow down response times, even for fast requests.
- ✅ **Measuring response time on the client side** is critical to seeing **real user experience**.
- ✅ **Load testing** must simulate **real-world request patterns** to avoid misleading results.

---
## Understanding Percentiles in Backend Services

### 1. Why Are High Percentiles Important in Backend Services?
Backend services often need to handle multiple requests per user action. These requests may be made in parallel, but the end-user still has to wait for the **slowest request** to finish.

#### 💡 Example:
Imagine a mobile app making 5 backend calls to load a page:
- ✅ 4 calls finish in **100 ms**.
- ❌ 1 call takes **2 seconds**.
- 🚨 The user must wait **2 seconds**—even though most calls were fast!

This problem is called **Tail Latency Amplification** because even a small percentage of slow requests can significantly affect user experience.

---

### 2. How to Monitor Response Time Percentiles?
To monitor system performance, you should track **response times continuously** and update percentiles dynamically.

#### A Common Approach:
- 📌 **Maintain a rolling window** (e.g., the last 10 minutes of request times).
- 📌 Every minute, calculate:
  - The **median** (p50).
  - Other percentiles (p95, p99, etc.).
- 📌 **Plot these metrics** on a dashboard for real-time monitoring.

---

### 3. Efficiently Calculating Percentiles
A naïve way to compute percentiles is:
1️⃣ **Store all response times** for the last 10 minutes.
2️⃣ **Sort them** every minute.
3️⃣ **Pick the required percentiles.**

❌ **Problem:** Sorting is slow and inefficient!

✅ **Better Solution:** Use specialized approximation algorithms like:
- **Forward Decay**
- **t-Digest**
- **HdrHistogram**

---

### 4. Common Mistakes: Don't Average Percentiles!
A common mistake is averaging percentiles to reduce time resolution or combine data from multiple servers.

#### ❌ Incorrect Approach:
- (p95 of server A + p95 of server B) / 2

#### ✅ Correct Approach:
- Instead of averaging, **combine histograms** from different servers and recalculate percentiles.

---

### 5. Key Takeaways
- ✅ **High percentiles matter**—a single slow backend call can slow down the entire user experience.
- ✅ Track percentiles continuously with **rolling windows** and dashboards.
- ✅ Use **efficient algorithms** like **t-Digest** and **HdrHistogram** instead of sorting all data.
- ✅ Never average percentiles—**combine histograms** instead.

---
## Understanding Approaches for Coping with Load

As a system grows, it must handle increasing load while maintaining good performance. To achieve this, we need scalability strategies.

### 1. Scaling Up vs. Scaling Out
There are two primary approaches to scaling a system:

#### 🔹 Scaling Up (Vertical Scaling)
- 📌 **Definition:** Increasing the power of a single machine by upgrading its CPU, RAM, or storage.
- 📌 **Advantages:**
  - ✅ Simpler architecture (fewer machines to manage).
  - ✅ Easier to develop and maintain.
- 📌 **Disadvantages:**
  - ❌ Limited by hardware capabilities (at some point, you can’t upgrade further).
  - ❌ Expensive high-end machines.

#### 💡 Example:
- Upgrading from a **4-core CPU** to a **32-core CPU**.

#### 🔹 Scaling Out (Horizontal Scaling)
- 📌 **Definition:** Adding more machines (servers) and distributing the load across them.
- 📌 **Advantages:**
  - ✅ More scalable than vertical scaling (no single machine limit).
  - ✅ Can be cheaper than a single high-end machine.
- 📌 **Disadvantages:**
  - ❌ More complex architecture (requires load balancing, distributed systems).
  - ❌ Potential network latency between machines.

#### 💡 Example:
- Instead of upgrading one machine, adding **10 smaller servers** and distributing requests among them.

This approach is also called a **Shared-Nothing Architecture** because each machine operates independently, without shared memory.

---

### 2. Hybrid Approach: The Best of Both Worlds
A good system often combines both approaches:
- 🔸 Use powerful machines where simplicity matters.
- 🔸 Scale out with multiple machines when handling high loads.

#### 💡 Example:
- Instead of using 1 supercomputer or 100 tiny servers, use **5-10 powerful machines** for a balance of performance, cost, and manageability.

---

### 3. Manual vs. Elastic Scaling

#### 🔹 Manual Scaling
- 📌 **Definition:** A human analyzes system load and adds machines manually when needed.
- 📌 **Advantages:**
  - ✅ Simpler, predictable behavior.
  - ✅ Less risk of unexpected costs.
- 📌 **Disadvantages:**
  - ❌ Slow reaction time to traffic spikes.

#### 💡 Example:
- A team monitors **CPU usage** and adds more servers when it reaches 80%.

#### 🔹 Elastic Scaling (Auto-Scaling)
- 📌 **Definition:** The system automatically adds or removes computing resources based on load.
- 📌 **Advantages:**
  - ✅ Handles unpredictable spikes automatically.
  - ✅ Saves cost by scaling down when demand is low.
- 📌 **Disadvantages:**
  - ❌ More complex setup.
  - ❌ Can lead to unexpected costs if misconfigured.

#### 💡 Example:
- **AWS Auto Scaling** detects increased traffic and adds servers automatically.

---

### 4. Key Takeaways
- ✅ **Scaling Up (Vertical Scaling):** Upgrade a single machine (simple but has limits).
- ✅ **Scaling Out (Horizontal Scaling):** Add more machines (complex but highly scalable).
- ✅ **Hybrid Approach:** Combine both for efficiency.
- ✅ **Manual Scaling:** Simpler but slower to react to changes.
- ✅ **Elastic Scaling:** Automatically adjusts capacity but adds complexity.

---
## Scaling Stateful vs. Stateless Systems

Scaling stateless services is relatively easy—just add more machines and distribute the requests. However, scaling stateful systems (e.g., databases) is much more complex because data consistency, availability, and coordination become major challenges.

### 1. Scaling Stateful Systems: Challenges & Approaches

#### 🔹 Traditional Approach: Scale Up First
- 📌 **Common Wisdom:** Keep the database on a single powerful node until it becomes too costly or unreliable.
- 📌 **Why?**
  - ✅ Avoids the complexity of distributed systems.
  - ✅ Ensures strong consistency without complex coordination.

#### 🔹 Distributed Databases: The Future?
As distributed system tools improve, more applications are adopting distributed databases early.
- 📌 **Possible Future Trend:** Distributed-first approach, even for smaller applications.

#### 💡 Example:
- **Cloud-native databases** (e.g., **CockroachDB**, **Spanner**, **YugabyteDB**) are designed for distributed use from the start.

---

### 2. No One-Size-Fits-All Scaling Solution
Each system has different scaling needs based on:
- 📌 **Read volume** (e.g., caching needed for high read traffic).
- 📌 **Write volume** (e.g., partitioning/sharding for high write traffic).
- 📌 **Data size** (e.g., distributed storage for massive datasets).
- 📌 **Latency requirements** (e.g., real-time systems need fast response times).

#### 💡 Example:
- 🔸 A system handling **100,000 requests/sec**, each **1 kB** needs efficient request distribution (load balancing, caching).
- 🔸 A system handling **3 requests per minute**, each **2 GB** needs optimized storage and batch processing.

> 🛑 Same data throughput, but completely different scaling architectures!

---

### 3. Assumptions Matter in Scalability Planning
A scalable system is built based on assumptions about:
- 📌 **Which operations** are frequent vs. rare?
- 📌 **Expected traffic growth patterns?**
- 📌 **How data** is accessed and stored?

If assumptions are wrong, scaling efforts can be:
- ❌ **Wasted:** Overengineering a problem that doesn’t exist.
- ❌ **Counterproductive:** Introducing unnecessary complexity.

#### 💡 Example:
- A **startup** should prioritize rapid iteration on product features over premature optimization for hypothetical large-scale traffic.

---

### 4. Scaling Uses General-Purpose Building Blocks
Even though every system is unique, scalable architectures often rely on common patterns:
- 📌 **Caching** (e.g., **Redis**, **Memcached**) – To reduce load on databases.
- 📌 **Load Balancing** (e.g., **Nginx**, **HAProxy**, **AWS ELB**) – To distribute requests across servers.
- 📌 **Database Sharding** (e.g., **MySQL**, **PostgreSQL with Citus**) – To handle high write loads.
- 📌 **Replication** (e.g., **Leader-Follower**, **Multi-Leader**, **Eventual Consistency**) – To improve availability and read performance.

#### 💡 Key Takeaway:
- Scaling isn’t about finding a magic solution—it’s about choosing the right patterns for your specific use case.

