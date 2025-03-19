### **Arsitektur Agno Service**

**Gambaran Umum**
Agno Service adalah microservice yang memisahkan "AI orchestration" untuk customer service agent dari layanan utama agar lebih modular dan scalable. Tujuan utama Agno Service adalah:

1. **Meningkatkan fleksibilitas** dalam pengelolaan AI untuk customer service agent tanpa bergantung langsung pada backend utama Laravel.
2. **Mengurangi beban backend Laravel**, sehingga tugas AI-driven agent processing tidak memperlambat fungsi inti lainnya seperti autentikasi, pembayaran, atau komunikasi WA.
3. **Memudahkan integrasi dengan berbagai model AI**, baik berbasis OpenAI, self-hosted LLM, atau fine-tuned AI lainnya.
4. **Menerapkan arsitektur microservice** agar lebih scalable dan maintainable.

**Di mana saya akan Agno menempatkannya di dalam sistem backend?**
Berdasarkan backend Agen Cerdas saat ini, Agno Service akan saya tempatkan sebagai microservice yang berdiri sendiri dan bertindak sebagai middleware antara backend Laravel dan AI processing.

---

### **Teknologi dan Komponen (Event-Driven Arch)**

| **Komponen**         | **Teknologi**                      | **Fungsi**                                       |
|----------------------|--------------------------------|------------------------------------------------|
| **Event Bus**       | Kafka / RabbitMQ              | Menyediakan event-driven komunikasi antar layanan |
| **API Gateway**     | FastAPI (Python)              | Memproses request dari backend Laravel dan meneruskan event |
| **AI Processing**   | OpenAI, Local LLM (Llama/Mistral), Rasa NLU | Menjalankan inferensi AI dan mengirimkan hasilnya melalui event |
| **Queue Management**| Redis + Celery (Python)       | Mengatur antrian AI processing untuk skalabilitas tinggi |
| **WebSocket**       | FastAPI WebSocket              | Komunikasi real-time dengan backend Laravel     |
| **Logging & Monitoring** | Prometheus + Grafana, MongoDB | Memantau performa AI dan menyimpan history AI logs |

### **Alur Kerja Agno Service (Dengan Event-Driven)**

1. User mengirim pesan ke chatbot melalui WhatsApp atau Webchat.
2. **Mem-publish event** event bus yang akan di-consume oleh service lainnya.
3. Agno Service menerima event tersebut dan memproses pesan menggunakan AI yang telah dikonfigurasi.
4. Jika ada knowledge base yang relevan, Agno mencari jawaban berdasarkan data yang tersedia.
5. Agno mem-publish event berisi hasil AI processing ke event bus, yang kemudian diambil oleh backend Laravel untuk diteruskan ke WhatsApp atau Webchat.
6. Jika AI mengalami kesulitan menjawab, Agno bisa mengarahkan request ke Human Agent secara otomatis dengan event-driven message routing.

---

### **Perbandingan Situasi Saat Ini dan Setelah Perubahan**

| **Fitur**                 | **Backend Saat Ini** (Monolitik) | **Backend Setelah Perubahan** (Event-Driven Microservice) |
|---------------------------|--------------------------------|------------------------------------------------|
| **Request Handling**      | Synchronous API calls         | Event-driven dengan Kafka / RabbitMQ         |
| **AI Processing**         | Langsung diproses di Laravel  | Dijalankan di microservice AI yang terpisah   |
| **Queue Management**      | Redis hanya opsional         | Redis + Celery sebagai core antrian           |
| **WebSocket Communication** | Laravel Reverb (WebSocket) | FastAPI WebSocket + Event Bus                 |
| **Scalability**           | Terbatas karena monolitik    | Horizontal scaling dengan event-driven        |
| **Monitoring & Logging**  | Basic logging dengan Laravel | Prometheus + Grafana + MongoDB log analysis   |
| **Respon AI ke Laravel**  | Direct API response         | Event-driven message passing                  |

---
~ Mas Gendon
