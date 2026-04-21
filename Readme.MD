### REFLECTION 1

Pada milestone pertama, saya mempelajari cara membuat web server sederhana menggunakan Rust dengan TcpListener yang melakukan binding ke alamat 127.0.0.1:7878. Server ini menerima koneksi dari browser melalui listener.incoming() dan setiap koneksi direpresentasikan sebagai TcpStream. Saya juga menambahkan fungsi handle_connection untuk membaca request HTTP menggunakan BufReader, lalu mengumpulkan baris-baris request hingga mencapai baris kosong yang menandakan akhir header.

Dari hasil percobaan, saya melihat bahwa request dari browser berisi berbagai header seperti User-Agent, Accept, dan lainnya, yang dapat berbeda tergantung browser dan sistem yang digunakan. Namun bagian terpenting tetap sama, yaitu request line seperti GET / HTTP/1.1, yang akan digunakan untuk menentukan response pada tahap berikutnya. Milestone ini membantu saya memahami dasar komunikasi HTTP serta bagaimana server membaca dan memproses request dari client secara langsung tanpa framework.

### REFLECTION 2

![Commit 2 screen capture](/assets/images/commit2.png)

Pada milestone ini, saya memodifikasi fungsi handle_connection agar server tidak hanya menerima request, tetapi juga mengirimkan response berupa halaman HTML ke browser. Server membaca file hello.html menggunakan fs::read_to_string, lalu membentuk response HTTP secara manual dengan menambahkan status line (HTTP/1.1 200 OK), header Content-Length, serta isi HTML sebagai body. Response tersebut kemudian dikirim ke client melalui stream.write_all, sehingga browser dapat merender halaman yang dikirim oleh server.

Saya akhirnya memahami bahwa komunikasi HTTP pada level ini masih dilakukan secara manual tanpa bantuan framework, sehingga semua komponen response harus ditulis dengan benar agar browser dapat memahaminya. Header seperti Content-Length penting agar browser mengetahui ukuran data yang dikirim. Melalui milestone ini, saya menjadi lebih memahami bagaimana server mengubah request menjadi response yang dapat ditampilkan di browser, serta bagaimana Rust digunakan untuk membaca file dan mengirim data melalui network stream secara langsung.

### REFLECTION 3

![Commit 3 screen capture](/assets/images/commit3.png)

Pada milestone ini, saya memodifikasi server agar tidak selalu mengembalikan hello.html untuk semua request. Saya membaca baris pertama dari HTTP request, lalu menggunakannya untuk menentukan response yang sesuai. Jika request line adalah GET / HTTP/1.1, server akan mengembalikan hello.html dengan status 200 OK. Jika request tidak sesuai, server akan mengembalikan 404.html dengan status 404 NOT FOUND.

Saya juga memahami pentingnya refactoring pada bagian pemilihan response agar kode lebih jelas dan mudah dirawat. Dengan memisahkan status_line dan filename berdasarkan hasil pengecekan request, alur program menjadi lebih rapi dibandingkan jika semua response ditulis terpisah secara berulang. Milestone ini membantu saya memahami bahwa server harus memvalidasi request terlebih dahulu sebelum menentukan halaman yang dikirim, serta menunjukkan bagaimana konsep response selection bekerja pada web server sederhana.

### REFLECTION 4

![Commit 4 screen capture](/assets/images/commit4.png)

Pada milestone ini, saya menambahkan simulasi request lambat dengan membuat route /sleep yang akan menunda response selama 10 detik menggunakan thread::sleep(Duration::from_secs(10)). Dengan penambahan ini, server tetap mengembalikan hello.html, tetapi secara sengaja dibuat lambat agar saya dapat mengamati perilaku server ketika menangani request yang membutuhkan waktu lama.

Dari percobaan menggunakan dua tab browser, saya melihat bahwa ketika satu tab mengakses /sleep, request lain ke / juga ikut tertunda. Hal ini terjadi karena server masih berjalan secara single-threaded, sehingga hanya dapat menangani satu request dalam satu waktu. Milestone ini membantu saya memahami secara langsung bahwa server satu thread tidak efisien untuk banyak pengguna, karena satu request lambat dapat memblokir request lainnya. Inilah alasan mengapa pada tahap berikutnya dibutuhkan server multithreaded.

### REFLECTION 5

![Commit 5.1 screen capture](/assets/images/commit5.png)
![Commit 5.2 screen capture](/assets/images/commit52.png)


Pada milestone ini, saya mengimplementasikan server multithreaded menggunakan ThreadPool untuk mengatasi keterbatasan server single-threaded yang sebelumnya menyebabkan blocking antar request. Dengan pendekatan ini, setiap koneksi yang masuk tidak langsung diproses oleh main thread, tetapi dikirim sebagai job melalui channel ke worker thread yang tersedia. Worker kemudian mengambil job tersebut dan mengeksekusinya secara paralel, sehingga beberapa request dapat diproses secara bersamaan tanpa saling menunggu.

Saya juga memahami bahwa penggunaan Arc<Mutex<Receiver<Job>>> memungkinkan beberapa worker berbagi akses terhadap satu sumber job secara aman, sementara constraint FnOnce + Send + 'static memastikan closure dapat dipindahkan dan dijalankan di thread lain tanpa masalah ownership maupun lifetime. Dari hasil pengujian, terlihat bahwa request ke endpoint /sleep tidak lagi memblokir request lain selama masih ada worker yang tersedia. Hal ini menunjukkan bahwa concurrency secara signifikan meningkatkan responsivitas server, terutama dalam menangani request yang bersifat lambat atau blocking. Namun, jumlah worker yang terbatas tetap menjadi bottleneck jika jumlah request melebihi kapasitas thread pool.

### BONUS

Pada bagian bonus ini, saya mengimplementasikan fungsi build sebagai alternatif dari new dalam pembuatan ThreadPool. Berbeda dengan new yang menggunakan assert! dan akan langsung menyebabkan panic ketika ukuran thread pool tidak valid, fungsi build mengembalikan Result<ThreadPool, String>. Dengan pendekatan ini, kesalahan dapat ditangani dengan lebih aman dan fleksibel oleh pemanggil fungsi, tanpa langsung menghentikan program secara paksa.

Perbandingan ini menunjukkan bahwa penggunaan Result lebih sesuai dengan prinsip error handling dalam Rust yang menghindari panic sebisa mungkin. Dengan build, program menjadi lebih robust karena dapat mengantisipasi input yang tidak valid dan memberikan respon yang lebih terkontrol. Hal ini juga mencerminkan praktik yang lebih baik dalam pengembangan sistem nyata, di mana kegagalan tidak selalu harus menghentikan keseluruhan aplikasi, tetapi dapat ditangani secara elegan.