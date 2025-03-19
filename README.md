### **Arsitektur Agno Service**

**Gambaran Umum**
Agno Service adalah microservice yang memisahkan "AI orchestration" untuk customer service agent dari layanan utama agar lebih modular dan scalable. Tujuan utama Agno Service adalah:

1. **Meningkatkan fleksibilitas** dalam pengelolaan AI untuk customer service agent tanpa bergantung langsung pada backend utama Laravel.
2. **Mengurangi beban backend Laravel**, sehingga tugas AI-driven agent processing tidak memperlambat fungsi inti lainnya seperti autentikasi, pembayaran, atau komunikasi WA.
3. **Memudahkan integrasi dengan berbagai model AI**, baik berbasis OpenAI, self-hosted LLM, atau fine-tuned AI lainnya.
4. **Menerapkan arsitektur microservice** agar lebih scalable dan maintainable.

**Di mana Agno harus ditempatkan dalam sistem backend?**
Berdasarkan backend Agen Cerdas saat ini, Agno Service ditempatkan sebagai microservice yang berdiri sendiri dan bertindak sebagai middleware antara backend Laravel dan AI processing.

---

### **Teknologi dan Komponen**

| **Komponen**         | **Teknologi**                      | **Fungsi**                                       |
|----------------------|--------------------------------|------------------------------------------------|
| API Gateway         | FastAPI (Python)              | Memproses request dari backend Laravel         |
| AI Processing      | OpenAI, Local LLM (Llama/Mistral), Rasa NLU | Menjalankan inferensi AI                     |
| Queue Management   | Redis + Celery (Python)       | Mengatur antrian AI processing                 |
| WebSocket         | FastAPI WebSocket              | Komunikasi real-time dengan backend Laravel    |
| Logging & Monitoring | Prometheus + Grafana, MongoDB | Memantau performa AI dan menyimpan history AI logs |

### **Alur Kerja Agno Service**

1. User mengirim pesan ke chatbot melalui WhatsApp atau Webchat.
2. Backend Laravel menerima request dan meneruskannya ke Agno Service.
3. Agno Service memproses pesan menggunakan AI yang telah dikonfigurasi.
4. Jika ada knowledge base yang relevan, Agno mencari jawaban berdasarkan data yang tersedia.
5. Agno mengembalikan respons ke backend Laravel, yang kemudian meneruskannya ke WhatsApp atau Webchat.
6. Jika AI mengalami kesulitan menjawab, Agno bisa mengarahkan request ke Human Agent secara otomatis.

### **Perbaikan dari Arsitektur Sebelumnya**

1. **Integrasi dengan Backend Laravel Lebih Detail**
   - Komunikasi dapat dilakukan via **REST API atau WebSocket** untuk interaksi real-time.
   - **Autentikasi request** dari Laravel ke Agno dilakukan menggunakan API Key atau JWT.
   - **Format data request/response** sudah ditentukan agar tidak terjadi kesalahan komunikasi.

2. **Peningkatan Logging dan Error Handling**
   - **Strategi retry** jika AI tidak merespons dalam waktu tertentu.
   - Logging mencakup monitoring performa dan debugging error.
   - **Sistem logging tambahan seperti ELK Stack atau Graylog** untuk pencatatan yang lebih mendetail.

3. **Optimasi Pemilihan Model AI**
   - Menambahkan **model registry** untuk menentukan AI model mana yang akan digunakan sesuai skenario.
   - Fleksibilitas memilih **OpenAI, Llama, atau Rasa NLU** berdasarkan kebutuhan dan biaya.

4. **Database AI Log Lebih Terstruktur**
   - MongoDB digunakan untuk menyimpan **user ID, timestamp, model yang digunakan, confidence score, response time, dan logs lainnya**.
   - Jika dibutuhkan analisis mendalam, data bisa diintegrasikan dengan **PostgreSQL atau Elasticsearch**.

5. **Scalability dan Failover**
   - **AI processing dapat dijalankan di server terpisah** untuk mengurangi beban backend utama.
   - **Fallback AI Model**: Jika model utama gagal, request bisa dialihkan ke model cadangan.
   - **Redis Queue Management** memastikan setiap request AI diproses secara optimal.

---
