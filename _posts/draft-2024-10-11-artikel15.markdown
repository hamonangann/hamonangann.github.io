---
layout: post
title: "Mengenal Concurrency pada Bahasa Pemrograman Go dengan Goroutine"
categories: tips
---

Salah satu konsep yang cukup [tricky](https://hamonangann.github.io/tips/2024/09/30/artikel14.html) pada bahasa pemrograman Go (lang) adalah Goroutine. Goroutine umum digunakan untuk mengimplementasi concurrency pada program komputer.

## Perbedaan Goroutine dengan Thread

Goroutine **berbeda** dari thread, meskipun fungsinya serupa dengan thread. Bedanya, goroutine dibuat di bagian user space dari memori dan tidak berkomunikasi dengan kernel sistem operasi, melainkan dikelola oleh Go runtime itu sendiri sehingga lebih cepat.

Selain itu, goroutine juga berukuran sangat ringan. Fungsi dari Goroutine ini adalah menjalankan serangkaian komputasi secara asinkronus bersamaan dengan goroutine-goroutine lainnya.

Faktanya, fungsi main() di Go juga dijalankan oleh sebuah Goroutine.

## Membuat Goroutine baru

Membuat goroutine sangat sederhana, kita tinggal perlu menjalankan fungsi dengan awalan go.

Untuk jalankan kode, salah satu caranya adalah kita dapat salin kode-kode di artikel ini ke [Go Playground](https://go.dev/playground)

```go
package main

import (
	"fmt"
	"time"
)

func hello(name string) {
	fmt.Printf("Hi, %s!\n", name)
}

func main() {
	go hello("Bob")
	time.Sleep(time.Second)
	fmt.Println("How are you?")
}
```

Pada kode di atas, goroutine main() menjalankan sleep selama 1 detik, lalu mencetak "How are you?". Sembari kedua kode tersebut dijalankan, sebuah goroutine baru dijalankan untuk mencetak "Hi, Bob!".

Di sini, hello() berjalankan secara asinknronus. Untuk mendapatkan gambaran lebih jelas terkait proses asinkronus, coba jalankan kode di atas dengan menghapus "time" (pada import) dan baris kode "time.Sleep...", lalu perhatikan apa yang dicetak terlebih dahulu oleh program.

## Komunikasi antar Goroutine dengan Channel

Goroutine menjadi powerful karena bisa berkomunikasi satu sama lain. Ini bisa dicapai dengan menggunakan channel.

```go
package main

import (
	"fmt"
)

func hello(name string, done chan bool) {
	fmt.Printf("Hi, %s!\n", name)
	<-done
}

func main() {
	ch := make(chan bool)
	go hello("Bob", ch)

	ch <- true
	fmt.Println("How are you?")
}
```

Ketika program di atas dijalankan, program menjalankan goroutine hello() dengan parameter channel ch. Goroutine tersebut kemudian mencetak "Hi, Bob!" dan kemudian memasukkan nilai `true` ke dalam channel.

Sintaks `<-` digunakan untuk mengarahkan nilai ke atau dari channel. Sebagai contoh `done <- true` artinya memasukkan nilai `true` ke dalam channel `done`, sedangkan `var x bool = <- ch` artinya mengeluarkan nilai dari channel `done` untuk selanjutnya menjadi nilai dari variabel x. Setelah nilai dikeluarkan, channel menjadi kosong dan siap dipakai ulang.

Perhatikan bahwa kita tidak lagi memerlukan time.Sleep untuk memastikan "Hi, Bob!" selalu dicetak lebih dulu dari "How are you?". Ini karena baris `<- ch` bersifat blocking, yaitu menunggu sampai channel menerima nilai tertentu. Setelah channel menerima nilai `true` dan goroutine hello() selesai, barulah baris berikutnya mencetak "How are you?".


## Channel dengan buffer

Pada kode sebelumnya, nilai dari channel diterima lalu dibuang begitu saja. Sebenarnya, kita dapat memodifikasi kode di atas agar nilai dari channel tersebut dapat digunakan dengan assign ke variabel dengan contoh sebagai berikut.

```go
package main

import (
	"fmt"
)

func hello(name string, done chan bool) {
	fmt.Printf("Hi, %s!\n", name)
	done <- true
}

func main() {
	ch := make(chan bool)
	go hello("Bob", ch)

	var doneHello bool = <- ch
	if doneHello {
		fmt.Println("How are you?")
	}
}
```

Dengan demikian, doneHello akan berisi `true` dan kode berlanjut.

Bagaimana jika sinyal `done` perlu dikirimkan dari main seperti kode berikut?

```go
package main

import (
	"fmt"
	"time"
)

func hello(name string, done chan bool) {
	time.Sleep(5 * time.Second)
    fmt.Println("Hi, %s!\n", name)
	<- done
}

func main() {
	ch := make(chan bool)
	go hello("Bob", ch)
	
	ch <- true
	
	fmt.Println("How are you?")
}
```

Pada program di atas, program tidak langsung mengeluarkan output. Ketika goroutine hello() dijalankan, program sleep dulu 5 detik, mencetak "Hi, Bob!", barulah nilai dari channel done diterima.

Akan tetapi, yang terjadi di fungsi main() adalah baris `ch <- true` blocking. Ia menunggu nilainya diterima dulu di hello() yang memakan waktu sekitar 5 detik, baru bisa lanjutk baris berikutnya (mencetak "How are you").

Bagaimana jika kita ingin mencetak "How are you terlebih dahulu"? Sebetulnya, kita sudah mengakali goroutine hello() dengan sleep 5 detik. Yang perlu diatasi adalah membuat channel kita bisa menyimpan boolean di memori, sehingga tidak mem-block kode-kode berikutnya.

Kita ubah sedikit channelnya sebagai berikut.

```go
package main

import (
	"fmt"
	"time"
)

func hello(name string, done chan bool) {
	time.Sleep(5 * time.Second)
    fmt.Println("Hi, %s!\n", name)
	<- done
}

func main() {
	ch := make(chan bool, 1)
	go hello("Bob", ch)
	
	ch <- true
	
	fmt.Println("How are you?")
}
```