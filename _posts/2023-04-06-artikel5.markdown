---
layout: post
title:  "Prinsip SOLID pada API"
categories: tips
---

Kita akan mencoba menerapkan prinsip SOLID pada sebuah API yang dibuat dengan Django.

SOLID adalah gabungan dari lima design principles yang relevan dengan pengembangan program berorientasi objek. SOLID
itu sendiri dicetuskan oleh Robert C. Martin (Uncle Bob). Istilah SOLID dipopulerkan oleh Michael Feathers.

Ada lima prinsip dalam SOLID:

1. Single-responsibility principle 
2. Open–closed principle 
3. Liskov substitution principle 
4. Interface segregation principle 
5. Dependency inversion principle

Kali ini kita akan mengembangkan API sederhana yang menerapkan prinsip SOLID. API ini punya satu endpoint dan
mengembalikan nama orang secara random.

## Setup Proyek

Pastikan sudah menginstal Python 3. Panduan instalasinya terdapat pada [situs resmi Python](https://www.python.org/downloads/)
Tutorial ini menggunakan versi 3.11.2. Pada instalasi, jangan lupa tambahkan python ke PATH.

Buat proyek baru. Eksekusi perintah berikut pada CMD/terminal:
```shell
# Buat folder
mkdir praktik-solid-api-django
cd praktik-solid-api-django

# Buat virtual environment untuk instalasi Django
python3 -m venv env
source env/bin/activate  # Pada Windows gunakan perintah `env\Scripts\activate`

# Install Django dan Django REST framework
pip install django
pip install djangorestframework

# Buat satu project dan satu app
django-admin startproject solidapi .  # Jangan lupa tambahkan . di akhir
cd solidapi
django-admin startapp generator
cd ..
```

Selanjutnya eksekusi perintah berikut untuk coba menjalankan server

```shell
python3 manage.py migrate
python3 manage.py runserver
```

Setup proyek selesai.

## Menerapkan Single Responsibility Principle (SRP)

Django sendiri sudah membuat ini menjadi mudah dengan membuat folder `generator` berisi file-file yang mengatur
satu kepentingan saja. Ini sejalan dengan SRP yang menginginkan kelas atau fungsi hanya bertanggu jawab terhadap
satu hal saja.

- `admin.py`: mengatur Django Admin
- `apps.py`: mengatur config
- `models.py`: mengatur model (apa saja atribut yang diperlukan sebuah objek, dalam hal ini nama orang)
- `tests.py`: unit test
- `views.py`: mengatur controller (request dan response untuk user)

Kita tinggal melanjutkan ini dengan konsisten.

Mari kita isi beberapa file yang dibutuhkan.

```shell

```

Kita dapat membuat pseudocode untuk API sebagai berikut

```javascript
function main() {
    numberOfCats = number of "*.png" on "./assets"
    when have request with URL == "/cat" {
        catChoice = string(random between(1, numberOfCats))
        catImage = get file(formattedString("./assets/cat%s", catChoice))
        response = render(catImage)
        return response
    }
    listen to port(":9001")
}
```

Secara singkat, API berjalan di port 9001. Gambar-gambar kucing disimpan pada folder `assets` dengan nama `cat1.png`,
`cat2.png`, dan seterusnya.

Program di atas melakukan berbagai hal: menerima API request, melakukan filter (URL harus "/cat"), generate bilangan
acak, dan mengambil gambar dari folder. Jika ingin menerapkan SOLID kita harus memisahkan program sehingga setiap
fungsi atau prosedur hanya fokus menjalankan satu hal. Ini bisa kita capai dengan membuat beberapa fungsi:

```javascript
function main() {
    handler()
    listen to port(":9001")
}
```

```javascript
function handler() {
    when have request with URL == "/cat" {
        catImage = service()
        response = render(catImage)
        return response
    }
}
```

```javascript
function service() {
    numberOfCats = number of "*.png" on "./assets"
    catChoice = string(random between(1, numberOfCats))
    catImage = repository(catChoice)
}
```

```javascript
function repository(catChoice) {
    catImage = get file(formattedString("./assets/cat%s", catChoice))
    return catImage
}
```

Nah! Kita telah menerapkan salah satu prinsip SOLID yaitu Single Responsibility principle, di mana setiap kelas atau
fungsi hanya melakukan satu hal.

## Bagaimana dengan prinsip lainnya?

API kita populer dan kita ingin menambahkan fitur untuk mengambil gambar anjing. Di masa depan bisa saja kita akan
perlu fitur untuk mengirim gambar panda, kelinci dan sebagainya.

Berdasarkan prinsip Open-Closed, sebisa mungkin kelas tidak diubah melainkan ditambahkan (kecuali untuk bug fix).
Kita bisa melakukan refactoring untuk memudahkan hal ini ke depannya.

Pertama-tama, akan lebih rapi jika setiap layer diletakkan pada satu folder, jadi kita bisa membuat folder
`handler`, `service`, dan `repository`, sedangkan `main` tetap pada root project folder.

Kemudian refactor Repository menjadi berikut:

```javascript
class Repository {
    function getImage(choice) {
        image = get file(formattedString("./assets/%s", choice))
        return image
    }
}
```

Sekarang repository bisa mengambil seluruh file PNG pada folder assets.

Pada folder `service` buatlah file `CatService` dan `DogService`

```javascript
class CatService {
    function serve() {
        numberOfCats = number of regex("cat[0-9]+.png") on "./assets"
        catChoice = string(random between(1, numberOfCats))
        catImage = Repository.getImage("cat"+catChoice)
    }
}
```

```javascript
class DogService {
    function serve() {
        numberOfDogs = number of regex("dog[0-9]+.png") on "./assets"
        dogChoice = string(random between(1, numberOfDogs))
        dogImage = Repository.getImage("dog"+dogChoice)
    }
}
```

Pada folder `handler` buatlah file `CatHandler` dan `DogHandler`

```javascript
class CatHandler {
    function handle() { // ini adalah method untuk class CatHandler
        when have request with URL == "/cat" {
            catImage = CatService.serve()
            response = render(catImage)
            return response
        }
    }
}
```

```javascript
class DogHandler {
    function handle() {
        when have request with URL == "/dog" {
            dogImage = DogService.serve()
            response = render(dogImage)
            return response
        }
    }
}
```

Pada main, kita tinggal menambahkan handler:

```javascript
function main() {
    CatHandler.handle()
    DogHandler.handle()
    listen to port(":9001")
}
```

Nah, ketika ada fitur baru kita tidak perlu lagi mengubah kode selain pada main. Kita telah menerapkan Open-Closed
principle.

## Referensi

Martin, R. C. (2018). Clean Architecture: A Craftsman’s Guide to Software Structure and Design. Pearson Professional.