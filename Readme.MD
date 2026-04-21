### REFLECTION 1

Pada milestone pertama, saya mempelajari cara membuat web server sederhana menggunakan Rust dengan TcpListener yang melakukan binding ke alamat 127.0.0.1:7878. Server ini menerima koneksi dari browser melalui listener.incoming() dan setiap koneksi direpresentasikan sebagai TcpStream. Saya juga menambahkan fungsi handle_connection untuk membaca request HTTP menggunakan BufReader, lalu mengumpulkan baris-baris request hingga mencapai baris kosong yang menandakan akhir header.

Dari hasil percobaan, saya melihat bahwa request dari browser berisi berbagai header seperti User-Agent, Accept, dan lainnya, yang dapat berbeda tergantung browser dan sistem yang digunakan. Namun bagian terpenting tetap sama, yaitu request line seperti GET / HTTP/1.1, yang akan digunakan untuk menentukan response pada tahap berikutnya. Milestone ini membantu saya memahami dasar komunikasi HTTP serta bagaimana server membaca dan memproses request dari client secara langsung tanpa framework.

### REFLECTION 2

![Commit 2 screen capture](/assets/images/commit2.png)

Pada milestone ini, saya memodifikasi fungsi handle_connection agar server tidak hanya menerima request, tetapi juga mengirimkan response berupa halaman HTML ke browser. Server membaca file hello.html menggunakan fs::read_to_string, lalu membentuk response HTTP secara manual dengan menambahkan status line (HTTP/1.1 200 OK), header Content-Length, serta isi HTML sebagai body. Response tersebut kemudian dikirim ke client melalui stream.write_all, sehingga browser dapat merender halaman yang dikirim oleh server.

Saya akhirnya memahami bahwa komunikasi HTTP pada level ini masih dilakukan secara manual tanpa bantuan framework, sehingga semua komponen response harus ditulis dengan benar agar browser dapat memahaminya. Header seperti Content-Length penting agar browser mengetahui ukuran data yang dikirim. Melalui milestone ini, saya menjadi lebih memahami bagaimana server mengubah request menjadi response yang dapat ditampilkan di browser, serta bagaimana Rust digunakan untuk membaca file dan mengirim data melalui network stream secara langsung.