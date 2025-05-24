# Tugas-3 Pemrograman Jaringan (C) - 5025221138

### Farrel Akmalazmi Nugraha
### 5025221138

---

### Soal 1 
-  Upload File
#### File yang diunggah harus dikodekan terlebih dahulu ke dalam format Base64 sebelum dikirim.
-  Hapus File
#### Implementasikan fitur untuk menghapus file yang sudah ada di server.
--- 
### Penambahan PROKOTOL.TXT
```
UPLOAD
* TUJUAN: Mengunggah file dari client ke server.
* PARAMETER:
  - PARAMETER1: nama file
  - PARAMETER2: isi file dalam base64
* RESULT:
- BERHASIL:
  - status: OK
  - data: pesan sukses
- GAGAL:
  - status: ERROR
  - data: pesan kesalahan

DELETE
* TUJUAN: Menghapus file di server berdasarkan nama file.
* PARAMETER:
  - PARAMETER1: nama file
* RESULT:
- BERHASIL:
  - status: OK
  - data: pesan sukses
- GAGAL:
  - status: ERROR
  - data: pesan kesalahan
```

---
### Tambahkan fungsi upload dan delete di file_interface.py
- Edit file `file_interface.py` dan tambahkan dua method baru: `upload` dan `delete`.
- Tambahkan ini di dalam class `FileInterface`:
```
    def upload(self, params=[]):
        try:
            filename = params[0]
            filedata_base64 = params[1]
            filedata = base64.b64decode(filedata_base64)

            with open(filename, 'wb') as f:
                f.write(filedata)

            return dict(status='OK', data=f'File {filename} berhasil diupload')
        except Exception as e:
            return dict(status='ERROR', data=str(e))

    def delete(self, params=[]):
        try:
            filename = params[0]
            if not os.path.exists(filename):
                return dict(status='ERROR', data=f'File {filename} tidak ditemukan')

            os.remove(filename)
            return dict(status='OK', data=f'File {filename} berhasil dihapus')
        except Exception as e:
            return dict(status='ERROR', data=str(e))

```
Fungsi `upload(self, params=[])` digunakan untuk menangani proses unggah file dari client ke server. Fungsi ini menerima parameter berupa list params, di mana `params[0]` adalah nama file `(filename)` dan `params[1]` adalah data file yang telah di-encode dalam format Base64 `(filedata_base64)`. Data yang di-encode ini kemudian didekodekan kembali menjadi bentuk aslinya menggunakan base64.b64decode(), dan hasilnya disimpan ke dalam file baru dengan nama filename menggunakan mode tulis biner `('wb')`. Jika proses unggah berhasil, fungsi akan mengembalikan dictionary dengan status bernilai 'OK' dan pesan bahwa file berhasil diupload. Jika terjadi kesalahan selama proses, fungsi akan menangkap exception dan mengembalikan dictionary dengan status bernilai 'ERROR' beserta pesan kesalahannya.

Fungsi `delete(self, params=[])` berfungsi untuk menghapus file yang telah disimpan di server. Fungsi ini juga menerima parameter berupa list params, di mana params[0] berisi nama file yang ingin dihapus. Fungsi terlebih dahulu mengecek apakah file tersebut ada di sistem menggunakan `os.path.exists()`. Jika file tidak ditemukan, maka fungsi akan mengembalikan dictionary dengan status bernilai `ERROR` dan pesan bahwa file tidak ditemukan. Namun jika file ada, maka file akan dihapus menggunakan os.remove(). Setelah penghapusan berhasil, fungsi akan mengembalikan dictionary dengan status bernilai 'OK' dan pesan bahwa file berhasil dihapus. Jika terjadi kesalahan selama proses, fungsi akan menangkap exception dan mengembalikan dictionary dengan status bernilai `ERROR` beserta detail pesan kesalahannya.

---
### Update  `file_protocol.py` agar mengenali perintah `UPLOAD` dan `DELETE`
- Karena di file_protocol.py kita sudah menggunakan getattr(self.file, c_request) untuk memanggil fungsi berdasarkan nama perintah, maka kita tidak perlu mengubah banyak.
- Namun, kita perlu memastikan bahwa saat perintah UPLOAD diproses, parameter-nya benar—karena dia butuh dua parameter: nama_file dan file_base64.
```
class FileProtocol:
    def __init__(self):
        self.file = FileInterface()
    def proses_string(self,string_datamasuk=''):
        logging.warning(f"string diproses: {string_datamasuk}")
        c = shlex.split(string_datamasuk)
        try:
            c_request = c[0].strip().lower()
            logging.warning(f"memproses request: {c_request}")
            params = [x for x in c[1:]]
            cl = getattr(self.file, c_request)(params)
            return json.dumps(cl)
        except Exception as e:
            return json.dumps(dict(status='ERROR', data='request tidak dikenali'))
```
Kelas  `FileProtocol` digunakan sebagai protokol penghubung antara perintah dalam bentuk string dan pemanggilan fungsi yang sesuai dalam antarmuka file `(FileInterface)`. Pada saat inisialisasi `(__init__)`, kelas ini membuat objek self.file yang merupakan instance dari kelas `FileInterface`, tempat fungsi-fungsi seperti upload, delete, dan lain-lain berada.

