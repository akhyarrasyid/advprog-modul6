Advance Programming
# Module 6 - Concurrency

- Nama    : Akhyar Rasyid Asy syifa
- Kelas   : Advance Programming - A
- NPM     : 2306241682


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







