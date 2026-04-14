# KIRIM WA

[English](#english) | [Bahasa Indonesia](#bahasa-indonesia)

---

<a name="english"></a>
## English

This system utilizes a **Database-Driven Asynchronous Architecture** to orchestrate complex campaigns across multiple accounts simultaneously.

### Core Architecture: The "Global Pool" Logic
The system is built on a **Decoupled Architecture**, where the list of recipients is separate from the phones (Workers), allowing a "Pull-Based" distribution.

* **Database-Driven Groups:** Numbers are sanitized via Regex and de-duplicated automatically on import.
* **Asynchronous Worker Pull:** Workers "compete" for tasks. They poll the database for `pending` rows and "lock" them to prevent double-sending.
* **Fault Tolerance:** If a worker crashes, the task reverts to `pending` so others can finish the job.

### The "Human-Mimic" Strategic Engine
* **Unique Identities:** Each worker is assigned a unique **User Agent**. You can also assign an optional **Proxy** per worker to ensure each phone operates from a unique IP.
* **Non-Linear Timing (Min/Max Delay):** Between every message, the worker waits for a random number of seconds based on your **Min and Max Delay** settings. This removes the predictable rhythm that triggers bot detection.
* **Intelligent Validation:** Before sending, the system checks if a number is registered on WhatsApp. Non-registered numbers are automatically skipped.
* **Presence Simulation:** Workers set status to "Online" and simulate a **Typing Delay** based on the specific message length before hitting send.
* **Shuffle Engine (Spintax):** Uses variable tags to create unique permutations for every single message.
* **Batch, Rest & Health Check:** Workers take breaks after a set **Batch Size**. During "Rest Mode," the system performs a **Health Check** to ensure the session is still active and connected.

---

<a name="bahasa-indonesia"></a>
## Bahasa Indonesia

Sistem ini menggunakan **Arsitektur Asinkron Berbasis Database** memastikan  data dan antrian pesan tidak terganggu ketika server atau workers mengalami gangguan seperti terputus dari jaringan, gagal tersinkron dengan handphone yang terhubung. Status Antrian pesan akan kembali ke `pending` dan di pegang alih oleh workers yang masih aktif.

### Arsitektur Inti: Logika "Global Pool"
Sistem ini menggunakan **Arsitektur Terdekopel**, di mana daftar penerima terpisah dari ponsel (Worker), memungkinkan distribusi berbasis "Pull".

* **Grup Berbasis Database:** Daftar nomor di sanitasi dengan Regex untuk mencegah diduplikasi atau nomor tidak valid secara otomatis saat diimpor.
* **Sistem Pull Worker Asinkron:** Worker secara bergantian mengirimkan pesan. Mereka mengambil antrian dengan status `pending` dari database dan "menguncinya" agar tidak diambil worker lain dan menghindari pengiriman ganda.
* **Crash Handling:** Jika worker crash, tugas dikembalikan ke status `pending` agar bisa diselesaikan oleh worker lain.

### Mencegah Worker Teridentifikasi sebagai oleh WhatsApp.
* **User Agent/Browser:** Worker berjalan di atas Headless Browder dengan Puppeteer.Setiap worker memiliki **User Agent** berbeda (Chrome Windows/Mozilla Linux/Mac OS). User dapat menambahkan **Proxy** per worker agar setiap worker beroperasi dari IP yang berbeda. Jika tidak menambahkan proxy, worker akan mengirim pesan melalui IP Server. Sangat disarankan Worker memiliki Proxy.
* **waktu non-linear (Jeda Min/Max):** Di antara setiap pesan, worker menunggu selama beberapa detik secara acak berdasarkan pengaturan **Jeda Minimal dan Maksimal**. Ini menghapus pola kaku berulang yang biasanya memicu deteksi bot.
* **Validasi Nomor:** Sebelum mengirim, sistem mengecek apakah nomor terdaftar di WhatsApp. Nomor yang tidak terdaftar akan otomatis dilewati.
* **Online Status:** Worker mengirimkan status "Online" dan mensimulasikan **Mengetik** berdasarkan panjang pesan sebelum mengirim.
* **Shuffle Engine (Spintax):** Menggunakan tag variabel untuk membuat template unik di setiap pesan.
* **Batch, Rest & Health Check:** Worker akan berhenti mengirimkan pesan setelah mencapai **Batch Size** tertentu. dan beralih ke "Rest Mode", sistem melakukan **Health Check** untuk memastikan sesi tetap aktif.

---

### **Strategy Summary Table**

| Component | UI Action | Strategic Logic (The Hood) |
| :--- | :--- | :--- |
| **Identities** | Add Worker | Unique User Agents + Optional Proxy support. |
| **Recipients** | Paste messy text. | Regex cleaning + SQL indexing + De-duplication. |
| **Validation** | Automatic. | Pre-send check to skip non-registered numbers. |
| **Humanizing** | Automatic. | Typing Delay (Length-based) + Online Status. |
| **Pacing** | Min/Max Delay. | **Randomized intervals** to eliminate bot signatures. |
| **Endurance** | Batch/Rest settings. | Rest Mode with integrated Health Checks. |
| **Multi-Phone** | Connect devices. | Asynchronous Pull load-balances the queue. |

