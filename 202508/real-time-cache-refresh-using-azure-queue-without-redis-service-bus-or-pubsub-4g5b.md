---
Title: 🔄 Real-Time Cache Refresh Using Azure Queue (Without Redis, Service Bus, or Pub/Sub)
Description: 
Author: Nitin Singh
Date: 2025-08-18T21:05:45.000Z
Robots: noindex,nofollow
Template: index
---
<p>🚀 A Lightweight Cache Refresh Mechanism with Azure Queue (No Redis, No Service Bus!)<br>
💡 The Problem</p>

<p>In our application, data was stored in the database, but not updated frequently.<br>
To improve performance, we wanted this data cached across multiple app instances.</p>

<p>The challenge:</p>

<p>Data changes occasionally (Insert, Update, Delete in Azure DB).</p>

<p>Whenever data changes, all app instances should refresh their cache.</p>

<p>We wanted to avoid Redis, Pub/Sub, or Service Bus and stick with Azure Queue Storage.</p>

<p>⚙️ The Solution</p>

<p>Here’s the approach I designed:</p>

<p>Cache Layer: Store the DB data in memory cache (per app instance).</p>

<p>Trigger on Change: When a DB operation (insert/update/delete) occurs, send a message to an Azure Queue with a TTL (Time-to-Live).</p>

<p>Background Listener: Each instance runs a lightweight listener that:</p>

<p>Periodically checks the approximate message count in the queue.</p>

<p>If count &gt; 0 → Refreshes local cache.</p>

<p>Message Cleanup: Since ApproximateMessagesCount is not real-time, each sender also schedules a DeleteMessageAsync after a delay, ensuring the message is eventually cleaned up.</p>

<p>This ensures:</p>

<p>All instances refresh cache consistently.</p>

<p>Queue acts like a broadcast signal, not a transport layer.</p>

<p>No dependency on Redis or Service Bus.</p>

<p>🔄 Flow Diagram<br>
flowchart TD<br>
    A[DB Change: Insert/Update/Delete] --&gt; B[Send message to Azure Queue (with TTL)]<br>
    B --&gt; C[Background Listener reads Approximate Message Count]<br>
    C --&gt;|Count &gt; 0| D[Refresh Local Cache]<br>
    D --&gt; E[Mark message for delayed DeleteMessageAsync]<br>
    E --&gt; F[Cache Synchronized Across Instances]</p>

<p>📌 Key Points</p>

<p>Uses Azure Storage Queue only (no Service Bus, no Redis).</p>

<p>ApproximateMessageCount drives the decision, but cleanup ensures correctness.</p>

<p>Messages act as cache refresh signals, not data carriers.</p>

<p>Works well in multi-instance deployments.</p>

<p>✅ Benefits</p>

<p>Lightweight – no extra infra needed.</p>

<p>Cost-effective – Azure Queue is cheap.</p>

<p>Resilient – each instance eventually refreshes even if messages are delayed.</p>

<p>🔮 Future Improvements</p>

<p>Use custom message metadata (e.g., message type, source host).</p>

<p>Replace ApproximateMessageCount polling with event-based push if scale increases.</p>

<p>Add dead-letter queue for failed refresh attempts.</p>

<p>✍️ Final Thoughts</p>

<p>This approach is perfect for scenarios where:</p>

<p>Data changes are infrequent.</p>

<p>You want shared cache consistency across multiple instances.</p>

<p>You don’t want to introduce Redis, Pub/Sub, or costly Service Bus.</p>

<p><strong>Additional Use Case: Configuration Management</strong></p>

<p>This approach isn’t limited to cache refresh.<br>
Imagine you have an application deployed across multiple environments or instances where only configuration values change from time to time.</p>

<p>Traditionally:</p>

<p>You’d need to redeploy apps or push configs manually.</p>

<p>With this pattern:</p>

<p>Store config values/keys in the database.</p>

<p>On config update → send a message to the queue.</p>

<p>All instances refresh their in-memory config cache automatically.</p>

<p>👉 Result: No redeployments needed for simple config changes.</p>

<p>👉 If you liked this idea, follow me here on Dev.to and let’s discuss how you handle cache refresh in your systems!</p>

