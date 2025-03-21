Advance Programming
# Module 6 - Concurrency

- Nama    : Akhyar Rasyid Asy syifa
- Kelas   : Advance Programming - A
- NPM     : 2306241682

**reflections shortcut**
- [Commit 1](#milestone-1-single-threaded-web-server)
- [Commit 2](#milestone-2-returning-html)
- [Commit 3](#milestone-3-validating-request-and-selectively-responding)
- [Commit 4](#milestone-4-simulation-of-slow-request)


## Milestone 1: Single-Threaded Web Server

Pada tahap pertama ini, saya telah membuat sebuah **web server sederhana** menggunakan **Rust** yang berjalan dalam **mode single-threaded**. Web server ini terdiri dari dua fungsi utama:

1. **`main()`**: Mengatur **TCP listener** dan menangani koneksi yang masuk  
2. **`handle_connection()`**: Memproses setiap **TCP stream (koneksi)** yang diterima  

---

### Setup TCP Listener

Pertama, saya menetapkan **TCP listener** yang akan menangani koneksi masuk pada port `7878`:

```rust
let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
```

Kode ini berfungsi untuk:
- Membuat **TCP listener** yang terhubung ke `127.0.0.1:7878`  
- Menggunakan `.unwrap()` untuk mengekstrak nilai sukses atau **panic** jika terjadi error (misalnya, jika port sudah digunakan)  

---

### Loop untuk Menangani Koneksi

Setelah **TCP listener** siap, saya membuat loop yang akan menangani setiap koneksi masuk:

```rust
for stream in listener.incoming() {
    let stream = stream.unwrap();
    
    handle_connection(stream);
}
```

Penjelasan:
- `listener.incoming()` mengembalikan **iterator** untuk setiap percobaan koneksi yang masuk  
- `stream.unwrap()` digunakan untuk menangani koneksi yang berhasil dibuat  
- Setiap koneksi yang diterima akan diproses oleh fungsi `handle_connection(stream)`  

Karena ini masih dalam mode **single-threaded**, server akan menangani satu koneksi **secara berurutan** sebelum menerima koneksi berikutnya.  

---

### Memproses Request dalam `handle_connection()`

function `handle_connection()` digunakan untuk membaca **request dari browser** dan mencetaknya ke terminal:

```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    println!("Request: {:#?}", http_request);
}
```

penjelasan langkah-langkahnya:

1. **Membuat Buffered Reader dari TCP Stream**  
   ```rust
   let buf_reader = BufReader::new(&mut stream);
   ```
   - **BufReader** digunakan untuk membaca data dari **TCP stream** dengan lebih efisien  

2. **Membaca dan Memproses Header HTTP**  
   ```rust
   let http_request: Vec<_> = buf_reader
       .lines()
       .map(|result| result.unwrap())
       .take_while(|line| !line.is_empty())
       .collect();
   ```
   - `.lines()`: Mengembalikan **iterator** untuk setiap baris dalam request  
   - `.map(|result| result.unwrap())`: Mengambil string dari **Result**, atau **panic** jika ada error  
   - `.take_while(|line| !line.is_empty())`: Menghentikan pembacaan jika menemukan baris kosong (**akhir dari header HTTP**)  
   - `.collect()`: Mengumpulkan semua **header request** ke dalam sebuah **vector**  

3. **Mencetak Request ke Terminal**  
   ```rust
   println!("Request: {:#?}", http_request);
   ```
   - Menampilkan request dalam format yang lebih **rapi dan mudah dibaca** (`{:#?}`)  

---

### Hasil/output yang Didapat

Saat saya menjalankan server dan mengakses `127.0.0.1:7878` dari browser, terminal mencetak **header HTTP request** seperti ini:

```
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "Connection: keep-alive",
    "Cache-Control: no-cache",
    "sec-ch-ua: \"Chromium\";v=\"134\", \"Not:A-Brand\";v=\"24\", \"Google Chrome\";v=\"134\"",
    "sec-ch-ua-mobile: ?0",
    "sec-ch-ua-platform: \"Windows\"",
    "Upgrade-Insecure-Requests: 1",
    "User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7",
    "Accept-Encoding: gzip, deflate, br, zstd",
    "Accept-Language: en-US,en;q=0.9,id-ID;q=0.8,id;q=0.7",
]
```

dari output yang dihasilkan di terminal saya tsb, saya melihat bahwa browser mengirim **request GET** ke server dengan berbagai informasi tambahan seperti **User-Agent, Host, Accept-Encoding, dll.**  

---

## Milestone 2: Returning HTML
### Screenshot
![Commit 2 Screenshot](/assets/images/commit2.jpg)

setelah saya menangani _connection_ dan mencetak request HTTP di terminal pada milestone 1 tadi, langkah selanjutnya di milestone 2 ini adalah mengirimkan respons HTML agar dapat ditampilkan di browser.

### Modifikasi `handle_connection()`
saya memodifikasi fungsi `handle_connection()` agar dapat membaca file HTML dan mengirimkannya sebagai respons HTTP.

### Perubahan dalam `main.rs`
```rust
use std::{
    fs,
    io::{prelude::*, BufReader},
    net::{TcpListener, TcpStream},
};

fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();

    let status_line = "HTTP/1.1 200 OK"; 
    let contents = fs::read_to_string("hello.html").unwrap(); 
    let length = contents.len();
    let response = format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");
    stream.write_all(response.as_bytes()).unwrap();
}
```

### Struktur File HTML
saya juga membuat file `hello.html` yang berisi halaman sederhana:
```html
<!DOCTYPE html>
 <html lang="en">
     <head>
         <meta charset="utf-8">
         <title>Hello!</title> 
     </head> 
     <body>
         <h1>Hello!</h1> 
         <p>Hi from Rust, running from Akhyar’s machine.</p>
     </body>
 </html>
```

### Penjelasan perubahan di main cara bisa membaca html nya
1. **Membaca File HTML**
   - menggunakan `fs::read_to_string("hello.html")` untuk membaca konten file HTML.
   
2. **Membentuk Respons HTTP**
   - status HTTP 200 OK ditetapkan untuk menunjukkan bahwa permintaan berhasil.
   - header `Content-Length` digunakan untuk memberi tahu browser ukuran konten yang dikirimkan.
   - konten HTML kemudian dikirim dalam respons.

3. **Menulis Respons ke Stream**
   - `stream.write_all(response.as_bytes()).unwrap();` memastikan data dikirim ke browser.


## Milestone 3: Validating Request and Selectively Responding

![Commit 3 screen capture](/assets/images/commit3.jpg)

pada milestone 3 ini, saya meningkatkan web server agar dapat memvalidasi permintaan yang masuk dan memberikan respons yang sesuai. sekarang web-server bisa menampilkan halaman lain yang berbeda dan bisa juga me-return page 404 kalau requestnya tidak sesuai.

Pada pembaruan ini, server hanya bisa mengekstrak request line pertama yang berisi metode HTTP, path, dan versi, dibanding mengumpulkan semua header HTTP. Saya juga melakukan refactoring dengan menggunakan `match` untuk menangani permintaan dengan lebih efisien. Jika request menuju root path (GET / HTTP/1.1), server bakal mengembalikan `hello.html` dengan status 200 OK. sedangkan untuk rute lainnya, server mengembalikan `404.html` dengan status 404 NOT FOUND. Dengan pendekatan ini, server bisa menampilkan halaman yang sesuai berdasarkan request sehingga memberikan umpan balik yang lebih baik kepada pengguna. intinya sekarang, server dapat menangani permintaan, menentukan respons berdasarkan path, dan memberikan halaman yang sesuai tanpa kehilangan efisiensi dalam model single-threaded.

### Manfaat Refactoring dengan `match`:
1. **Menghindari kode berulang** yang terjadi jika menggunakan `if-else`.
2. **Memastikan semua variabel terinisialisasi dengan benar**, membantu compiler Rust dalam validasi kode.
3. **Kode lebih bersih dan mudah diperluas** dengan menambahkan case baru tanpa mengulang logika yang sama.
4. **Menggunakan variabel immutable** (`let` tanpa `mut`), meningkatkan keamanan dalam pemrosesan paralel.


## Milestone 4: Simulation of slow request

Pada tahap ini, saya menambahkan penanganan untuk permintaan ke URI `/sleep`. Jika ada pengguna yang mengakses `/sleep`, server akan "tidur" selama 10 detik sebelum mengembalikan respons. Karena server masih berjalan secara single-threaded, permintaan lain yang masuk dalam periode tersebut harus menunggu hingga permintaan sebelumnya selesai diproses. Masalah ini muncul karena fungsi handle_connection menangani setiap permintaan secara berurutan. Ketika ada permintaan yang membutuhkan waktu lama untuk dieksekusi, seperti `/sleep`, seluruh permintaan lain ikut tertunda. Hal ini menunjukkan bahwa server single-threaded memiliki keterbatasan dalam menangani banyak permintaan sekaligus. Secara teknis, server bakal bekerja dengan memproses koneksi secara berurutan dari `listener.incoming()`. Karena hanya ada satu thread yang aktif, setiap koneksi harus selesai sebelum koneksi berikutnya dapat diproses. Dengan menambahkan `thread::sleep()`, saya mensimulasikan operasi yang membutuhkan waktu lama, seperti pemrosesan data yang kompleks atau pemanggilan API eksternal.

dari yang sudah saya lakukan ini, saya paham kalau hal ini menunjukkan mengapa server single-threaded kurang efisien dalam menangani beban tinggi dan bagaimana model multi-threading atau async menjadi solusi untuk meningkatkan performa.