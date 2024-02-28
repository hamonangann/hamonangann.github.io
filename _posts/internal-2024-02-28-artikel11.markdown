---
layout: post
title: "Panduan Deployment ke Server Fasilkom UI"
categories: tips
feature_image: "/img/artikel10-featured.jpeg"

summary: "Panduan instalasi aplikasi, static files, reverse proxy, dan database pada server on-premise dalam jaringan Fasilkom UI."
url: "N/A"
category: "DevOps"
status: "Published"
author: "Bornyto Hamonangan <bornyto.hamonangan@ui.ac.id>"
version: 1.4
license: "Creative Commons Attribution-ShareAlike 4.0 International ([CC-BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/))."
revision_history: "N/A"
---

# Pengantar

Beberapa aplikasi dapat dideploy dengan pembagian akses untuk meningkatkan keamanan. Sebagai contoh, perhatikan arsitektur aplikasi yang dideploy di jaringan Fasilkom UI berikut ini:

![Arsitektur On Premise Fasilkom](/img/artikel11-arsitektur.png)

Sebagai pengembang, Anda akan mendapatkan akses sebagai berikut:

- _Virtual Machine Instance Name_
- _Virtual Machine IP_
- _Virtual Machine Username and Password (as sudoers)_
- _Domain name_

Anda atau siapapun di internet dapat mengakses aplikasi Anda sebagai pengguna dengan memasukkan _domain name_ pada browser. _Request_ akan diteruskan oleh _reverse proxy_ **milik Fasilkom** (_proxy pass_) menuju IP privat _virtual machine (VM)_ Anda. Server akan mengembalikan _response_ ke user lewat _reverse proxy_ Fasilkom. Sebagai catatan, _domain name_ didaftarkan oleh IT Fasilkom UI dan setiap request ke _domain name Anda_ akan diarahkan secara otomatis ke VM Anda.

Akan tetapi, sebagai pengembang atau pengelola, Anda tidak dapat mengakses VM dari luar jaringan Fasilkom. Dengan demikian, Anda tidak bisa SSH atau SCP secara langsung ke server Anda. Contoh: `ssh ppl.cs.ui.ac.id` tidak bisa dilakukan. Anda perlu SSH dengan _jumphost_ Kawung ke IP privat VM Anda. Caranya, Anda masuk ke Kawung dengan SSH key pribadi: `ssh -i <your_ssh_key_file> -p 12122 <your_UI_SSO_username>@kawung.cs.ui.ac.id`. Setelah itu SSH ke dalam VM dengan private key: `ssh admin@<your_vm_private_ip>`

Setidaknya ada 4 hal yang perlu dikonfigurasi pada _deployment_ Anda:

