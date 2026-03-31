# BambangShop Receiver App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a Rocket web framework skeleton that you can work with.

As this is an Observer Design Pattern tutorial repository, you need to implement a feature: `Notification`.
This feature will receive notifications of creation, promotion, and deletion of a product, when this receiver instance is subscribed to a certain product type.
The notification will be sent using HTTP POST request, so you need to make the receiver endpoint in this project.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Receiver" folder.

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    ROCKET_PORT=8001
    APP_INSTANCE_ROOT_URL=http://localhost:${ROCKET_PORT}
    APP_PUBLISHER_ROOT_URL=http://localhost:8000
    APP_INSTANCE_NAME=Safira Sudrajat
    ```
    Here are the details of each environment variable:
    | variable                | type   | description                                                     |
    |-------------------------|--------|-----------------------------------------------------------------|
    | ROCKET_PORT             | string | Port number that will be listened by this receiver instance.    |
    | APP_INSTANCE_ROOT_URL   | string | URL address where this receiver instance can be accessed.       |
    | APP_PUUBLISHER_ROOT_URL | string | URL address where the publisher instance can be accessed.       |
    | APP_INSTANCE_NAME       | string | Name of this receiver instance, will be shown on notifications. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)
3.  To simulate multiple instances of BambangShop Receiver (as the tutorial mandates you to do so),
    you can open new terminal, then edit `ROCKET_PORT` in `.env` file, then execute another `cargo run`.

    For example, if you want to run 3 (three) instances of BambangShop Receiver at port `8001`, `8002`, and `8003`, you can do these steps:
    -   Edit `ROCKET_PORT` in `.env` to `8001`, then execute `cargo run`.
    -   Open new terminal, edit `ROCKET_PORT` in `.env` to `8002`, then execute `cargo run`.
    -   Open another new terminal, edit `ROCKET_PORT` in `.env` to `8003`, then execute `cargo run`.

## Mandatory Checklists (Subscriber)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create SubscriberRequest model struct.`
    -   [ ] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Notification repository.`
    -   [ ] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 3: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Commit: `Implement receive_notification function in Notification service.`
    -   [ ] Commit: `Implement receive function in Notification controller.`
    -   [ ] Commit: `Implement list_messages function in Notification service.`
    -   [ ] Commit: `Implement list function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections

#### Reflection Subscriber-1
1. Penggunaan mekanisme sinkronisasi seperti RwLock atau Mutex sangat diperlukan di Rust saat kita memiliki variabel statis yang bisa diubah (mutable) karena variabel tersebut akan diakses oleh banyak thread (dalam hal ini, setiap request yang masuk ke server Rocket berjalan di thread yang berbeda). Tanpa ini, akan terjadi data race yang bisa membuat program crash.

Kita memilih RwLock (Read-Write Lock) alih-alih Mutex karena alasan efisiensi:

- RwLock memungkinkan banyak thread untuk membaca data secara bersamaan selama tidak ada yang menulis. Ini sangat cocok untuk kasus NotificationRepository di mana fungsi list_all_as_string (operasi baca) kemungkinan akan dipanggil lebih sering daripada fungsi add (operasi tulis).

- Mutex bersifat jauh lebih kaku karena hanya mengizinkan satu thread saja yang mengakses data, baik itu untuk membaca maupun menulis. Jika kita memakai Mutex, maka saat satu orang sedang melihat daftar notifikasi, orang lain tidak bisa mendaftarkan notifikasi baru, yang mana hal ini akan menghambat performa aplikasi.

2. Di Java, variabel statis disimpan di heap dan pengelolaannya diserahkan ke JVM, namun Java tidak menjamin secara compile-time bahwa akses ke variabel tersebut aman dari thread lain (kita harus manual menambahkan synchronized).

Di Rust:
- Variabel static harus memiliki ukuran yang pasti saat compile-time dan harus bersifat thread-safe (Sync).

- Mengubah variabel statis secara langsung dianggap unsafe karena Rust tidak bisa menjamin bahwa tidak ada dua thread yang mengubah data yang sama di waktu yang bersamaan.

- lazy_static digunakan karena ia melakukan inisialisasi variabel pada saat runtime (ketika pertama kali diakses), bukan compile-time. Dengan membungkusnya dalam lazy_static dan struktur sinkronisasi (RwLock/DashMap), kita memberikan jaminan kepada compiler bahwa akses ke data global tersebut dikelola dengan aman secara thread-safe, sesuai dengan aturan ownership dan borrowing Rust.

#### Reflection Subscriber-2
1. Jujur saya belum mengeksplorasi isi dari src/lib.rs secara mendalam. Alasan utamanya adalah karena saya memilih untuk fokus sepenuhnya pada instruksi yang ada di modul tutorial guna memastikan implementasi Observer Pattern di bagian Model, Service, Repository, dan Controller berjalan dengan benar sesuai kriteria penilaian. Karena materi ini cukup kompleks, saya memprioritaskan pemahaman alur kerja utama aplikasi (proses subscribe dan notify) sebelum mempelajari file konfigurasi atau modul pendukung lainnya yang ada di luar langkah-langkah wajib.

2. 
- Menambah Subscriber: Observer pattern sangat memudahkan penambahan subscriber karena adanya Loose Coupling. Publisher tidak perlu tahu detail internal dari setiap Receiver; selama Receiver memiliki endpoint yang sesuai, kita bisa menambah puluhan instance baru tanpa mengubah satu baris kode pun di Main App.

- Menambah Main App (Multiple Publishers): Untuk saat ini, menjalankan lebih dari satu Main App tidak akan mudah. Hal ini dikarenakan daftar subscriber disimpan di dalam memori (DashMap di dalam RAM). Jika kita punya dua Main App, subscriber yang mendaftar ke App A tidak akan diketahui oleh App B. Agar bisa mendukung banyak instance Main App, kita harus memindahkan penyimpanan subscriber ke database terpusat (seperti PostgreSQL) atau menggunakan message broker (seperti Redis).

3. Ya, saya mencoba mengeksplorasi fitur Postman. Fitur ini sangat membantu dalam memverifikasi respon API secara otomatis tanpa harus melakukan pengecekan manual satu per satu. Penggunaan Postman Collection juga sangat bermanfaat untuk dokumentasi yang bisa dibagikan kepada anggota tim lain (terutama tim Frontend). Hal ini sangat krusial dalam pengerjaan Proyek Kelompok (PPL) agar integrasi antar bagian aplikasi bisa berjalan lebih cepat dan meminimalisir kesalahan komunikasi terkait format data.