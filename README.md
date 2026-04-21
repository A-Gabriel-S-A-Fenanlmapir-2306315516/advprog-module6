Commit 1 Reflection NotesDalam tahap ini, saya membuat fungsi handle_connection untuk menangani pembacaan request dari browser. Berikut adalah poin-poin penting yang saya pelajari dari kode tersebut:BufReader & Streams: Fungsi ini menggunakan BufReader untuk membungkus TcpStream. Hal ini dilakukan agar proses pembacaan data dari stream menjadi lebih efisien dengan menyimpannya di memori sementara (buffer) sebelum diproses lebih lanjut.Parsing HTTP Request: Baris kode .lines().map(|result| result.unwrap()).take_while(|line| !line.is_empty()).collect() bertugas untuk mengambil seluruh baris request yang dikirimkan oleh browser.lines() membagi stream menjadi baris-baris teks berdasarkan karakter newline.take_while(|line| !line.is_empty()) sangat krusial karena protokol HTTP menandakan akhir dari bagian header request dengan baris kosong. Tanpa ini, program akan terus menunggu data dan "nyangkut".Logging: Hasil request kemudian dicetak ke konsol menggunakan println! dengan format {:#?} agar struktur datanya terlihat rapi (pretty-print) di terminal.Dengan langkah ini, server saya sekarang sudah bisa "mendengar" dan "membaca" apa yang diminta oleh klien (browser), meskipun belum memberikan balasan balik berupa halaman HTML.# Reflection - Module 6: Concurrency

**Nama:** Gabriel Selwas Aboyaman Fenanlampir  
**NPM:** 2306315516  
**Kelas:** Adpro - A  

---

## Commit 1 Reflection Notes

[cite_start]Dalam tahap ini, saya membuat fungsi `handle_connection` untuk menangani pembacaan request dari browser[cite: 1431]. Berikut adalah poin-poin penting yang saya pelajari dari kode tersebut:

* [cite_start]**BufReader & Streams**: Fungsi ini menggunakan `BufReader` untuk membungkus `TcpStream`[cite: 1416, 1426]. [cite_start]Hal ini dilakukan agar proses pembacaan data dari stream menjadi lebih efisien dengan menyimpannya di memori sementara (buffer) sebelum diproses lebih lanjut[cite: 1426].
* [cite_start]**Parsing HTTP Request**: Baris kode `.lines().map(|result| result.unwrap()).take_while(|line| !line.is_empty()).collect()` bertugas untuk mengambil seluruh baris request yang dikirimkan oleh browser[cite: 1426, 1427, 1428].
    * [cite_start]`lines()` membagi stream menjadi baris-baris teks berdasarkan karakter *newline*[cite: 1427].
    * [cite_start]`take_while(|line| !line.is_empty())` sangat krusial karena protokol HTTP menandakan akhir dari bagian *header* request dengan baris kosong[cite: 1428]. Tanpa ini, program akan terus menunggu data dan mengakibatkan terminal "hang" karena tidak tahu kapan request berakhir.
* [cite_start]**Logging**: Hasil request dicetak ke konsol menggunakan `println!` dengan format `{:#?}` agar struktur datanya terlihat rapi (pretty-print) dan mudah dianalisis di terminal[cite: 1429].

[cite_start]Dengan langkah ini, server saya sekarang sudah bisa mendengarkan (*listening*) dan membaca data yang dikirimkan oleh klien melalui socket TCP[cite: 1404, 1432].

---

## Commit 2 Reflection Notes

[cite_start]Pada Milestone ini, saya memodifikasi fungsi `handle_connection` agar server dapat memberikan respon balik berupa halaman visual[cite: 1453, 1454]. Berikut adalah hal-hal yang saya pelajari:

* [cite_start]**HTTP Response Structure**: Sebuah respon HTTP yang valid harus memiliki format standar agar bisa dirender oleh browser[cite: 1274]. [cite_start]Format tersebut terdiri dari *Status Line* (contoh: `HTTP/1.1 200 OK`), *Headers* (seperti `Content-Length`), dan *Message Body* (isi konten file)[cite: 1468, 1472].
* [cite_start]**File System (`fs`)**: Saya menggunakan modul `std::fs` untuk membaca file `hello.html` dari penyimpanan lokal[cite: 1457, 1469]. [cite_start]Fungsi `fs::read_to_string` mengubah isi file tersebut menjadi `String` di dalam Rust[cite: 1469].
* [cite_start]**Response Formatting**: Penggunaan makro `format!` mempermudah penggabungan metadata respon[cite: 1471, 1472]. [cite_start]Misalnya, `length` digunakan untuk memberi tahu browser berapa besar data yang akan diterima melalui header `Content-Length`[cite: 1470, 1472].
* [cite_start]**Stream Writing**: Dengan menggunakan `stream.write_all()`, seluruh string respon yang telah diubah menjadi bytes dikirimkan kembali melalui koneksi TCP ke klien[cite: 1472].
* [cite_start]**Visualisasi**: Sekarang, ketika mengakses `http://127.0.0.1:7878`, browser tidak lagi kosong tetapi menampilkan halaman HTML yang berisi pesan "Hi from Rust"[cite: 1482, 1511].

![Commit 2 screen capture](/assets/images/commit2.png)