Fungsi utama dalam kelas ini adalah `proses_string(self, string_datamasuk='')`, yang bertugas memproses input berupa string perintah. String tersebut terlebih dahulu dipisahkan menjadi token menggunakan `shlex.split()`, yang berguna agar string dengan spasi yang berada dalam tanda kutip tetap dianggap satu kesatuan. Token pertama (c[0]) dianggap sebagai nama metode (misalnya upload atau delete), lalu diubah menjadi huruf kecil dan dibersihkan dari spasi menggunakan `strip().lower()`. Token-token berikutnya (c[1:]) dijadikan sebagai parameter (params) yang akan diteruskan ke fungsi yang sesuai.

Fungsi kemudian menggunakan `getattr(self.file, c_request)` untuk memanggil metode yang sesuai dari objek FileInterface berdasarkan nama yang diberikan. Metode tersebut kemudian dijalankan dengan params sebagai argumen. Hasil eksekusi akan dikembalikan dalam format JSON menggunakan `json.dumps()`. Jika terjadi kesalahan — misalnya perintah tidak dikenali atau tidak sesuai dengan metode yang ada di FileInterface — maka fungsi akan mengembalikan respon JSON dengan status 'ERROR' dan pesan 'request tidak dikenali'.

---

### Tambahkan fungsi `remote_upload()` dan `remote_delete()` di `file_client_cli.py`
- Kita akan menambahkan dua fungsi baru di client:
`remote_upload(namafile)` dan `remote_delete(namafile)`

- Kedua fungsi ini akan mengirim perintah ke server sesuai format protokol:
`UPLOAD <namafile> <base64_data>` dan `DELETE <namafile>`

```
def remote_upload(namafile=""):
    try:
        # baca file dan encode base64
        with open(namafile, "rb") as f:
            file_data = base64.b64encode(f.read()).decode()

        # kirim perintah ke server
        command_str = f"UPLOAD {namafile} {file_data}"
        hasil = send_command(command_str)

        if hasil['status'] == 'OK':
            print(f"File {namafile} berhasil diupload.")
        else:
            print("Gagal upload:", hasil['data'])
    except Exception as e:
        print("Terjadi kesalahan:", str(e))


def remote_delete(namafile=""):
    command_str = f"DELETE {namafile}"
    hasil = send_command(command_str)

    if hasil['status'] == 'OK':
        print(f"File {namafile} berhasil dihapus.")
    else:
        print("Gagal menghapus:", hasil['data'])
```
Fungsi `remote_upload(namafile="")` digunakan oleh client untuk mengunggah file ke server. Fungsi ini pertama-tama membuka file lokal berdasarkan nama yang diberikan melalui parameter namafile, membaca seluruh isinya dalam mode `biner ('rb')`, kemudian mengenkripsi isi file tersebut ke dalam format `Base64` menggunakan `base64.b64encode()`. Hasil encoding tersebut diubah menjadi string UTF-8 agar dapat dikirim melalui jaringan. Setelah itu, fungsi menyusun perintah dalam bentuk string, dimulai dengan kata kunci UPLOAD, diikuti dengan nama file dan isi file yang telah di-encode, lalu dikirim ke server melalui fungsi `send_command()`. Jika server membalas dengan status 'OK', maka akan ditampilkan pesan bahwa file berhasil diunggah. Jika tidak, pesan kesalahan akan ditampilkan. Fungsi juga dilengkapi blok try-except untuk menangkap dan menampilkan pesan jika terjadi kesalahan seperti file tidak ditemukan atau gagal dibaca.

Fungsi `remote_delete(namafile="")` digunakan untuk menghapus file yang sebelumnya telah diunggah ke server. Fungsi ini menyusun perintah dalam bentuk string dengan format `DELETE <namafile>` lalu mengirimkannya ke server melalui fungsi `send_command()`. Hasil respon dari server kemudian diperiksa. Jika status yang dikembalikan adalah 'OK', maka akan ditampilkan pesan bahwa file berhasil dihapus. Namun jika server mengembalikan status error, maka pesan kegagalan akan ditampilkan. Fungsi ini lebih sederhana karena tidak memproses isi file, melainkan hanya mengirimkan perintah berdasarkan nama file yang ingin dihapus.

---

### Update bagian if __name__ == '__main__':
```
if __name__=='__main__':
    server_address=('172.16.16.101',6969)

    remote_list()
    remote_get('donalbebek.jpg') // contoh file yang akan di minta ke server
    remote_upload('test.txt') //untuk pengetesan upload file ke server
    remote_delete('donalbebek.jpg')
    remote_list()

```
