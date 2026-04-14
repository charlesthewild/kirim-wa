# KIRIM WA

A high-performance, distributed messaging engine built for resilience and account safety. This system utilizes a **Database-Driven Asynchronous Architecture** to orchestrate complex messaging campaigns across multiple WhatsApp accounts simultaneously.

---

## Core Architecture: The "Global Pool" Logic

The system is built on a **Decoupled Architecture**, meaning the list of recipients (The Job) is entirely separate from the phones (The Workers). This allows for a "Pull-Based" distribution model.

### 1. Database-Driven Recipient Groups
Instead of processing numbers in real-time from a UI input, the system utilizes a persistent SQLite backend:
* **Intelligent Parsing:** When numbers are pasted into the UI, the engine uses a Regex sanitizer to extract digits, ignoring spaces, dashes, brackets, and non-numeric text.
* **Persistence & De-duplication:** Numbers are stored in indexed **Groups**. The engine performs a collision check to ensure no duplicate numbers exist within the same group, saving database overhead and preventing double-messaging.
* **Cold Storage Strategy:** By keeping recipients in the database, the UI remains lightweight. The system only loads the necessary "chunk" of numbers when a campaign begins.

### 2. Asynchronous Worker Pull System
The Workers (linked WhatsApp accounts) act as independent processors that "compete" for tasks.
* **The Pending Bucket:** When a campaign starts, all recipients are moved to a `queue` table with a `pending` status.
* **Polling & Locking:** Each active Worker runs an independent asynchronous loop. It polls the database for the next `pending` row. To prevent race conditions, the engine uses an **Atomic Update (Grab & Lock)**:
    1. Worker finds a `pending` row.
    2. Worker immediately updates the status to `processing` and stamps it with its `Worker_ID`.
    3. Other workers are now ignored for that specific row.
* **Fault Tolerance:** If a Worker crashes or its phone loses internet, the status of its current task remains `processing`. If the system detects a stale process, the task is reverted to `pending` so another Worker can finish the job.

---

## The "Human-Mimic" Strategic Engine

To bypass automated spam filters, the engine generates a unique "Digital Fingerprint" for every single interaction through three layers of randomization.

### 1. The Variable & Shuffle Engine
The system moves away from static templates toward a **Possibility Matrix**.
* **Global Variables:** Users define a library of variables (e.g., `{{greetings}}`). Each variable contains a list of options (e.g., *Hi, Hello, Hey, Greetings*).
* **The Shuffle Logic:** * The engine scans the template for `{{tags}}`.
    * It performs a random selection from the variable’s values for *each* recipient.
    * **Result:** If a template has 4 variables with 4 options each, there are **256 unique versions** of that message. This prevents WhatsApp from detecting the "identical byte-code" signature common in bot broadcasts.

### 2. Non-Linear Delay Logic
Bots are often caught because they send messages at fixed intervals (e.g., every 30 seconds). 
* **Dynamic Wait:** The Worker calculates a new, random integer between the **Min Delay** and **Max Delay** for *every* message. 
* **Result:** The sending rhythm is unpredictable (e.g., 22s, then 39s, then 25s), mimicking the irregular pace of a human manual sender.

### 3. The Batch & Rest Orchestrator
To simulate a natural work-rest cycle, the engine uses a **Threshold-Based Pausing** strategy.
* **Batching:** Users define a **Batch Size** (e.g., 30 messages). The Worker tracks its internal counter.
* **Rest Mode:** Once the limit is hit, the Worker switches to a `resting` state and sets a "Wake-up" timestamp. 
* **Background Idling:** During rest, the Headless Browser remains open but idle, consuming minimal VPS resources. Once the rest period ends, the Worker automatically resumes its polling of the `pending` queue.

---

## Detailed User Workflow

### Step 1: Data Preparation (UI)
1. Navigate to the **Recipients** section.
2. Create a **New Group** (e.g., "August Leads").
3. Paste your list of numbers into the textarea. The backend automatically extracts digits and cleans the input.

### Step 2: Content Personalization
1. Create **Global Variables** in the library (e.g., greetings, intros).
2. Draft your **Message Template** using the variable tags.
3. Use the **Shuffle Preview** button to verify the randomization logic.

### Step 3: Worker Deployment (Adding Workers)
1. Navigate to **Workers** and click "Add Worker."
2. Scan the generated **QR Code** with your phone's WhatsApp.
3. **Strategic Note:** The system uses `LocalAuth` to save your session in the `.wwebjs_auth` directory, ensuring you don't have to re-scan if the server reboots.

### Step 4: Campaign Execution
1. Select your **Recipient Group** and **Template**.
2. Configure **Safety Settings**: Min/Max Delay, Batch Size, and Rest Time.
3. **Start:** Monitor the real-time logs as Workers pull tasks from the queue.

---

## Maintenance & VPS Strategy
* **Zero-Footprint Deletion:** When a Worker is removed, the engine kills the background process and deletes the physical session folder to save disk space.
* **PM2 Resilience:** The engine is managed as a persistent service. If a browser crashes, it is automatically revived to resume the asynchronous loop.

---

### **Strategy Summary Table**

| Component | User Experience (UI) | Strategic Logic (The Hood) |
| :--- | :--- | :--- |
| **Recipient Lists** | Paste messy text into a box. | Regex parsing + SQL indexing + De-duplication. |
| **Messaging** | Write one template with tags. | **Shuffle Engine** creates unique permutations. |
| **Pacing** | Set Min/Max sliders. | **Non-Linear Delay** eliminates bot signatures. |
| **Volume Control** | Set Batch/Rest numbers. | **Batch Orchestrator** mimics human breaks. |
| **Multi-Phone** | Connect multiple devices. | **Asynchronous Pull** load-balances the queue. |
