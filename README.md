## Reflection 1

#### (1) Commit 1 Reflection
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
- Kemudian, `map(|result| result.unwrap())` digunakan untuk "me-unwrap" setiap result menjadi String. 
- Adanya `take_while(|line| !line.is_empty())` membuat program terus mengambil baris dari iterator selama belum ditemukan baris yang kosong.
- `collect()` akan mengumpulkan semua baris yang sudah diolah tadi ke object vector yang sudah dibuat.

Terakhir, isi dari variabel `http_request` ini akan di-print untuk kemudian ditampilkan ke terminal.