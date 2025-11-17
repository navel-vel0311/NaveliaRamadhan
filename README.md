<img width="795" height="622" alt="image" src="https://github.com/user-attachments/assets/0b75577e-d5f1-4192-88ec-77ce86a7921e" />1. Jelaskan teorema CAP dan BASE dan keterkaitan keduanya. Jelaskan menggunakan contoh yang pernah anda gunakan.
Jawab:
1. Teorema CAP dan BASE serta Keterkaitannya
Teorema CAP (Brewer's Theorem) menyatakan batasan fundamental pada sistem terdistribusi mana pun: kita hanya dapat menjamin dua dari tiga properti: Konsistensi (C), Ketersediaan (A), dan Toleransi Partisi (P). Karena Toleransi Partisi (P)—kemampuan sistem untuk terus beroperasi meskipun terjadi kegagalan komunikasi antar node—adalah prasyarat mutlak dalam sistem terdistribusi, pilihan praktisnya adalah memilih antara CP atau AP. Sistem yang memilih CP (Consistency dan Partition Tolerance), seperti basis data relasional atau MongoDB yang dikonfigurasi ketat, akan mengorbankan ketersediaan dengan cara memblokir permintaan selama partisi jaringan terjadi demi menjaga semua data selalu identik. Di sisi lain, sistem yang memilih AP (Availability dan Partition Tolerance), seperti Cassandra, akan mengorbankan konsistensi segera demi ketersediaan tinggi, di mana sistem terus melayani permintaan respons meskipun data mungkin berbeda antar node untuk sementara waktu.
Model BASE adalah filosofi desain yang timbul sebagai konsekuensi logis dari pilihan AP dalam Teorema CAP. BASE mengutamakan Basically Available (Ketersediaan Dasar) dan menerima bahwa status sistem akan berada dalam Soft State (Keadaan Fleksibel) yang berubah seubah-ubah, dengan harapan akan mencapai Eventually Consistent (Konsisten Akhirnya). Dengan kata lain, data pada akhirnya akan sinkron setelah partisi diselesaikan. Oleh karena itu, keterkaitan keduanya sangat erat: Model BASE adalah implementasi praktis dari trade-off AP yang didikte oleh keterbatasan Teorema CAP.
2. Keterkaitan antara GraphQL dengan Komunikasi Antar Proses pada Sistem Terdistribusi
GraphQL berfungsi sebagai lapisan antarmuka yang sangat efisien yang menjembatani klien dengan layanan backend dalam arsitektur sistem terdistribusi, khususnya microservices. Meskipun GraphQL sendiri adalah protokol komunikasi antara klien dan server gateway, peran utamanya adalah memicu Komunikasi Antar Proses (IPC - Inter-Process Communication) secara internal di dalam backend. Ketika klien mengirimkan satu kueri GraphQL yang kompleks, server GraphQL bertindak sebagai aggregator atau API Gateway yang cerdas. Server ini kemudian melakukan beberapa panggilan IPC secara paralel (biasanya menggunakan HTTP/REST, gRPC, atau message queue) ke berbagai microservices (misalnya, Layanan Autentikasi, Layanan Produk, Layanan Pembayaran) untuk mengambil bagian data yang berbeda. Kueri tunggal dari klien dipecah menjadi beberapa panggilan IPC internal, dan hasilnya dikumpulkan kembali, diolah, dan dikirimkan kembali ke klien sebagai satu respons terstruktur yang minimalis (menghindari over-fetching). Dengan demikian, GraphQL menyederhanakan komunikasi klien-ke-layanan sekaligus mengoptimalkan lalu lintas IPC internal di dalam sistem terdistribusi.
3. Streaming Replication di PostgreSQL dengan Docker Compose
Streaming Replication adalah mekanisme yang digunakan PostgreSQL untuk menyediakan redundancy data dan failover dengan menjaga salinan database sekunder (Replica) tetap sinkron dengan database primer (Primary). Untuk mendemonstrasikan sinkronisasi ini menggunakan Docker Compose, kita memerlukan dua container: db_primary dan db_replica.
Langkah-langkah Pengerjaan:
Konfigurasi Primary: Pada layanan db_primary, kita mengaktifkan fitur replikasi dengan mengatur wal_level=replica dan max_wal_senders di Docker Compose.
Inisialisasi User Replikasi: Skrip init_primary.sh dijalankan untuk membuat user khusus (misalnya, replicator) dengan izin REPLICATION yang dibutuhkan oleh Replica untuk mengambil stream data.
Base Backup Replica: Layanan db_replica akan menunggu Primary siap (healthy). Kemudian, skrip init_replica.sh menjalankan utilitas pg_basebackup untuk menyalin data awal dari Primary dan secara otomatis membuat file standby.signal dan konfigurasi koneksi (postgresql.auto.conf).
Aktivasi Streaming: Ketika Replica dimulai, adanya file standby.signal membuatnya memasuki mode standby. Ia kemudian membuka koneksi ke Primary sebagai WAL Receiver.
Sinkronisasi: Pada Primary, setiap transaksi yang berkomitmen dicatat dalam Write-Ahead Log (WAL). Proses WAL Sender secara real-time mengirimkan catatan WAL ini ke WAL Receiver di Replica. Replica kemudian mengaplikasikan (mereplay) catatan WAL tersebut ke datanya, memastikan bahwa semua perubahan yang terjadi pada Primary segera tercermin pada Replica.
Verifikasi: Dengan menjalankan docker-compose up -d, kita dapat memverifikasi bahwa insert data pada db_primary langsung terlihat melalui select pada db_replica, menegaskan bahwa sinkronisasi streaming berhasil.
2. Jelaskan keterkaitan antara GraphQL dengan komunikasi
Jawab:
Keterkaitan GraphQL dengan Komunikasi Antar Proses di Sistem Terdistribusi
GraphQL bukan protokol komunikasi antar proses seperti gRPC, RabbitMQ, atau REST antar microservices.
Namun, GraphQL sering digunakan sebagai lapisan antarmuka (API Gateway) untuk mengorkestrasi komunikasi antar proses di sistem terdistribusi.
Peran GraphQL dalam Sistem Terdistribusi
1. Single Entry Point
Klien hanya mengakses satu endpoint GraphQL, sementara server GraphQL-lah yang berkomunikasi dengan banyak layanan di belakangnya.
2. Orkestrasi Data (Data Orchestration)
Resolver GraphQL dapat memanggil microservice lain, message broker, database, atau komponen lain dalam sistem terdistribusi.
3. Abstracting Inter-Process Communication
Walaupun sistem di belakang menggunakan:
-REST antar layanan
-gRPC
-event/message broker (Kafka, RabbitMQ)
-direct socket communication
… klien tidak perlu tahu, karena GraphQL menyembunyikan kompleksitas tersebut.
4. Mengurangi Over-fetching & Under-fetching
Klien dapat meminta data sesuai kebutuhan, meskipun data berasal dari banyak proses atau microservice berbeda.
5. Schema Federation / Apollo Federation
Mengizinkan banyak service terdistribusi mempublikasikan bagian skema GraphQL masing-masing, lalu digabungkan menjadi satu graph yang besar.
Diagram Hubungan GraphQL & Komunikasi Antar Proses
Berikut diagram sederhana dalam bentuk teks:
                +-----------------------------+
                |           Client            |
                |   (Web/Mobile/Backend)      |
                +--------------+--------------+
                               |
                               | GraphQL Query
                               v
                    +----------+-----------+
                    |      GraphQL API     |
                    | (Gateway / Resolver) |
                    +----------+-----------+
                               |
       -----------------------------------------------------
       |                         |                         |
       v                         v                         v
+-------------+         +---------------+         +----------------+
|  Service A  | <-----> |  Service B    | <-----> |   Service C    |
| (REST/gRPC) |         |(Event-driven) |         | (Database I/O) |
+-------------+         +---------------+         +----------------+
       |                         |                         |
       -----------------------------------------------------
                               |
                               v
                    +---------------------------+
                    |       Data Aggregation    |
                    +---------------------------+
                               |
                               v
                    +---------------------------+
                    |     GraphQL Response      |
                    +---------------------------+
                               |
                               v
                        Returned to Client
Interpretasi Diagram
-Client hanya berinteraksi dengan GraphQL API.
-GraphQL API memiliki resolver yang berfungsi sebagai jembatan komunikasi antar proses.
-GraphQL dapat memanggil:
-Service A (misalnya lewat REST atau gRPC)
-Service B (misalnya event-driven seperti Kafka)
-Service C (akses database atau service lain)
-GraphQL kemudian menggabungkan hasil dari layanan-layanan tersebut menjadi satu respons yang sesuai permintaan klien.
3. Dengan menggunakan Docker / Docker Compose, buatlah streaming replication di PostgreSQL yang bisa menjelaskan sinkronisasi. Tulislah langkah-langkah pengerjaannya dan buat penjelasan secukupnya.
Jawab:
Streaming Replication di PostgreSQL Windows 
https://www.crunchydata.com/blog/postgres-streaming-replication-on-windows-a-quick-guide
Persiapan Awal
Aktifkan Docker Desktop: 
Struktur Direktori dan FIle



