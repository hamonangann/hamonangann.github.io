---
layout: post
title:  "Mengatur Testing dengan Test Double"
categories: tips
---

Pernahkah Anda membuat aplikasi yang memiliki beberapa layer atau bergantung pada third party library/software? Kita
bisa gunakan seluruh aplikasi kita untuk diuji dalam unit test, misalnya menggunakan jaringan internet sungguhan atau
memanggil third party library sungguhan. Kemudian, kita menjalankan tes untuk happy path, dan semuanya bekerja dengan
semestinya.

Lantas kita mencoba untuk membuat negative path. Kita tahu persis apa error yang ingin kita simulasikan, tapi error yang
ingin kita uji merupakan unexpected error atau error yang tidak bisa kita reproduksi tanpa menulis ulang library
tersebut.

Best practice untuk mengatasi permasalahan di atas adalah dengan mengecilkan scope:

- Pastikan kita hanya menguji satu logic atau behavior aplikasi yang kita buat
- Dependensi dapat diwakili oleh test double

Ada apa saja jenis test double dan perbedaannya? Mari kita kupas satu-satu:

## Dummy

Dummy adalah test double paling sederhana. Ia tidak punya fungsionalitas apa-apa dan biasanya digunakan hanya untuk
menempati parameter sebuah fungsi.

Sebagai contoh dalam Python kita memiliki program untuk memilih nama dari daftar secara acak sebagai berikut

```python
def 
```

## Stub

Stub adalah dummy yang naik level. Stub punya fungsionalitas, tapi dirancang untuk memiliki kepastian sesuai kebutuhan
unit test. Misalnya pada fungsi randomize user, stub bisa digunakan untuk selalu mengembalikan user bernama "Budi".
Sehingga pada assertion, kita tinggal pastikan apakah user yang dikembalikan bernama Budi atau tidak.

## Spy

Spy adalah level berikutnya dari stub. Stub punya fungsionalitas tambahkan yang bahkan tidak dimiliki oleh fungsi atau
class yang asli (yang digunakan di production), yaitu kemampuan untuk mendengar. Misalnya, spy dapat mencatat berapa 
kali dirinya atau salah satu fungsinya dipanggil.

## Mocks

Mocks adalah spy dalam versi yang lebih canggih. Jika spy punya kemampuan untuk mendengar, mock punya kemampuan untuk
merangkum apa yang ia dengar. Misalnya dalam suatu test, fungsi yang ditest harus memanggil fungsi lain A() 1 kali,
B() 1 kali, dan C() 2 kali. Jika menggunakan spy kita bisa membutuhkan 3-4 kali assertion untuk memastikan setiap fungsi 
dipanggil dengan jumlah yang tepat. Akan tetapi, mocks dilengkapi dengan fungsionalitas verify sehingga kita cukup 
melakukan assertion sebanyak 1 kali.

Kebanyakan test framework membuat objek/fungsi dalam bentuk mocks ini.

## Fakes

Fakes punya fungsionalitas yang serupa dengan fungsi asli, tapi dalam behavior yang palsu. Misalnya pada randomizer user 
di atas, fakes dapat dibuat untuk selalu mengembalikan user pertama dalam list, atau null jika listnya kosong. Tentu
saja kita tidak mau production app kita selalu mengembalikan user pertama, tapi untuk kebutuhan testing dan development
ini akan sangat berguna untuk menguji suatu behavior.

## Referensi

Eftimov, I. (2019). Testing in Go: Test Doubles by Example. https://ieftimov.com/posts/testing-in-go-test-doubles-by-example/ 

Test Double at XUnitPatterns.com. (n.d.). http://xunitpatterns.com/Test%20Double.html