- [Aplikasi](#aplikasi)
- [_Static Files (html, css, img, pdf)_](#static-files)
- [_Reverse Proxy (NGINX)_](#reverse-proxy)
- [_Database_](#database)


# Aplikasi

Karena Anda perlu mengakses SSH atau SCP dengan kredensial pribadi, penggunaan SSH atau SCP untuk _deployment_, baik secara manual maupun lewat CI/CD Pipeline akan menjadi cukup rumit. Sebagai solusi, Anda dapat **membungkus _build artifact_ aplikasi sebagai Docker _image_**, lalu _publish_ ke sebuah Docker _registry_. Nantinya, _image_ tersebut dapat diunduh (_pull_) di VM Anda.

<!-- TODO buat panduan GitLab Registry-->

Untuk mengunduh Docker _image_ setelah di-_publish_ ke _registry_, SSH ke VM. Sebelum berinteraksi dengan _container_, lakukan instalasi Docker pada VM terlebih dahulu. Untuk itu, Anda dapat merujuk [Panduan Instalasi Docker di Debian](https://docs.docker.com/engine/install/debian/).

### _Action items_:

- _Setup_ repositori _Docker registry_
- Ggunakan _docker-in-docker_ (dind) pada CI/CD pipeline untuk _publish image_ otomatis
- Instalasi Docker di VM. Jalankan `docker version` untuk memastikan
- Unduh _image_ dan jalankan _container_ di VM

Sebagai alternatif, Anda dapat menggunakan beberapa _container_ sekaligus. Anda dapat mengeksplorasi docker-compose, Docker Swarm, atau Kubernetes untuk mempermudah instalasi lebih dari satu _container_.


# Static Files

Yang termasuk dalam _static files_ dalam sebuah aplikasi web adalah **file-file seperti gambar, audio, video, dokumen PDF, HTML, atau CSS**. Terdapat dua jenis _static files_, yaitu yang dihasilkan oleh proses _build_ aplikasi, maupun yang di-generate pada saat aplikasi dijalankan atau diunggah oleh end-user. Anda perlu memperlakukan aplikasi yang Anda deploy sebagai _stateless_. Nantinya _maintainer_ mungkin perlu menambahkan, menghentikan, atau bahkan menghapus _container_ aplikasi, tapi _file-file_ tidak boleh hilang.

Selain itu, Anda perlu mempertimbangkan bahwa di lingkungan _production_, barangkali aplikasi Anda tidak melakukan _serving static file_. Sebagai contoh, Gunicorn pada Python yang digunakan untuk menjalankan _application_ server tidak bisa menyediakan _static file_. Hal ini dapat diatasi dengan penggunaan _reverse proxy_ yang akan dibahas pada bagian selanjutnya. Akan tetapi, langkah-langkah berikut ini tetap perlu Anda lakukan.

Untuk melakukan hal ini, pertama-tama Anda perlu memastikan aplikasi Anda telah memiliki URL untuk mengunduh _static file_ serta lokasi untuk mengunduh dan mengunggah _static files_. Sebagai contoh, pengaturan lokasi _static files_ pada Django terletak pada variabel `STATIC_URL` dan `STATIC_ROOT` pada `settings.py`. Yang perlu Anda lakukan adalah **membuat suatu _persistent volume_ untuk menyimpan file** pada host (VM). Dengan demikian, baik _container_ aplikasi maupun _host_ dapat mengakses _static file_ Anda. Caranya, Anda dapat melakukan ini dengan fitur Docker _volume_. Sebagai contoh, untuk membuat Docker _volume_ bernama `static_volume`, Anda dapat menjalankan `docker volume create static_volume` 

Sebagai contoh, berikut cara membuat dan menjalankan container baru yang dipasangkan dengan volume.

```shell
docker create -p 8000:8000 --name app <url_image> --volume static_volume:<lokasi_static_file_di_container>
docker start app
```

Pada server Fasilkom berbasis Linux, lokasi _default_ untuk Docker volume adalah pada `/var/lib/docker/volumes/`.

### _Action items_:

- _Setup_ URL dan lokasi _static file_ pada aplikasi
- Buat sebuah _persistent volume_ di luar _container_ dan pasangkan dengan _container_


# Reverse Proxy

Secara default, server di VM Anda menggunakan **NGINX sebagai _reverse proxy_**. Anda dapat memastikannya dengan `nginx -V`. Ini bisa Anda manfaatkan untuk dua hal

- Melakukan _proxy pass_ ke host dan port aplikasi
- Secara langsung _serving static files_

Anda dapat **mengonfigurasi NGINX _reverse proxy_** dengan mengubah file `/etc/nginx/sites-available/default` pada VM. Berikut contoh konfigurasi NGINX sederhana untuk _proxy pass_ ke server dengan _domain name_ `ppl.cs.ui.ac.id` pada port `8000` dan _serving static files_ dengan URL `/static/`.

```shell
server {
	listen 80;
	listen [::]:80;

	server_name ppl.cs.ui.ac.id;

	location / {
		proxy_pass http://127.0.0.1:8000;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto https;
        proxy_redirect off;
	}

    location /static/ {
        alias /var/lib/docker/volumes/static_volume/_data;
    }
}
```

Setelah mengubah file konfigurasi, _restart_ NGINX dengan `sudo systemctl restart nginx`. Anda dapat mencoba mengakses situs Anda dan juga URL _endpoint_ yang Anda tetapkan untuk _serving static file_.

Sebagai catatan, konfigurasi server di atas digunakan untuk setiap _request_ ke port 80 (HTTP) dengan menggunakan _domain name_ `ppl.cs.ui.ac.id`. Jika end-user mengakses `ppl.cs.ui.ac.id`, NGINX akan mengarahkan request ke port 8000 (_proxy pass_) dan menambahkan beberapa header. Akan tetapi jika end-user mengakses `ppl.cs.ui.ac.id/static/namafile.jpg`, maka NGINX akan mencari dan menampilkan `namafile.jpg` jika ada dalam direktori `/var/lib/docker/volumes/static_volume/_data`. Jika end-user mengakses `ppl.cs.ui.ac.id/static/admin/namafile.jpg`, maka NGINX akan mencari dan menampilkan `namafile.jpg` jika ada dalam direktori `/var/lib/docker/volumes/static_volume/_data/admin`. 

Anda dapat membaca dokumentasi terkait NGINX berikut

- [NGINX sebagai _Web Server_](https://docs.nginx.com/nginx/admin-guide/web-server/web-server/)
- [NGINX untuk _Static Files_](https://docs.nginx.com/nginx/admin-guide/web-server/serving-static-content/)

### _Action items_:

- Konfigurasi NGINX di `sites_available` pada VM.
- Pastikan web dapat diakses dan NGINX dapat menyediakan _static file_. Misalnya, tampilan web Anda seharusnya sesuai CSS rule yang Anda tetapkan di static file `style.css`.

# Database

Sama seperti _static files_, pada lingkungan _production_, _database_, _cache_, _message queue_ dan sebagainya **seharusnya diletakkan di luar aplikasi** atau _container_ aplikasi Anda dan tidak diekspos portnya pada NGINX proxy VM Anda. Ini penting demi keamanan data Anda. Anda dapat menginstalasi _database_ secara langsung di dalam VM dengan port standar atau membungkusnya sebagai _container_. Anda dapat merujuk pada tutorial berikut sesuai kebutuhan aplikasi Anda:

- [Instalasi PostgreSQL](https://www.postgresql.org/download/linux/debian/)
- [Instalasi MySQL](https://dev.mysql.com/doc/mysql-apt-repo-quick-guide/en/#apt-repo-fresh-install)
- [Instalasi Redis](https://redis.io/docs/install/install-redis/install-redis-on-linux/)

Setelah melakukan instalasi, jangan lupa untuk meng-_update_ kredensial _database_ (host, port, username, password) Anda pada aplikasi. Anda dapat membuat suatu `.env` file pada VM Anda (di luar aplikasi dengan akses root), lalu membuat _container_ dengan perintah seperti contoh berikut. 

```shell
chmod 440 .env
docker create -p 8000:8000 --env-file .env --name app <url_image> --volume static_volume:<lokasi_static_file_di_container>
```

Sebagai tips, Anda dapat menggunakan `172.17.0.1` sebagai variabel DB Host, jika DB berjalan pada VM sedangkan aplikasi berjalan pada _container_. Alamat `172.17.0.1` adalah alamat untuk VM host pada setiap _containers._

### _Action items_:

- Instalasi _database_ untuk menyimpan data di luar aplikasi.
- Konfigurasi kredensial database secara aman untuk aplikasi Anda.