# Modul 9 Software Architecture

## Reflection 1

a. What is amqp?  
amqp (Advanced Message Queuing Protocol) adalah protokol standar terbuka pada application layer untuk message-oriented middleware. Protokol ini memungkinkan client application untuk berkomunikasi dengan message broker (seperti RabbitMQ) secara aman dan efisien.  
b. What does it mean? guest:guest@localhost:5672 , what is the first guest, and what
is the second guest, and what is localhost:5672 is for?  
String tersebut adalah URL koneksi. guest pertama adalah username default, dan guest kedua adalah password default untuk RabbitMQ. localhost:5672 menunjukkan bahwa server RabbitMQ berjalan di komputer lokal (localhost) dan mendengarkan koneksi pada port standar AMQP, yaitu 5672.

## Reflection 2

### Simulation slow subscriber

![Simulation slow subscriber](public/simulation-slow-subscriber.png)

Berdasarkan tangkapan layar, terdapat penumpukan sebanyak 11 pesan di dalam antrean (queued messages). Hal ini terjadi karena adanya simulasi slow subscriber dengan penambahan jeda waktu 1 detik (thread::sleep). Ketika program publisher dijalankan beberapa kali secara cepat, pesan-pesan dikirim ke RabbitMQ secara instan. Namun, karena subscriber hanya mampu memproses maksimal satu pesan per detik, laju konsumsi menjadi jauh lebih lambat dibandingkan laju pengiriman. Akibatnya, sisa pesan yang belum mendapat giliran untuk diproses tertahan dan menumpuk di dalam antrean.

### Reflection and Running at least three subscribers

![Running three subscribers](public/running-three-subscriber-spike.png)
![Running three subscribers](public/running-three-subscriber.png)

Berdasarkan observasi, ketika menjalankan tiga subscriber secara bersamaan, lonjakan antrean (queue spike) di RabbitMQ menurun jauh lebih cepat dan beban pemrosesan pesan splitted ke masing-masing terminal. Hal ini terjadi karena arsitektur Event-Driven pada RabbitMQ mendistribusikan pesan secara round-robin kepada semua consumer yang terhubung ke antrean yang sama. Dengan bertambahnya jumlah worker (concurrency), sistem dapat memproses backlog antrean secara paralel dan jauh lebih efisien.

Hal yang bisa ditingkatkan dari kode (Improvement):

1. Blocker pada Async Runtime, dimana kode subscriber menggunakan std::thread::sleep untuk simulasi delay. Karena program ini menggunakan runtime asinkron (Tokio), penggunaan std::thread::sleep adalah sebuah anti-pattern karena akan block keseluruhan thread OS. Sehingga seharusnya menggunakan tokio::time::sleep agar proses await berjalan secara async dan thread bisa mengerjakan task lain.
2. Hardcoded Configuration: URL koneksi RabbitMQ (amqp://guest:guest@localhost:5672) dan nama antrean hardcoded di dalam kode utama publisher maupun subscriber. Best practice nya adalah menyimpannya di dalam file .env agar sistem lebih fleksibel dan aman saat deployment.
