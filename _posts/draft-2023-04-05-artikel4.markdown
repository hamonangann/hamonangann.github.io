---
layout: post
title:  "Cara Mudah Menjamin Kualitas Kode dengan Gitlab CI"
categories: tips
---

Kualitas perangkat lunak perlu dijamin baik, baik pada tahap pengembangan maupun maintenance. Akan tetapi, menjalankan
berbagai prosedur penjaminan bisa memakan waktu yang cukup panjang jika tidak diautomasi. Pada artikel kali ini, kita
akan menjalankan beberapa prosedur penjaminan perangkat lunak (Sofware Quality Assurance) dengan menggunakan Gitlab.

## Berbagai jenis penjaminan kualitas kode

Ada beberapa jenis prosedur penjaminan kualitas kode, biasanya berkutat pada kemudahan kode untuk di-maintain dan faktor
keamanan perangkat lunak. Pada praktik ini kita akan mencoba 3 buah prosedur:

1. **Code Quality**: pemeriksaan kualitas kode. Prosedur ini akan menganalisis kompleksitas kode, memastikan kode yang
ditambahkan readable dan mudah di-maintain. Salah satu tool yang terkenal untuk ini adalah
[Code Climate](https://docs.codeclimate.com/docs/list-of-engines)

2. **Secret Detection**: penting untuk kode tidak mengekspos hal-hal yang seharusnya tidak terbaca langsung oleh 
developer atau pengunjung source code, misalnya alamat dan password database. Caranya adalah dengan menyimpan nilai
rahasia pada OS environment variable (env) suatu server. Prosedur secret detection membantu mendeteksi adanya kode
yang mengekspos nilai atau alamat yang semestinya rahasia.

3. **Static Application Security Testing (SAST)**: seiring perkembangan teknologi terdapat berbagai kerentanan
(vulnerability) yang bisa menjadi celah keamanan perangkat lunak yang kita kembangkan. Prosedur SAST bekerja sesuai
dengan bahasa pemrograman yang digunakan, serupa dengan Code Quality. SAST mendeteksi apakah kode yang kita buat
memiliki kerentanan, dan melaporkannya apabila ada

## Gitlab untuk Continuous Integration

Gitlab adalah salah satu tool yang digunakan untuk hosting sumber kode yang dikelola dengan version control system Git
di Internet. Gitlab dapat diakses melalui web browser dan memiliki fitur yang terus bertambah. Pada artikel sebelumnya
kita telah membahas Gitlab sebagai [Project Tracking tool](https://hamonangann.github.io/tips/2023/03/03/artikel1.html).

Salah satu fitur yang dapat digunakan pada Gitlab adalah Continuous Integration (CI). Pada CI, terdapat satu komputer
(server) yang dikhususkan untuk menyatukan kode dari berbagai developer. Komputer ini tidak akan menjalankan kode secara
permanen, tetapi melaksanakan sekian prosedur yang menjamin kode baru dapat digabungkan tanpa merusak kualitas kode
yang sudah ada. Ini mencakup tahap build (membuat executable) dan test.

Pada Gitlab, kita dapat membuat konfigurasi CI dengan mudah melalui fitur 
[Auto Devops](https://docs.gitlab.com/ee/topics/autodevops/) atau menulis file konfigurasi `.gitlab-ci.yml` pada root
project kita. Di sini kita akan memakai cara kedua yang lebih fleksibel. Gitlab menjalankan CI dengan sistem pipeline
yang memungkinkan beberapa proses berjalan bersamaan.

## Mengonfigurasi Gitlab Ci untuk Menjamin Kualitas Kode

Pertama-tama kita buka project kita pada editor teks dan tuliskan konfigurasi berikut

```yaml
// .gitlab-ci.yml
---
include:
  - template: Jobs/Code-Quality.gitlab-ci.yml
  - template: Jobs/SAST.gitlab-ci.yml
  - template: Jobs/Secret-Detection.gitlab-ci.yml

stages:
  - test
```

Simpan pada root folder, lalu lakukan `git add .gitlab-ci.yml`, `git commit` dan `git push` ke Gitlab.

Sebagai catatan, kita perlu membuat stages test karena ketiga template di atas dapat dijalankan pada stage test. Apabila
sudah memiliki script, kita tinggal perlu menambahkan include tanpa mengubah kode yang sudah ada sebelumnya. Selain
itu, masih ada lagi jenis-jenis prosedur penjaminan perangkat lunak seperti Dynamic Application Security Testing (DAST),
Dependency Scanning, dan sebagainya.

## Referensi

Code Quality. GitLab. (n.d.). https://docs.gitlab.com/ee/ci/testing/code_quality.html 

Fowler, M. (n.d.). Continuous Integration. martinfowler.com. https://martinfowler.com/articles/continuousIntegration.html 

Secret Detection. GitLab. (n.d.). https://docs.gitlab.com/ee/user/application_security/secret_detection/ 

Static Application Security Testing (SAST). GitLab. (n.d.). https://docs.gitlab.com/ee/user/application_security/sast/