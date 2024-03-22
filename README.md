## Reflection 1

#### Commit 1 Reflection
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

#### Commit 2 Reflection
Commit 2 Screen Capture
![Commit 2 screen capture](/assets/images/commit2.png)

Baris-baris tambahan dalam function handle_connection berfungsi untuk mengirimkan kembali HTTP response ke client. Berikut ini adalah deskripsi dari setiap baris tambahan tersebut.
```rust
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