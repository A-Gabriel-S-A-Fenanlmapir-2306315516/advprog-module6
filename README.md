Commit 1 Reflection NotesDalam tahap ini, saya membuat fungsi handle_connection untuk menangani pembacaan request dari browser. Berikut adalah poin-poin penting yang saya pelajari dari kode tersebut:BufReader & Streams: Fungsi ini menggunakan BufReader untuk membungkus TcpStream. Hal ini dilakukan agar proses pembacaan data dari stream menjadi lebih efisien dengan menyimpannya di memori sementara (buffer) sebelum diproses lebih lanjut.Parsing HTTP Request: Baris kode .lines().map(|result| result.unwrap()).take_while(|line| !line.is_empty()).collect() bertugas untuk mengambil seluruh baris request yang dikirimkan oleh browser.lines() membagi stream menjadi baris-baris teks berdasarkan karakter newline.take_while(|line| !line.is_empty()) sangat krusial karena protokol HTTP menandakan akhir dari bagian header request dengan baris kosong. Tanpa ini, program akan terus menunggu data dan "nyangkut".Logging: Hasil request kemudian dicetak ke konsol menggunakan println! dengan format {:#?} agar struktur datanya terlihat rapi (pretty-print) di terminal.Dengan langkah ini, server saya sekarang sudah bisa "mendengar" dan "membaca" apa yang diminta oleh klien (browser), meskipun belum memberikan balasan balik berupa halaman HTML.# Reflection - Module 6: Concurrency

**Nama:** Gabriel Selwas Aboyaman Fenanlampir  
**NPM:** 2306315516  
**Kelas:** Adpro - A  

---

## Commit 1 Reflection Notes

Dalam tahap ini, saya membuat fungsi `handle_connection` untuk menangani pembacaan request dari browser. Berikut adalah poin-poin penting yang saya pelajari dari kode tersebut:

* **BufReader & Streams**: Fungsi ini menggunakan `BufReader` untuk membungkus `TcpStream` . Hal ini dilakukan agar proses pembacaan data dari stream menjadi lebih efisien dengan menyimpannya di memori sementara (buffer) sebelum diproses lebih lanjut.
* **Parsing HTTP Request**: Baris kode `.lines().map(|result| result.unwrap()).take_while(|line| !line.is_empty()).collect()` bertugas untuk mengambil seluruh baris request yang dikirimkan oleh browser 1427, .
    * `lines()` membagi stream menjadi baris-baris teks berdasarkan karakter *newline*.
    * `take_while(|line| !line.is_empty())` sangat krusial karena protokol HTTP menandakan akhir dari bagian *header* request dengan baris kosong. Tanpa ini, program akan terus menunggu data dan mengakibatkan terminal "hang" karena tidak tahu kapan request berakhir.
* **Logging**: Hasil request dicetak ke konsol menggunakan `println!` dengan format `{:#?}` agar struktur datanya terlihat rapi (pretty-print) dan mudah dianalisis di terminal.

Dengan langkah ini, server saya sekarang sudah bisa mendengarkan (*listening*) dan membaca data yang dikirimkan oleh klien melalui socket TCP .

---

## Commit 2 Reflection Notes

Pada Milestone ini, saya memodifikasi fungsi `handle_connection` agar server dapat memberikan respon balik berupa halaman visual . Berikut adalah hal-hal yang saya pelajari:

* **HTTP Response Structure**: Sebuah respon HTTP yang valid harus memiliki format standar agar bisa dirender oleh browser. Format tersebut terdiri dari *Status Line* (contoh: `HTTP/1.1 200 OK`), *Headers* (seperti `Content-Length`), dan *Message Body* (isi konten file) .
* **File System (`fs`)**: Saya menggunakan modul `std::fs` untuk membaca file `hello.html` dari penyimpanan lokal . Fungsi `fs::read_to_string` mengubah isi file tersebut menjadi `String` di dalam Rust.
* **Response Formatting**: Penggunaan makro `format!` mempermudah penggabungan metadata respon . Misalnya, `length` digunakan untuk memberi tahu browser berapa besar data yang akan diterima melalui header `Content-Length` .
* **Stream Writing**: Dengan menggunakan `stream.write_all()`, seluruh string respon yang telah diubah menjadi bytes dikirimkan kembali melalui koneksi TCP ke klien.
* **Visualisasi**: Sekarang, ketika mengakses `http://127.0.0.1:7878`, browser tidak lagi kosong tetapi menampilkan halaman HTML yang berisi pesan "Hi from Rust" .

![Commit 2 screen capture](/assets/images/commit2.png)



## Commit 3 Reflection Notes

Pada Milestone 3 ini, saya mengimplementasikan validasi request agar server dapat memberikan respon yang berbeda tergantung pada path yang diminta pengguna:

* **Validasi Request**: Saya menggunakan `request_line` untuk mengecek baris pertama dari HTTP request. Jika baris tersebut adalah `GET / HTTP/1.1`, maka server menganggapnya valid. Jika tidak, server akan mengembalikan status 404.
* **Refactoring**: Awalnya saya menggunakan blok `if` dan `else` yang memanggil fungsi baca file dan kirim respon di kedua cabangnya. Hal ini menyebabkan duplikasi kode. Saya melakukan refactoring dengan memisahkan logika penentuan status dan nama file ke dalam variabel `(status_line, filename)`.
* **Keuntungan Refactoring**: Kode menjadi lebih bersih, mudah dibaca, dan meminimalisir kesalahan jika nanti ada perubahan pada cara server mengirimkan respon (cukup mengubah satu tempat saja di akhir fungsi).
* **Efisiensi**: Sekarang server tidak lagi selalu memberikan file yang sama, melainkan sudah bisa menangani "halaman tidak ditemukan" dengan mengirimkan `404.html` .

![Commit 3 screen cap(/assets/images/commit3.png)

---

## Commit 4 Reflection Notes

Berdasarkan percobaan simulasi *slow response* di atas, berikut adalah analisis saya:

* **Sifat Single-Threaded**: Server ini masih bekerja secara sekuensial (satu per satu). Artinya, server hanya bisa menangani satu request dalam satu waktu.
* **Blocking Mechanism**: Ketika ada request ke path `/sleep`, server menjalankan `thread::sleep` selama 10 detik. Selama waktu tersebut, *main thread* server benar-benar berhenti bekerja dan tidak bisa memproses request lain yang masuk.
* **Dampaknya pada User Experience**: Request kedua (ke path utama `/`) terpaksa mengantri di belakang request `/sleep`. Meskipun request kedua seharusnya sangat cepat, ia ikut tertahan (terblokir) karena server sedang sibuk mengerjakan request pertama yang lama.
* **Kesimpulan**: Model *single-thread* sangat tidak efisien untuk aplikasi dunia nyata di mana banyak pengguna mengakses server secara bersamaan. Jika satu pengguna melakukan proses berat, pengguna lainnya akan mengalami *delay* yang tidak perlu .


---

## Commit 5 Reflection Notes

Pada Milestone 5, saya telah mengimplementasikan `ThreadPool` untuk membuat server menjadi multi-threaded. Berikut adalah poin-poin yang saya pelajari:

* **Mekanisme ThreadPool**: Alih-alih membuat thread baru tanpa batas untuk setiap request (yang bisa memicu DoS), `ThreadPool` mengelola sekumpulan thread yang jumlahnya sudah ditentukan sejak awal (pre-allocated).
* **MPSC Channel**: Saya menggunakan `mpsc` (Multiple Producer, Single Consumer) untuk mengirimkan "tugas" (Job) dari main thread ke worker threads. Main thread bertindak sebagai producer yang mengirimkan request, sementara worker threads bertindak sebagai consumers yang mengambil tugas tersebut dari antrian.
* **Arc dan Mutex**: Karena banyak worker thread yang perlu mengakses receiver yang sama dari channel secara bergantian, saya menggunakan `Arc` (Atomic Reference Counting) untuk berbagi kepemilikan receiver dan `Mutex` untuk memastikan hanya satu worker yang bisa mengambil tugas di satu waktu (mutual exclusion) agar tidak terjadi race condition .
* **Efisiensi**: Sekarang, server dapat menangani request lain (seperti halaman utama) meskipun ada thread lain yang sedang tertahan oleh proses `/sleep` selama 10 detik.

---

## Commit Bonus Reflection Notes

Untuk bagian bonus, saya melakukan perbaikan pada fungsi inisialisasi `ThreadPool`:

* **Fungsi `build` sebagai pengganti `new`**: Saya menambahkan fungsi `pub fn build(size: usize) -> Result<ThreadPool, &'static str>`.
* **Error Handling**: Berbeda dengan fungsi `new` yang menggunakan `assert!` (yang akan memicu `panic!` jika input salah), fungsi `build` mengembalikan `Result`. Jika user memasukkan jumlah thread `0`, fungsi akan mengembalikan `Err`, sehingga program utama bisa menangani error tersebut dengan lebih elegan tanpa harus langsung crash.
* **Penerapan**: Ini mengikuti praktik terbaik di Rust untuk membiarkan pemanggil fungsi memutuskan bagaimana menangani kegagalan inisialisasi daripada mematikan seluruh program secara paksa.