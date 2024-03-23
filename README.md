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

### Commit 2 Reflection Notes
![Commit 2 screen capture](/assets/images/commit2.png)

Method `handle_connection` mengalami beberapa penambahan kode pada commit ke-2 ini. Penambahan kode tersebut berfungsi agar method juga dapat mengirim response ke client terhadap HTTP request yang diterima. Proses ini dibuat dengan cara sebagai berikut.
1. Membuat variable `status_line` yang berisi status line HTTP dan menyatakan bahwa request berhasil diproses dengan kode "200 OK".
```rust
let status_line = "HTTP/1.1 200 OK";
```

2. Membaca content dari file `hello.html` dan mengonversinya menjadi string untuk kemudian disimpan di variable `contents`.
```rust
let contents = fs::read_to_string("hello.html").unwrap();
```

3. Menghitung panjang `contents` untuk disertakan dalam header pada response.
```rust
let length = contents.len();
```

4. Membuat HTTP response yang lengkap dimana mencakup status line, header, dan contents. 
```rust
let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");
```

5. Response yang telah dibuat kemudian dikirim kembali ke client melalui `stream` yang sama.
```rust
stream.write_all(response.as_bytes()).unwrap();
```

### Commit 3 Reflection Notes
![Commit 3 screen capture](/assets/images/commit3.png)

Pada awalnya, website memberikan response yang sama terhadap semua request dari client, yakni dengan mereturn content HTML dari file `hello.html`. Sekarang, website perlu menangani respons yang berbeda, sehingga perlu dilakukan pemeriksaan terhadap `request_line` untuk memisahkan dan memberikan response yang sesuai. 

Perubahan pada commit ini terdapat pada method `handle_connection`, yang isinya menjadi seperti ini:
```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    if request_line == "GET / HTTP/1.1" {
        let status_line = "HTTP/1.1 200 OK";
        let contents = fs::read_to_string("hello.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    } else {
        let status_line = "HTTP/1.1 404 NOT FOUND";
        let contents = fs::read_to_string("404.html").unwrap();
        let length = contents.len();

        let response = format!(
            "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
        );

        stream.write_all(response.as_bytes()).unwrap();
    }
}
```

Pada kode tersebut, response dibagi menjadi 2 kondisi, yakni ketika menuju ke path `/` dan ke path lain selain path tersebut. Jika request_line sesuai dengan GET request dari path yang diinginkan (`/`), content dari file `hello.html` akan dikirimkan sebagai response. Jika tidak sesuai (selain path `/`), maka content dari `file 404.html` akan dikirim sebagai response dan menandakan bahwa path yang direquest tidak ditemukan.

Setelah membagi menjadi 2 kondisi, saya menyadari bahwa diperlukan refactoring untuk kode yang telah dibuat. Refactoring diperlukan karena conditional block (if-else) pada kode tersebut mengulangi proses pembacaan file HTML dan penulisan contentnya ke dalam `stream`. Seharusnya, conditional (if-else) bisa diterapkan hanya untuk mengecek `request_line`. Kemudian, untuk membaca file dan mengirim contentnya ke dalam `stream` dilakukan setelah pengecekan berdasarkan conditional tersebut sesuai dengan value dari `request_line`, sehingga kode menjadi lebih terstruktur dan tidak memiliki duplikasi.

Method `handle_connection` setelah dilakukan refactoring akan menjadi seperti ini:
```rust
fn handle_connection(mut stream: TcpStream) {
    let buf_reader = BufReader::new(&mut stream);
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "404.html")
    };

    let contents = fs::read_to_string(filename).unwrap();
    let length = contents.len();

    let response =
        format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
}
```

### Commit 4 Reflection Notes
Setelah memodifikasi block conditional (if else) menjadi match dalam method `handle_connection`, kita dapat membandingkan `request_line` dengan pola yang ditentukan. Sebagai contoh single-threaded, endpoint `/sleep` bisa ditambahkan sehingga menyebabkan delay atau penundaan process selama 10 detik menggunakan `thread::sleep(Duration::from_secs(10))`. Dalam skenario ini, akses secara bersamaan ke` 127.0.0.1` dan `127.0.0.1/sleep` akan mengakibatkan delay pada response dari `/sleep`, yang juga menunda response untuk request lain. Hal ini terjadi karena thread sleep menghentikan eksekusi thread untuk sementara waktu. Oleh karena itu, penggunaan single-threaded bisa mengakibatkan keterlambatan dan penundaan dalam memproses request di server.

### Commit 5 Reflection Notes
Kita dapat membuat sistem multithreading dengan menciptakan sebuah ThreadPool. ThreadPool merupakan kumpulan thread di mana setiap thread siap untuk menangani sebuah request. Kemudian, kita memerlukan sebuah Worker yang memiliki id dan threadnya masing-masing pada program untuk menerima dan menjalankan pekerjaan yang lebih spesifik. Kita dapat menghubungkan ThreadPool dan Worker dengan membuat kode baru yang memungkinkan ThreadPool untuk mengirim pesan atau sinyal melalui sender ke receiver yang telah di-clone dan diberikan kepada setiap Worker. Dengan melakukan hal ini, ketika ThreadPool menerima sebuah request, sinyal akan dikirim dan menugaskan request tersebut ke Worker yang valid yang kemudian akan memproses request tersebut. Worker kemudian mengunci receiver untuk memproses data sampai selesai, dan baru setelah itu kunci akan dibuka yang memungkinkan Worker lain untuk menerima pekerjaan lain.

### Commit Bonus Reflection Notes
Penggunaan `build` dalam melakukan pembuatan thread lebih menjamin jika dilihat dari segi kemampuan error handling dibandingkan dengan penggunaan `new`, karena jika jumlah yang diberikan <= 0 maka program akan me-return error.