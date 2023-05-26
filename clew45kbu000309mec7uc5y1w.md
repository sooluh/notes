---
title: "Cloudflare Tunnel untuk Mengexpose Localhost"
datePublished: Mon Mar 06 2023 00:58:14 GMT+0000 (Coordinated Universal Time)
cuid: clew45kbu000309mec7uc5y1w
slug: cloudflared
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685071051499/3823d6b2-cda4-4447-9acf-76c0e1f13b70.png
tags: cloudflare, tunnel, cloudflared

---

Dari beberapa tahun yang lalu aku telah mengenal [ngrok](https://ngrok.com) sebagai layanan yang dapat mengexpose localhost ke internet. Namun setup yang cukup rempong atau ribet membuat aku malas menggunakannya, terlebih saat subdomain atau alamat yang diberikan random, setidaknya untuk versi gratisnya.

Kemudian muncul serveo yang menawarkan kemudahan penggunaanya hanya dengan perintah `ssh` di terminal, dan itu gratis tanpa akun. Namun sayang sekali karena banyaknya penyalahgunaan terutama untuk [phising](https://id.wikipedia.org/wiki/Pengelabuan), layanan tersebut akhirnya ditutup. Dan masih banyak lagi layanan-layanan yang serupa.

Aku mencoba menggunakan Cloudflare untuk DNS management yang tentu karena gratislah aku menggunakannya. Dan aku baru tahu kalau ternyata Cloudflare memiliki layanan tunneling untuk mengexpose localhost ke internet. Layanan tersebut adalah [`cloudflared`](https://developers.cloudflare.com/cloudflare-one/glossary/#cloudflared).

Mari kita lihat apa saja fitur yang ditawarkan `cloudflared` ini, berikut diantaranya:  
🤑 Bebas digunakan  
💎 Tidak ada rate limit  
🔒 Dukungan HTTPS  
🚫 Tidak akan mengexpose IP

Pertama, kita mulai dengan memasang paket `cloudflared` terlebih dahulu. Kita bisa referensikan ke dokumentasi resminya [disini](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/). Tersedia untuk Linux, macOS, Windows, bahkan Docker imagenya.

Setelah terpasang, untuk memverifikasi bahwa paket `cloudflared` ini sudah terpasang atau belum, kamu bisa jalankan perintah berikut.

```bash
cloudflared --version
```

Sebelum melangkah lebih jauh, pastikan kamu sudah terdaftar di Cloudflare dan sudah ada domain yang terparkir, nantinya domain tersebut akan dijadikan subdomain/alamat yang digunakan untuk tunneling ke localhost kita.

Kemudian kita harus login terlebih dahulu menggunakan `cloudflared` tersebut dengan akun yang sudah terdaftar di Cloudflare, kamu bisa ketikkan perintah dibawah ini dan masukkan kredensial saat jendela browser terbuka.

```bash
cloudflared tunnel login
```

Setelah masuk ke akun, kamu diharuskan memilih domain untuk dijadikan alamat tunelling, kemudian klik tombol Authorize untuk mengotorisasi.

Sekarang, mari kita coba untuk membuat tunnel pertama dengan menjalankan perintah seperti dibawah ini.

```bash
cloudflared tunnel create namaaplikasi
```

Selanjutnya tinggal buat entry subdomain custom baru dengan perintah dibawah ini.

```bash
cloudflared tunnel route dns namaaplikasi contoh.domain.com
```

Langkah terakhir, pastikan aplikasi kamu sudah jalan di local terlebih dahulu, maka aplikasi tersebut bisa di expose dengan mudah menggunakan perintah berikut.

```bash
cloudflared tunnel --url http://localhost:8080 run namaaplikasi
```

Dan tada! aplikasi dari localhost kamu sudah bisa diakses oleh siapapun yang terhubung ke internet.

Jangan lupa untuk menyesuaikan perintah-perintah diatas seperti `namaaplikasi`, `contoh.domain.com` dan `http://localhost:8080` sesuai dengan keadaan aplikasi yang ingin kamu expose.

Terima kasih.