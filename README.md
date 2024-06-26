## Module 6 Reflection

### Commit 1 Reflection
Function handle_connection berfungsi untuk menangani data dari sebuah TCP stream. Pertama-tama, kita perlu mengimpor std::io::prelude dan std::io::BufReader untuk mengakses semua resources yang sekiranya diperlukan dalam function ini. Setelah itu, kita akan membuat instance BufReader yang memiliki parameter reference dari stream.
```rust
let buf_reader = BufReader::new(&mut stream);
```

Berikut ini adalah deskripsi dari bagian kode berikutnya.
```rust
let http_request: Vec<_> = buf_reader
    .lines()
    .map(|result| result.unwrap())
    .take_while(|line| !line.is_empty())
    .collect();
```
- Variabel `http_request` dibuat untuk menyimpan baris-baris berupa data HTTP request yang dikirimkan oleh browser. Baris-baris ini akan disimpan dalam sebuah object vector.
- `lines()` berfungsi untuk membagi data stream berdasarkan newline atau \n.
- Kemudian, `map(|result| result.unwrap())` digunakan untuk mengubah setiap result menjadi String. 
- Adanya `take_while(|line| !line.is_empty())` membuat program terus mengambil baris dari iterator selama belum ditemukan baris yang kosong.
- `collect()` akan mengumpulkan semua baris yang sudah diolah tadi ke object vector yang sudah dibuat.

Terakhir, isi dari variabel `http_request` ini akan di-print untuk kemudian ditampilkan ke terminal.

---

### Commit 2 Reflection
Commit 2 Screen Capture
![Commit 2 screen capture](/assets/images/commit2.png)

Baris-baris tambahan dalam function handle_connection berfungsi untuk mengirimkan kembali HTTP response ke client. Berikut ini adalah deskripsi dari setiap baris tambahan tersebut.
```rust
    ...
    let status_line = "HTTP/1.1 200 OK";
    let contents = fs::read_to_string("hello.html").unwrap();
    let length = contents.len();

    let response = format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");

    stream.write_all(response.as_bytes()).unwrap();
```
- `HTTP/1.1 200 OK` digunakan untuk membuat HTTP response yang menunjukkan bahwa HTTP request berhasil diproses.
- `read_to_string` dari fs akan membaca isi dari file hello.html dan mengubahnya menjadi String.
- Kemudian, akan dihitung length dari `contents` yang sebelumnya sudah diisi dengan isi dari file hello.html. Nilai length ini nantinya akan digunakan untuk menentukan nilai Content-Length dalam header HTTP response.
- Sebuah HTTP response lengkap yang terdiri dari header dan body akan disusun. HTTP response ini akan terdiri atas status line, header Content-Length, serta file hello.html sebagai body.
- Dengan `stream.write_all(response.as_bytes())`, HTTP response yang telah disusun ini akan dikonversi ke dalam bentuk byte dan ditulis ke stream.

---

### Commit 3 Reflection
Commit 3 Screen Capture
![Commit 3 screen capture](/assets/images/commit3.png)

Pada tahap ini, ada penambahan file error.html yang merupakan halaman default untuk menangani HTTP request yang tidak ditemukan. Selain itu, saya juga melakukan perubahan pada function handle_connection untuk memvalidasi HTTP request serta memberikan HTTP response yang sesuai. 
Berikut ini adalah perubahan baru pada function handle_connection.
```rust
    ...
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
        ("HTTP/1.1 200 OK", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND", "error.html")
    };
    ...
```
- Pertama, program akan mengecek path yang di-request oleh browser dan menyimpannya di variabel `request_line`. Secara sederhana, baris kode ini berfungsi untuk mengakses request line yang menjadi baris pertama dari HTTP request.
- Setelah itu, pada bagian berikutnya, saya memanfaatkan conditional untuk menetapkan nilai dari variabel status_line serta filename berdasarkan nilai `request_line`.
- Program akan menyusun HTTP response berdasarkan nilai status_line dan filename yang sudah ditentukan. Dengan begitu, program dapat memberikan HTTP response yang sesuai.

Perubahan kode yang akhirnya saya commit sudah melewati tahap refactoring. Sebagai gambaran, berikut ini adalah pseudocode dari function handle_connection yang baru sebelum melewati tahap refactoring.
```rust
    ...
    let request_line = buf_reader.lines().next().unwrap().unwrap();

    if request_line == "GET / HTTP/1.1" {
        let status_line = "HTTP/1.1 200 OK";
        // susun HTTP response dengan body hello.html
        stream.write_all(response.as_bytes()).unwrap();

    } else {
        let status_line = "HTTP/1.1 404 NOT FOUND";
        // susun HTTP response dengan body error.html
        stream.write_all(response.as_bytes()).unwrap();
    }
    ...
```
Jika dibandingkan, dapat dilihat bahwa function handle_connection sebelum melewati tahap refactoring memiliki banyak duplicated code. Hal ini tentunya dapat mengurangi code maintainability. Oleh karena itu, dilakukan refactoring pada function ini.

---

### Commit 4 Reflection
Berikut ini adalah perubahan baru pada function handle_connection.
```rust
    ...
    let (status_line, filename) = match &request_line[..] {
        "GET / HTTP/1.1" => ("HTTP/1.1 200 OK", "hello.html"),
        "GET /sleep HTTP/1.1" => {
            thread::sleep(Duration::from_secs(10));
            ("HTTP/1.1 200 OK", "hello.html")
        }
        _ => ("HTTP/1.1 404 NOT FOUND", "error.html"),
    };
    ...
```
- Jika sebelumnya potongan kode di atas menggunakan conditional, potongan kode ini sekarang menggunakan `match` untuk menentukan nilai status_file dan filename berdasarkan path yang di-request oleh browser.
- Selain itu, ditambahkan request line baru, yakni `GET /sleep HTTP/1.1` untuk menangani endpoint /sleep. Jika program menerima HTTP request dengan request line ini, program akan menjalankan `thread::sleep(Duration::from_secs(10))` untuk memberikan efek loading selama 10 detik sebelum memberikan HTTP response yang sesuai. 

Dengan mengakses `http://127.0.0.1:7878/` dan `http://127.0.0.1:7878/sleep` di saat yang sama, kita dapat membuat simulasi slow request. Pada simulasi ini, dapat dilihat bagaimana server yang berbasis single-threaded menangani lebih dari 1 HTTP request di saat yang sama. Ternyata, dalam skala yang lebih besar, penggunaan server berbasis single-threaded sangat tidak efisien karena memerlukan waktu yang lebih lama untuk memproses lebih dari 1 HTTP request.

---

### Commit 5 Reflection
Untuk membuat server berbasis multi-threaded, kita dapat membuat sebuah ThreadPool. Sebagai analogi, kita perlu membuat Worker (Thread) yang bertugas untuk "mengerjakan" suatu Job (misalnya HTTP request atau task tertentu). Ketika ThreadPool menerima suatu Job, ThreadPool akan mengirimkan sinyal ke Receiver milik salah satu Worker. Worker yang menerima sinyal tersebut kemudian akan me-lock Receiver-nya serta "mengerjakan" Job yang diberikan. Setelah Job tersebut selesai, Worker yang bekerja akan kembali me-unlock Receiver-nya. Singkatnya, mekanisme ini diperlukan untuk memastikan hanya ada 1 Worker yang bekerja sehingga mencegah konflik dalam pengerjaan Job. Dengan cara ini, sistem kerja berbasis multi-threaded dapat diimplementasikan secara efektif.