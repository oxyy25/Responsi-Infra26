# Analisis Perbaikan

## Permasalahan 1
 
### Gejala
muncul pesan : yaml: line 5, column 8: mapping values are not allowed in this context yang menandakan adanya salah syntax pada file konfigurasi yaml.
 
### Penyebab
pada bagian services, tidak ada tanda : yang membuat indentasi dari file konfigurasi bermasalah
 
### Solusi
menambahkan : setelah services agar indentasi tidak lagi bermasalah.
```
services:
  nginx:
    build:
      context: ./nginx
    container_name: nginx-lb
```

---

## Permasalahan 2
 
### Gejala
Web1 gagal terhubung ke database, container web1 terus restart atau error saat ingin melakukan koneksi ke database.
 
### Penyebab
Variabel `DB_HOST` pada service web1 di file  `docker-compose.yml` diisi dengan nilai `mysql`, padahal service database didefinisikan dengan `db`. Docker Compose menggunakan nama service sebagai hostname antar container, sehingga karena nama service yang salah ini menyebabkan koneksi gagal.
 
### Solusi
Mengubah nilai `DB_HOST: mysql` menjadi `DB_HOST: db` pada konfigurasi environment service web1 di file `docker-compose.yml`.
 
---
 
## Permasalahan 3
 
### Gejala
Web2 tidak bisa melakukan koneksi ke database, muncul error autentikasi.
 
### Penyebab
Variabel `DB_PASS` pada service web2 diisi dengan nilai `wrongpassword`, yang merupakan password yang salah dimana password yang didefinisikan pada service `db` yaitu `student123`. Akibatnya autentikasi ke MySQL selalu gagal.
 
### Solusi
Mengubah `DB_PASS: wrongpassword` menjadi `DB_PASS: student123` pada konfigurasi environment service web2 di file `docker-compose.yml`.
 
---
 
## Permasalahan 4
 
### Gejala
Docker Compose gagal melakukan build, muncul error `build path ./web33 not found`.
 
### Penyebab
Build untuk service web3 pada file `docker-compose.yml` mengarah ke direktori `./web33` yang tidak ada. Folder yang benar adalah `./web3`.
 
### Solusi
Mengubah `context: ./web33` menjadi `context: ./web3` pada bagian build service web3 di file `docker-compose.yml`.
 
---
 
## Permasalahan 5
 
### Gejala
Web3 tidak muncul dalam rotasi load balancer Nginx, hanya web1 dan web2 yang melayani request.
 
### Penyebab
Service web3 hanya terhubung ke network `backend`, tidak ke network `frontend`. Nginx hanya bisa berkomunikasi dengan service yang berada di network `frontend`, sehingga web3 tidak terjangkau oleh Nginx.
 
### Solusi
Menambahkan `- frontend` pada bagian `networks` service web3 di file `docker-compose.yml` agar web3 terhubung ke kedua network.
 
---
 
## Permasalahan 6
 
### Gejala
Docker Compose error saat startup karena volume tidak ditemukan.
 
### Penyebab
Nama volume yang didefinisikan di bagian `volumes:` adalah `database-data`, namun yang direferensikan di service `db` adalah `db-data`. Ketidakcocokan nama ini menyebabkan Docker tidak bisa menghubungkan volume dengan benar.
 
### Solusi
Menyamakan nama volume menjadi `db-data` di bagian `volumes:` pada file `docker-compose.yml` agar konsisten dengan referensi di service `db`.
 
---
 
## Permasalahan 7
 
### Gejala
Container web1 gagal build dengan error `php:8.2-apach: not found`.
 
### Penyebab
Terdapat typo pada FROM di `Dockerfile` web1, yaitu `php:8.2-apach` (kurang huruf e). Image dengan nama tersebut tidak ada di Docker Hub.
 
### Solusi
Mengubah `FROM php:8.2-apach` menjadi `FROM php:8.2-apache` pada `web1/Dockerfile`.
 
---
 
## Permasalahan 8
 
### Gejala
Container web3 gagal build dengan error `php:8.2-apche: not found`.
 
### Penyebab
Sama seperti web1,terdapat typo pada FROM di `Dockerfile` web3, yaitu `php:8.2-apche` (huruf tertukar). Image dengan nama tersebut tidak ada di Docker Hub.
 
### Solusi
Mengubah `FROM php:8.2-apche` menjadi `FROM php:8.2-apache` pada `web3/Dockerfile`.
 
---
 
## Permasalahan 9
 
### Gejala
Container nginx crash saat startup dengan error `unknown directive "```nginx" in /etc/nginx/nginx.conf:2`.
 
### Penyebab
File `nginx/nginx.conf` mengandung markdown (` ``` `) di bagian awal dan di bagian akhir.Hal ini menyebabkan Nginx tidak bisa membaca konfigurasi.
 
### Solusi
Menghapus baris ` ``` ` di awal dan di akhir file `nginx/nginx.conf` sehingga hanya menyisakan konfigurasi Nginx yang valid.
 
---
 ## Permasalahan 10
 
### Gejala
Nginx mengembalikan error 502 Bad Gateway saat diakses. Beberapa web server web1 tidak dapat dijangkau oleh Nginx.
 
### Penyebab
web1 ditulis sebagai web11 padahal nama container yang benar adalah web1. Docker tidak dapat menemukan hostname web11 dalam jaringan internal sehingga koneksi ke web1 selalu gagal.
### Solusi
Mengubah nama container dari web11 ke web1
 
---
## Permasalahan 11
 
### Gejala
Nginx berjalan namun selalu mengembalikan error `502 Bad Gateway` saat diakses melalui port 8080.
 
### Penyebab
Konfigurasi upstream di `nginx/nginx.conf` menggunakan port `8080` untuk web server web3(`server web3:8080`). Port 8080 adalah port host yang di-expose keluar, bukan port internal container. Web server PHP-Apache di dalam container berjalan pada port `80`.
 
### Solusi
Mengubah konfigurasi upstream di `nginx/nginx.conf` dari port `8080` menjadi port `80`:
```
upstream backend {
    server web1:80;
    server web2:80;
    server web3:80;
}
```
 
---
 
## Permasalahan 12
 
### Gejala
Container mysql-db crash saat dijalankan dengan error `ERROR 1064 (42000)` pada saat menjalankan `init.sql`.
 
### Penyebab
File `db/init.sql` mengandung  markdown (` ``` `) di baris pertama dan diakhir file seperti pada file `nginx/nginx.conf`.Hal ini yang menyebabkan error sintaks pada SQL.
 
### Solusi
Menghapus baris ` ``` ` di awal dan ` ``` ` di akhir file `db/init.sql` dan sisakan perintah SQL yang valid saja.
 