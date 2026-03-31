# BambangShop Publisher App
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
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [X] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Subscriber model struct.`
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [x] Commit: `Implement add function in Subscriber repository.`
    -   [x] Commit: `Implement list_all function in Subscriber repository.`
    -   [x] Commit: `Implement delete function in Subscriber repository.`
    -   [x] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [x] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [x] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [x] Commit: `Implement publish function in Program service and Program controller.`
    -   [x] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [x] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1
**1. In the Observer pattern diagram explained by the Head First Design Pattern book, Subscriber is defined as an interface. Explain based on your understanding of Observer design patterns, do we still need an interface (or trait in Rust) in this BambangShop case, or a single Model struct is enough?**
- Sebuah single model `struct` sudah cukup dan tidak perlu menggunakan `trait`. Akan butuh `trait` jika ada berbagai jenis subscriber dengan perilaku yang berbeda. Namun, karena semua subscriber memiliki perilaku yang sama persis yaitu semuanya menerima notifikasi melalui HTTP POST ke URL yang mereka miliki, tidak ada variasi tipe subscriber yang membutuhkan implementasi berbeda dari method `update()`.

**2. id in Program and url in Subscriber is intended to be unique. Explain based on your understanding, is using Vec (list) sufficient or using DashMap (map/dictionary) like we currently use is necessary for this case?**
- Menggunakan `DashMap` is necessary for this case karena id pada product dan url pada subscriber unik, maka butuh struktur data yang bisa menjamin keunikan dan mempermudah operasi lookup, insert, dan delete berdasarkan key. 
- Dengan `DashMap`, operasi menghapus satu subscriber berdasarkan URL bisa dilakukan dalam O(1). Sedangkan, dengan `Vec` harus iterate seluruh list sehingga kompleksitasnya adalah O(n).

**3. When programming using Rust, we are enforced by rigorous compiler constraints to make a thread-safe program. In the case of the List of Subscribers (SUBSCRIBERS) static variable, we used the DashMap external library for thread safe HashMap. Explain based on your understanding of design patterns, do we still need DashMap or we can implement Singleton pattern instead?**
- Tetap butuh `DashMap` dan singleton pattern tidak bisa menggantikan nya. Keduanya menangani masalah yang berbeda:
    - Singleton pattern memastikan hanya ada satu instance dari `SUBSCRIBERS` yang digunakan di seluruh app. Masalahnya adalah, singleton tidak menyelesaikan masalah thread safety saat terjadi akses bersamaan read/write. 
    - `DashMap` dibutuhkan karena aplikasi berjalan secara multi-threaded, sehingga banyak thread bisa mengakses `SUBSCRIBERS` secara bersamaan. 
- Intinya: singleton (`lazy_static!`) dan `DashMap` harus bekerja bersama yaitu singleton untuk memastikan satu instance, `DashMap` untuk thread safety pada akses yang concurrent.

#### Reflection Publisher-2
**1. In the Model-View Controller (MVC) compound pattern, there is no “Service” and “Repository”. Model in MVC covers both data storage and business logic. Explain based on your understanding of design principles, why we need to separate “Service” and “Repository” from a Model?**
- Pemisahan ini intinya supaya tiap bagian punya tanggung jawab yang sesuai dengan prinsip Single Responsibility Principle (SRP). jika semua di model, satu file harus menangani logika bisnis dan akses data sekaligus padahal keduanya bisa dimodify karena alasan yang berbeda. Dengan adanya repository yang khusus menangani akses data dan service yang khusus menangani logika bisnis, jika misal mau mengganti dari in memory storage ke database yang sungguhan, cukup ubah repository nya aja tanpa harus ubah bagian lain, berlaku juga sebaliknya jika logika bisnisnya yang berubah. Hasilnya, kode jadi lebih mudah untuk dimaintain.

**2. What happens if we only use the Model? Explain your imagination on how the interactions between each model (Program, Subscriber, Notification) affect the code complexity for each model?**
- Tiap model jadi harus menangani logika bisnisnya dan akses datanya sendiri. Akan bermasalah ketika antar model saling berinteraksi. Product perlu tahu soal subscriber untuk kirim notifikasi, subscriber perlu tahu soal notification untuk bentuk payload, dst. 
- Ketiga model jadi saling bergantung satu sama lain, sehingga mengubah satu model akan berdampak ke model lainnya. Makin banyak fitur yang ditambahkan, makin tercampur kodenya, dan unit testing pun akan jadi susah. Jadinya kompleksitas dari kode meningkat.

**3. Have you explored more about Postman? Tell us how this tool helps you to test your current work. You might want to also list which features in Postman you are interested in or feel like it is helpful to help your Group Project or any of your future software engineering projects.**
- Postman sangat membantu, karena saya jadi bisa langsung menguji endpoint yang sudah dibuat tanpa harus bikin frontend nya dulu. Lansung bisa kirim HTTP request ke endpoint aplikasi dan bisa langsung lihat response body, status code, dan juga header nya. 
- Fitur Collections juga sangat berguna untuk menyimpan dan organize request yang sering dipakai, sementara environment variables memudahkan penggantian base URL saat berpindah antara environment development dan production. 
- Di tutorial ini saya menggunakan postman untuk menguji endpoint subscribe, unsubscribe, dan publish produk, sehingga bisa langsung verifikasi apakah observer pattern nya berjalan sesuai dengan harapan.

#### Reflection Publisher-3
**1. Observer Pattern has two variations: Push model (publisher pushes data to subscribers) and Pull model (subscribers pull data from publisher). In this tutorial case, which variation of Observer Pattern that we use?**
- Menggunakan push model. Terlihat dari cara kerja `NotificationService::notify()`, tiap kali ada event seperti product created, deleted, atau promotion, publisher langsung mendorong data notifikasi yang lengkap ke setiap subscriber via HTTP POST. Subscriber tidak perlu meminta ataupun mengambil data apapun, semua sudah dikirimkan langsung oleh publisher.

**2. What are the advantages and disadvantages of using the other variation of Observer Pattern for this tutorial case? (example: if you answer Q1 with Push, then imagine if we used Pull)**
- Keuntungan menggunakan pull model:
    - Subscriber bisa mengambil only data yang mereka butuhkan saja, tidak ada data yang terkirim sia-sia.
    - Publisher jadi lebih simpel karena tidak perlu menyiapkan payload lengkap
    - Subscriber punya kendali lebih atas kapan mereka mau mengambil datanya sehingga bisa menghindari overload.
- Kerugian menggunakan pull model:
    - Subscriber harus melakukan HTTP request ke publisher, yang artinya menambahkan kompleksitas di sisi subscriber dan jumlah request secara keseluruhan.
    - Ada risiko data sudah berubah lagi ketika subscriber baru melakukan pull, sehingga data yang didapat bisa tidak sinkron dengan aslinya.
    - Model ini kurang cocok untuk tutorial ini karena ada jeda antara event terjadi dan subscriber mengetahuinya.

**3. Explain what will happen to the program if we decide to not use multi-threading in the notification process.**
- Pengiriman notifikasi ke setiap subscriber jadi akan berjalan secara sequential (satu per satu). Program harus menunggu response HTTP dari subscriber pertama selesai sebelum lanjut ke subscriber yang selanjutnya. Itu akan bermasalah, karena jika ada satu subscriber yang lambat merespons atau timeout, seluruh proses pengiriman notifikasi akan ikut tertunda. Lebih parah, request utama dari user seperti create product tidak akan selesai sampai semua notifikasi terkirim, sehingga response time bisa jadi sangat lama. Jika subscriber nya banyak, aplikasi bisa terasa lagging dari sudut pandang pengguna. 