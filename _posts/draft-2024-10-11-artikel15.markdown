---
layout: post
title: "Mengenal Concurrency pada Bahasa Pemrograman Go dengan Goroutine"
categories: tips
---

Salah satu konsep yang cukup [tricky](https://hamonangann.github.io/tips/2024/09/30/artikel14.html) pada bahasa pemrograman Go (lang) adalah Goroutine. Goroutine umum digunakan untuk mengimplementasi concurrency pada program komputer.

## Perbedaan Goroutine dengan Thread

Goroutine **berbeda** dari thread. Sebuah goroutine berukuran sangat ringan dan sebuah thread bawaan OS dapat menampung banyak goroutine sekaligus. Fungsi dari Goroutine ini adalah menjalankan serangkaian komputasi secara asinkronus (tidak mesti berurutan) bersamaan dengan Goroutine-goroutine lainnya.

Faktanya, fungsi main() di Go juga dijalankan oleh sebuah Goroutine.

## Membuat Goroutine baru

## Komunikasi antar Goroutine dengan Channel

## Mekanisme Locking dengan Mutex

