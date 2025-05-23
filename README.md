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
- upload: menerima nama file dan konten base64, lalu disimpan sebagai file baru.
- delete: mengecek apakah file ada, lalu menghapusnya.

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
