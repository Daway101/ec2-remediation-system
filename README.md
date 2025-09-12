# EC2 Monitoring and Remediation System

## 📖 Company Context

Netflix runs on thousands of EC2 instances. If one of those goes down, it can quickly affect streaming quality for millions of users. When I started this project, I wanted to answer a simple question: *How can ServiceNow help us catch failures faster, guide engineers on what to do, and even fix problems without having to leave the platform?*  

This repository shows a **semi-automated incident response system** that I built inside ServiceNow. It connects with AWS to detect failing EC2 instances, creates incidents automatically, alerts engineers on Slack, and gives them a one-click remediation button. Along the way, I learned a lot about integration, automation, and designing with reliability in mind.  

---

## ⚙️ Step 1: Creating the Backbone

I started with the data. I built two custom tables:  

- **EC2 Instance** — where the AWS Integration Server pushes the latest instance health every minute (`ON` or `OFF`).  
- **Remediation Log** — the history book. Every remediation attempt, whether successful or not, is recorded here with details like timestamps, payloads, and responses.  

These tables became the heartbeat and memory of the whole system.  

---

## 🔐 Step 2: Connecting ServiceNow to AWS

Next, I set up the bridge between ServiceNow and AWS:  

- A **Connection & Credential Alias** to tie things together.  
- An **HTTP Connection** pointing to the AWS Integration Server.  
- **Basic Auth Credentials** to keep it secure.  

This made sure ServiceNow could both **receive EC2 health updates** and **send remediation calls back**.  

---

## 🔨 Step 3: Giving Engineers a Button

I didn’t want engineers scrambling through AWS docs during an outage. So I created:  

- A **UI Action**: *Trigger EC2 Remediation*. A simple button on the EC2 Instance record.  
- A **Script Include**: *EC2RemediationHelper*. This does the heavy lifting — making the API call back to AWS and writing the results into the Remediation Log.  

Now engineers had a **one-click way to restart an instance** directly from ServiceNow.  

---

## ⚡ Step 4: Automating the Response

Then came the fun part — automation with **Flow Designer**.  

When an EC2 instance goes `OFF`, the flow runs three things in parallel:  

1. **AI Search** looks up Knowledge Articles with remediation guidance.  
2. **Incident** gets created automatically in ServiceNow with EC2 + KB details.  
3. **Slack Notification** alerts the DevOps team instantly.  

This way, engineers don’t just hear about the issue — they get context and instructions right away.  

---

## 📚 Step 5: Teaching the System

Automation is only useful if it points engineers to the right solution. So I created a **Knowledge Article** with simple guidance:  

> “Run the *Trigger EC2 Remediation* UI Action on the associated EC2 Instance record.”  

I added keywords like *EC2, AWS, reboot, restart, instance* so AI Search could always find it.  

---

## 🛡 Step 6: Testing and Tuning

I tested the system by flipping EC2 records to `OFF` and watching everything fire:  
- Incidents were created.  
- Slack messages appeared.  
- AI Search logs showed knowledge retrieval.  
- HTTP Logs confirmed Slack POSTs (`200 OK`).  
- Remediation Log captured my manual restart attempts.  

To optimize:  
- I enabled **Detailed Logging** in AI Search.  
- I used **parallel branches** in Flow Designer so Slack and Incidents fire at the same time.  
- I restricted the UI Action with roles, so only DevOps could use it.  

---

## ⚠️ Challenges I Faced

- At first, I mixed up **Workflow Studio** with **Flow Designer** — I quickly learned Flow Designer is the modern way forward.  
- Some records weren’t showing in the **Update Set** until I discovered the trick: *Force Save flows* and manually *Add to Update Set*.  
- I had to be careful with the **Slack webhook** — if I exported it raw, GitHub would disable it. I swapped it with a placeholder.  
- AI Search didn’t find my article at first because it was in the wrong Knowledge Base. Publishing it to the IT KB (indexed by Service Portal) fixed that.  
- Validating everything took digging through **System Logs** and **HTTP Logs**, but once I figured that out, I had clear proof the system was working.  

---

## 📚 What I Learned

- How to design a **scoped application** in ServiceNow end to end.  
- How to securely integrate external systems with **connections and credentials**.  
- The power of **Flow Designer** and how to build parallel, event-driven automation.  
- Why **auditability** matters — incidents, logs, and knowledge working together tell the full story.  

---

## 🚀 Optimizations and Future Ideas

- Add **retry logic** for Slack messages if the first POST fails.  
- Build **auto-remediation** for safe failures, with manual fallback.  
- Make AI Search smarter by including **error codes** in queries.  
- Add a **dashboard** for real-time visibility of EC2 health and remediation success rates.  
- Create a **DevOps role** instead of relying on `admin` for access.  
- Add **interactive Slack buttons** (remediate directly from Slack).  
- Expand to **multi-region EC2 monitoring** in one ServiceNow view.  

---

## 👨‍💻 DevOps Experience

- Here’s how it feels to use the system:
- You get a Slack alert: “EC2 Instance i-1234567890 is OFF. Remediation guidance available.”
- At the same time, a ServiceNow Incident is created with the same info and a KB link.
- You open the EC2 record in ServiceNow, click Trigger EC2 Remediation, and seconds later see Success = True in the Remediation Log.
- The old 45-minute delay is gone. Netflix engineers can now act in real time, with confidence.

