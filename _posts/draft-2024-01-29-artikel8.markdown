---
layout: post
title:  "Cara Deploy Kode yang Aman di GitHub"
categories: tips
---

Pada proyek-proyek pribadi atau berlevel 'iseng' yang diunggah ke GitHub, yang sering terjadi adalah proyek dibuat sebagai publik, tapi ada variabel yang harusnya tidak ketahuan siapapun. Misalnya ketika kita deploy kode sumber layanan web di GitHub, username dan password database disimpan dalam file bernama `.env`. Sayangnya value dari password tersebut juga ikut terunggah, sehingga database kita terekspos!

Ada beberapa workaround untuk ini, mari kita bahas salah satunya!

## 1. Menyimpan Environment Secara Lokal dengan Gitignore

Salah satu fitur git yang membuat satu file di-ignore (sehingga hanya tersimpan pada file system, tapi tidak masuk ke dalam git repository) adalah gitignore. Mari kita coba

Misalnya kita punya file `.env` seperti berikut.
```shell
username=budi123
password=pyth0nsudars0n0
```

```Perhatikan bahwa di sebelah kiri dan kanan tanda sama dengan tidak boleh ada spasi```

Mari kita simpan .env tersebut dalam suatu folder, lalu buat git repository untuk folder tersebut, dengan melakukan langkah-langkah di bawah ini

```shell
mkdir hands-on-env
cd hands-on-env
git init
git branch -M main
```

Kini kita berada pada branch main pada Git. Lalu pada folder hands-on-env buatlah file berjudul `.env` dan copy-paste isi dari file env di atas.

Setelah kita melakukan save, jika di titik ini kita melakukan git add dan commit, file env ini akan terekspos. Untuk mencegah hal ini, mari kita gunakan gitignore. Mari buat file berjudul `.gitignore` dan isikan seperti ini

```shell
.env
```

Save file ini dan kini kita sudah meng-exclude `.env`. Karena kita akan berbagi kode sumber di GitHub, kita perlu memberitahu bahwa env ini harus ada, TANPA mengekspos valuenya. Developer bisa cukup peka terhadap hal ini jika ada file berjudul `.env.example` dengan isi sebagai berikut

```shell
username=
password=
```

Mari buatkan `.env.example` ini di dalam folder yang sama dengan di mana kita menaruh `.env`

```Nama file example ini sebenarnya boleh apa saja selama ia tidak di-ignore. Hanya saja, .env.example lazim dipakai sehingga orang lain bisa mengetahuinya secara langsung. Perhatikan bahwa kita hanya mengekspos nama variabel tanpa memberitahu value-nya apa. Ini sudah dianggap aman```

Setelah kita menyimpan tiga file `.env`, `.env.example`, dan `.gitignore`, kita bisa unggah ke repositori git. Jalankan perintah berikut

```shell
git add -A
git commit -m "initial commit"
```

Kini baik `.env.example` maupun `.gitignore` sudah masuk repositori Git (lokal) kita. Akan tetapi, file `.env` tidak termasuk.

## 2. Mengunggah environment ke GitHub

Untuk mengunggah ke GitHub, kita perlu membuat remote repository baru. Buat akun GitHub atau login ke akun yang dipunya, lalu ke https://github.com/new untuk buat repository baru.

Ikuti panduan di bawah

![Setup Github](/img/artikel8-setup-github.png)

```Silakan buat repositori tanpa template, berikan nama 'hands-on-env' untuk repository dan setel sebagai Public. Tidak perlu mengubah konfigurasi lainnya```

Kita akan diarahkan ke laman GitHub repository kita. Jalankan command yang berada pada '...or push an existing repository from the command line'

```shell
git remote add origin https://github.com/<nama-user>/hands-on-env.git
git branch -M main
git push -u origin main
```

Silakan refresh laman GitHub kita. Perhatikan bahwa kita hanya mengunggah 2 file, yaitu `.env.example` dan `.gitignore`. Jadi amanlah proyek kita!

### 3. Membuat laman web sederhana

### 4. Deploy otomatis secara aman dengan Github Action CI/CD Pipeline

## Referensi

WIP