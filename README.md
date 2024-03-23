# Tutorial 6 Advanced Programming
__Reza Apriono__ </br>
__2206827945__</br>
__AdPro B - KVN__</br>

## Reflection
### Commit 1 Reflection Notes
Method `handle_connection` berfungsi mengatur behaviour server saat mendapatkan connection request dari browser. Hal-hal yang dilakukan diantaranya membaca data dari `TcpStream`, mengelola http request yang diterima, dan memberikan output berupa data dari `TcpStream` dan http request tersebut. Langkah-langkah yang dilakukan antara lain:
1. Method `handle_connection` menerima parameter `stream` dengan tipe `TCPstream` yang mutable. Parameter ini mewakili koneksi TCP yang diterima dari client.
```rust
fn handle_connection(mut stream: TcpStream)
```

2. Membuat sebuah instance `buffreader` dengan wrapping reference mutable ke `stream` untuk melakukan buffering dan pembacaan input dari `TcpStream` dengan efisien.
```rust
let buf_reader = BufReader::new(&mut stream);
```

3. Membuat vektor `http_request` yang berisi baris-baris permintaan HTTP yang diterima dari client yang telah dibaca oleh reader dan telah di-buffer. Proses ini menggunakan method `lines()` dari `BufReader` untuk membuat iterator yang mengembalikan setiap baris dari stream sebagai `Result<String, std::io::Error>`. Kemudian, dengan menggunakan `map()`, setiap baris di-unwrapping untuk mengekstrak isi baris, dan dilanjutkan dengan `take_while()` untuk mengambil baris-baris sampai baris kosong ditemukan. Hasilnya kemudian dikumpulkan ke dalam `Vec<_>` dengan menggunakan `collect()`.
```rust
let http_request: Vec<_> = buf_reader 
        .lines() 
        .map(|result| result.unwrap())
        take_while(|line| !line.is_empty()) 
        .collect();
```

4. Terakhir, baris-baris dari isi HTTP request akan diprint ke terminal dengan menggunakan `println!()`.
```rust
println!("Request: {:#?}", http_request);
```