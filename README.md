# Tugas-3 Pemrograman Jaringan (C) - 5025221138

### Farrel Akmalazmi Nugraha
### 5025221138

---

### Soal 1 
-  Upload File
#### File yang diunggah harus dikodekan terlebih dahulu ke dalam format Base64 sebelum dikirim.
-  Hapus File
#### Implementasikan fitur untuk menghapus file yang sudah ada di server.

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
