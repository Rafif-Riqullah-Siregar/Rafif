---
title: Tunneling IP Public ke IP Private

---

# 🌐 Cyber Physical System laboratory's Server IP Tunneling from IP Public to IP Private

**Menggunakan Proxmox, Cloudflare untuk IP Tunneling, CloudHost untuk DNS, dan OpenVPN untuk SSH**

---

## 👋 Perkenalan

Halo semuanya, aku **Rafif Riqullah Siregar**, biasa dipanggil **Rapip** atau **Lek** karena aku asli dari Medan. Aku dari jurusan **S1 Teknik Telekomunikasi Angkatan 47** dan saat ini jadi **Asisten Praktikum Cyber Physical System** angkatan 2023.
Ini media sosial aku yaa:
- Instagram: https://www.instagram.com/rafifriqullah?igsh=YmppdGdmeGgzd2V6&utm_source=qr 
- LinkedIn: https://www.linkedin.com/in/rafif-riqullah-siregar-84b624281?utm_source=share&utm_campaign=share_via&utm_content=profile&utm_medium=ios_app 

Di sini aku bakal kasih tutorial **step-by-step** gimana caranya instalasi core server di lab CPS, gampang kok.

Core Server ini fungsinya buat **tunneling IP dari IP public Telkom University ke IP private**, lalu dijadikan DNS supaya bisa diakses dari mana aja. Kenapa harus SSH atau bisa diakses dari mana aja? karena server ini juga nanti yang bakal jadi **database center** dari lab ini.

**Note:** Dokumentasi ini sesuai dengan konfigurasi server pada tanggal 15 Agustus 2025 (Pengerjaan kurang dari 2 jam kok)


---

## 🔧 Kenapa Kita Pakai Ini?

| Komponen | Alasan |
|---------|--------|
| **Proxmox** | Core dari server kita. Bisa buat container untuk database dan service lainnya. |
| **Cloudflare** | Untuk melakukan IP Tunneling dari IP publik → IP private secara aman dan stabil. |
| **Cloudhost** | Untuk bikin DNS dari IP hasil tunnel biar gampang diakses (lab CPS sendiri beli domain karena murah juga per tahunnya 😁). |
| **OpenVPN** | Untuk akses remote (SSH) ke server dari luar jaringan lab tanpa harus datang langsung. |

---

## 🧰 Perlengkapan yang Dibutuhkan

| Nama | Keterangan |
|------|------------|
| Kabel **Pigtail** | Untuk koneksi dari jalur WiFi Tel-U |
| **Converter** kabel pigtail → LAN | Ubah jalur dari pigtail → jalur LAN |
| Kabel **LAN Straight** | Dari converter → router wifi dan router wifi → server |
| **Router WiFi** *(opsional)* | Dipakai karena lab CPS butuh WiFi lokal juga |
| **Server** + Harddisk | Tempat install Proxmox dan service-server lainnya |
| **Flashdisk 8–32GB** | Untuk install Proxmox OS (walaupun mungkin yang terpakai hanya sekitar 4GB, di sini aku pakai 32GB) |

---

# 🌐 Arsitektur Proyek Server Remote Akses - Cyber Physical System Laboratory

## 📌 Deskripsi Singkat
Proyek ini bertujuan untuk menghubungkan server di lab Cyber Physical System ke internet menggunakan IP publik dari Tel-U, kemudian mengamankannya dengan tunnel via Cloudflare serta konfigurasi DNS untuk akses jarak jauh (remote access).

---

## 🧭 Alur Proyek

### 1. Koneksi Fisik Jaringan
- Hubungkan kabel **pigtail** dari jalur WiFi Tel-U ke perangkat **converter**.
- Sambungkan kabel **LAN**:
  - Dari **converter** ke **router WiFi**.
  - Dari **router WiFi** ke **server**.

📌 *Hasil: Server terhubung ke jaringan Tel-U.*

---

### 2. IP Address dan DNS
- Server memperoleh **IP publik** dari jaringan Tel-U.
- Gunakan **Cloudflare** untuk mengubah IP publik menjadi IP private yang aman.
- Buat **DNS record (A/CNAME)** untuk IP melalui layanan seperti **Cloudhost**.

📌 *Hasil: Server dapat diakses melalui domain, bukan langsung IP.*

---

### 3. Akses Jarak Jauh (Remote Access)
- Konfigurasi **akses SSH** agar server bisa diakses dari mana saja.
- Gunakan domain yang sudah dibuat sebagai endpoint SSH:  
  ```bash
  ssh cpsserver.my.id # disini aku make domain ini untuk DNS nya

# 💾 Download Rufus dan Proxmox

1. **Siapkan flashdisk kosong** (8–32GB). Kalau ada file di dalamnya, pindahin dulu. Colok ke laptop kalian.
2. **Download dan install Rufus**  
   🔗 [Download Rufus](https://rufus.ie/en/) → Pilih versi paling atas.  
   Aku download `rufus-4.9.exe` (standard, Windows x64, 2MB).
   ![Rufus Screenshot](https://hackmd.io/_uploads/Skd4V0auex.png)

3. **Download Proxmox ISO Installer**  
   🔗 [Download Proxmox](https://www.proxmox.com/en/downloads) → Pilih versi paling atas.  
   Aku download `Proxmox VE 9.0 ISO Installer`.
   ![Proxmox Download](https://hackmd.io/_uploads/BJaKHRTueg.png)

4. **Jalankan Rufus dan flash disk Proxmox ke USB**  
   - Pilih flashdisk
   - Masukkan file ISO Proxmox
   - Klik Start dan tunggu sampai selesai

5. **Cek hasilnya**  
   Kalau flashdisk udah gak bisa diakses dan isinya berubah → berarti ISO Proxmox sudah ter-flash.  
   Eject flashdisk dan colok ke server.

---

# 🖥️ Instalasi Proxmox di Server

1. **Nyalakan server** dan tunggu layar biru muncul.  
   Spam tekan `F11` sampai muncul tampilan seperti ini:
   ![Boot Manager](https://hackmd.io/_uploads/rJBeegA_gg.jpg)

2. Tekan `Enter` untuk masuk ke **Boot Manager** → pilih `One-shot UEFI Boot Menu` → tekan enter  
   ![UEFI Boot Menu](https://hackmd.io/_uploads/Hk2KweAuel.jpg)

3. Pilih flashdisk kalian, disini punyaku:  
   `"Disk connected to back USB 2: SanDisk 3.2Gen1"`  
   ![Select Boot Option](https://hackmd.io/_uploads/By9C_gCdge.jpg)

4. Masuk ke menu Proxmox → Pilih `Install Proxmox VE (Terminal UI)`  
   *(Pilih yang terminal biar ringan dan terlihat lebih “server”)*
   ![Install Proxmox](https://hackmd.io/_uploads/rkQnKxROxe.jpg)

5. Di tampilan EULA → Pilih `I Agree`  
   Di ‘Target Harddisk’ → Pilih harddisk kalian (aku cuma 1) → Tekan `Next`  
   ![EULA](https://hackmd.io/_uploads/rJ5JsxCuxe.jpg)  
   ![Target Harddisk](https://hackmd.io/_uploads/SJBeoxCuxe.jpg)

6. Pilih lokasi: Country, Timezone, dan Keyboard Layout → `Next`  
   ![Timezone](https://hackmd.io/_uploads/rJ5gsxRdel.jpg)

7. Di bagian `Management Interface` → isi Hostname, IP Address, Gateway, dan DNS Server  
   (Biasanya IP dan Gateway udah otomatis. Aku isi hostname: `cps.server`)  
   ![Management Interface](https://hackmd.io/_uploads/S1En2g0uxl.jpg)

8. Buat Password dan isi Email → Tekan `Next`  
   ![Password & Email](https://hackmd.io/_uploads/SkcfvWROgg.jpg)

9. Proxmox siap di-install → Pilih `Proxmox VE GNU/Linux` dan tekan `Enter`  
   ![Final Step](https://hackmd.io/_uploads/SJJ6DZAuex.jpg)

10. Setelah proses selesai → Masuk ke **terminal login**  
    Masukkan user (biasanya `root` dan password yang tadi kalian buat untuk login.

---

## ✅ Proxmox Berhasil Di-Install!

Kalau udah sampai sini, Proxmox kalian udah siap buat:
- Dijalankan lewat terminal
- Dipasang container buat database
- Lanjut ke setup **Cloudflare Tunnel**, **DNS**, dan **OpenVPN remote SSH**
---

# ☁️ Setup Cloudflare Tunnel untuk Server CPS

Dengan **Cloudflare Tunnel**, kamu bisa:
- Akses server lewat domain (disini DNS nya `cpsserver.my.id`)
- Sembunyiin IP publik asli (lebih aman)
- Gak perlu buka port di firewall/router
- Remote dari mana aja

---

## 📋 Prasyarat

Sebelum mulai, pastikan:
- Sudah punya akun Cloudflare
- Sudah punya domain (bisa beli di [cloudhost.id](https://cloudhost.id) atau lainnya)
- Domain sudah ditambahkan ke Cloudflare dan status DNS-nya **"Proxied"**
- Cek IP dengan  `ip a` untuk mengetahui IP yang kalian dapatkan

---

## 🧰 1. Install cloudflared

Login ke server kamu (via Proxmox atau SSH), lalu jalankan:

```bash
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb # kalau eror atau gabisa, jangan pake sudo ya
```

Cek versi untuk memastikan berhasil:
```bash
cloudflared --version
```

## 🔑 2. Login ke Cloudflare
```bash
cloudflared tunnel login
```

Salin URL yang muncul
- Buka lewat browser
- Login akun Cloudflare → pilih domain kamu
- Kalau berhasil, file auth akan tersimpan di:
```bash
/root/.cloudflared/cert.pem
```

## 🌐 3. Buat Tunnel Baru
```bash
cloudflared tunnel create cps-server
```

Setelah berhasil, akan muncul output seperti ini:
```bash
Created tunnel cps-server with ID 95931d2b-8144-4700-a577-9bb17aae0578
```
Catat Tunnel ID-nya.

## ⚙️ 4. Buat File Konfigurasi
Buat folder config:
```bash
mkdir -p /root/.cloudflared
```

Edit file konfigurasinya:
```bash
nano ~/.cloudflared/config.yml
```

Isi dengan:
```bash
tunnel: 95931d2b-8144-4700-a577-9bb17aae0578 # ganti dengan Tunnel ID
credentials-file: /root/.cloudflared/95931d2b-8144-4700-a577-9bb17aae0578.json # ganti dengan Tunnel ID (.json jangan dihapus)

ingress:
  - hostname: cpsserver.my.id # DNS kita
    service: https://192.168.0.117:8006 # IP yang didapat dari Tel-U, 8006 adalah port TCP yg biasa digunakan untuk Proxmox
    originRequest:
      insecureSkipVerify: true  # Karena Proxmox pakai self-signed cert
  - service: http_status:404
```

Pastikan kamu mengganti:

- `tunnel` → pakai nama tunnel kamu
- `credentials-file` → sesuai dengan nama file JSON yang muncul
- `hostname` → ganti dengan domain kamu sendiri
- `port:8006` → port service yang ingin kamu expose (misalnya: SSH)
![Screenshot 2025-08-16 230019](https://hackmd.io/_uploads/BykSsXAugl.png)

## 🌍 5. Buat DNS Record Otomatis
```bash
cloudflared tunnel route dns cps-server cpsserver.my.id
```

Kalau berhasil, akan otomatis muncul di dashboard DNS Cloudflare.

▶️ 6. Jalankan Tunnel
---
```bash
cloudflared tunnel run cps-server
```

Kalau berhasil, tunnel akan aktif dan domain kamu bisa digunakan untuk mengakses service di server (misalnya SSH).
![Screenshot 2025-08-16 225828](https://hackmd.io/_uploads/Bk4ls70uxx.png)

🔄 7. Autostart Saat Boot
---
Biar tunnel selalu aktif walaupun server di-reboot:
```bash
cloudflared service install
```

Tunnel akan otomatis jalan sebagai system service.
![Screenshot 2025-08-16 230230](https://hackmd.io/_uploads/Byfnsm0_ee.png)

Edit file di `nano /etc/systemd/system/cloudflared.service`. Isinya seperti ini:
```bash
[Unit]
Description=cloudflared
After=network.target

[Service]
TimeoutStartSec=0
Type=simple
ExecStart=/usr/bin/cloudflared --config /root/.cloudflared/config.yml tunnel run
Restart=always
RestartSec=5s
User=root
WorkingDirectory=/root/

[Install]
WantedBy=multi-user.target
```
![Screenshot (5)](https://hackmd.io/_uploads/SksUxsbYxx.png)


Setelah diganti, lakukan command ini:
```bash
systemctl daemon-reload
systemctl enable cloudflared
systemctl start cloudflared
systemctl status cloudflared
```

🧪 8. Tes SSH via Domain
---
Dari laptop/komputer yang kamu pakai (pastikan beda jaringan ya), buka browser dan ketik:
```bash
cpsserver.my.id
```
![Screenshot 2025-08-16 230342](https://hackmd.io/_uploads/ByiI2Q0_xe.png)

Pastikan:

- SSH aktif di server (port 8006 terbuka secara internal)
- Domain diarahkan ke service SSH di config

| Masalah | Solusi |
|------|------------|
| `cloudflared tunnel login` gagal | Pastikan server terkoneksi ke internet |
| Tunnel tidak aktif | Cek file config & path credentials |
| SSH gagal | Pastikan SSH service jalan, user & port benar |
| Port bentrok | Ubah port service di config atau SSH |

❗Hal yang Perlu di Perhatikan
---
1. Jalankan di browser https://www.cloudflare.com/ lalu login ke akun kalian. Pergi ke `Zero Trust` → `Networks` → `Tunnels`.
    - `Status` harus kategori `Healthy` dan `Connector ID` harus ada secara otomatis.
    - Tambahkan private networks, sesuaikan dengan IP kalian (cek di `ip a` pada terminal).
    - Tambahkan Public hostname atau DNS yang sudah kalian buat dan sesuaikan dengan IP dan prefix yang kalian inginkan.
    - Jangan lupa untuk mengubah status menjadi **ON** pada `additional application settings` → `TLS` → `No TLS Verify`.
2. Jalankan di browser https://www.cloudflare.com/ lalu login ke akun kalian. Pergi ke `Zero Trust` → `Networks` → `Routes`.
    - Sesuaikan dengan IP dan Prefix kalian.
    - Pada bagian `Tunnel`, sesuaikan dengan `cps-server` (atau tunnel yang kalian buat).
![Screenshot 2025-08-16 230800](https://hackmd.io/_uploads/SyNE6m0_gg.png)
![Screenshot 2025-08-16 230838](https://hackmd.io/_uploads/Syi46XRdlx.png)
![Screenshot 2025-08-16 230849](https://hackmd.io/_uploads/ByDHpmAOgx.png)
![Screenshot 2025-08-16 231220](https://hackmd.io/_uploads/rylfCQCulx.png)
![Screenshot 2025-08-16 232150](https://hackmd.io/_uploads/rJSVgVRdle.png)

# 🌐 Setup DNS di Cloudhost untuk Server

Setelah tunnel Cloudflare berhasil dibuat dan berjalan, kita akan **mendaftarkan domain** atau **subdomain** menggunakan layanan dari [Cloudhost.id](https://cloudhost.id), supaya server bisa diakses dari mana saja menggunakan nama domain (`cpsserver.my.id`).

---

## 📋 Yang Perlu Disiapkan

- Akun aktif di [https://cloudhost.id](https://cloudhost.id)
- Sudah membeli domain (misalnya: `cpsserver.my.id`)
- Akses ke dashboard Cloudflare
- Tunnel sudah aktif dengan `cloudflared`
- Subdomain atau domain yang ingin diarahkan

---

## 🛠️ 1. Login ke Dashboard Cloudhost

1. Kunjungi [https://cloudhost.id/login](https://cloudhost.id/login)
2. Masuk menggunakan akun yang telah didaftarkan
3. Buka menu **Layanan** > **Domain Saya**
4. Pilih domain yang ingin kamu kelola (misal: `cps-lab.site`)

---

## 🔧 2. Arahkan Nameserver ke Cloudflare

Kalau belum dilakukan:
1. Masuk ke **Kelola Domain**
2. Pilih tab **Nameserver**
3. Ganti nameserver default menjadi:
# 🌐 Cyber Physical System laboratory's Server IP Tunneling from IP Public to IP Private

**Menggunakan Proxmox, Cloudflare untuk IP Tunneling, CloudHost untuk DNS, dan OpenVPN untuk SSH**

---

## 👋 Perkenalan

Halo semuanya, aku **Rafif Riqullah Siregar**, biasa dipanggil **Rapip** atau **Lek** karena aku asli dari Medan. Aku dari jurusan **S1 Teknik Telekomunikasi Angkatan 47** dan saat ini jadi **Asisten Praktikum Cyber Physical System** angkatan 2023.
Ini media sosial aku yaa:
- Instagram: https://www.instagram.com/rafifriqullah?igsh=YmppdGdmeGgzd2V6&utm_source=qr 
- LinkedIn: https://www.linkedin.com/in/rafif-riqullah-siregar-84b624281?utm_source=share&utm_campaign=share_via&utm_content=profile&utm_medium=ios_app 

Di sini aku bakal kasih tutorial **step-by-step** gimana caranya instalasi core server di lab CPS, gampang kok.

Core Server ini fungsinya buat **tunneling IP dari IP public Telkom University ke IP private**, lalu dijadikan DNS supaya bisa diakses dari mana aja. Kenapa harus SSH atau bisa diakses dari mana aja? karena server ini juga nanti yang bakal jadi **database center** dari lab ini.

**Note:** Dokumentasi ini sesuai dengan konfigurasi server pada tanggal 15 Agustus 2025 (Pengerjaan kurang dari 2 jam kok)


---

## 🔧 Kenapa Kita Pakai Ini?

| Komponen | Alasan |
|---------|--------|
| **Proxmox** | Core dari server kita. Bisa buat container untuk database dan service lainnya. |
| **Cloudflare** | Untuk melakukan IP Tunneling dari IP publik → IP private secara aman dan stabil. |
| **Cloudhost** | Untuk bikin DNS dari IP hasil tunnel biar gampang diakses (lab CPS sendiri beli domain karena murah juga per tahunnya 😁). |
| **OpenVPN** | Untuk akses remote (SSH) ke server dari luar jaringan lab tanpa harus datang langsung. |

---

## 🧰 Perlengkapan yang Dibutuhkan

| Nama | Keterangan |
|------|------------|
| Kabel **Pigtail** | Untuk koneksi dari jalur WiFi Tel-U |
| **Converter** kabel pigtail → LAN | Ubah jalur dari pigtail → jalur LAN |
| Kabel **LAN Straight** | Dari converter → router wifi dan router wifi → server |
| **Router WiFi** *(opsional)* | Dipakai karena lab CPS butuh WiFi lokal juga |
| **Server** + Harddisk | Tempat install Proxmox dan service-server lainnya |
| **Flashdisk 8–32GB** | Untuk install Proxmox OS (walaupun mungkin yang terpakai hanya sekitar 4GB, di sini aku pakai 32GB) |

---

# 🌐 Arsitektur Proyek Server Remote Akses - Cyber Physical System Laboratory

## 📌 Deskripsi Singkat
Proyek ini bertujuan untuk menghubungkan server di lab Cyber Physical System ke internet menggunakan IP publik dari Tel-U, kemudian mengamankannya dengan tunnel via Cloudflare serta konfigurasi DNS untuk akses jarak jauh (remote access).

---

## 🧭 Alur Proyek

### 1. Koneksi Fisik Jaringan
- Hubungkan kabel **pigtail** dari jalur WiFi Tel-U ke perangkat **converter**.
- Sambungkan kabel **LAN**:
  - Dari **converter** ke **router WiFi**.
  - Dari **router WiFi** ke **server**.

📌 *Hasil: Server terhubung ke jaringan Tel-U.*

---

### 2. IP Address dan DNS
- Server memperoleh **IP publik** dari jaringan Tel-U.
- Gunakan **Cloudflare** untuk mengubah IP publik menjadi IP private yang aman.
- Buat **DNS record (A/CNAME)** untuk IP melalui layanan seperti **Cloudhost**.

📌 *Hasil: Server dapat diakses melalui domain, bukan langsung IP.*

---

### 3. Akses Jarak Jauh (Remote Access)
- Konfigurasi **akses SSH** agar server bisa diakses dari mana saja.
- Gunakan domain yang sudah dibuat sebagai endpoint SSH:  
  ```bash
  ssh cpsserver.my.id # disini aku make domain ini untuk DNS nya

# 💾 Download Rufus dan Proxmox

1. **Siapkan flashdisk kosong** (8–32GB). Kalau ada file di dalamnya, pindahin dulu. Colok ke laptop kalian.
2. **Download dan install Rufus**  
   🔗 [Download Rufus](https://rufus.ie/en/) → Pilih versi paling atas.  
   Aku download `rufus-4.9.exe` (standard, Windows x64, 2MB).
   ![Rufus Screenshot](https://hackmd.io/_uploads/Skd4V0auex.png)

3. **Download Proxmox ISO Installer**  
   🔗 [Download Proxmox](https://www.proxmox.com/en/downloads) → Pilih versi paling atas.  
   Aku download `Proxmox VE 9.0 ISO Installer`.
   ![Proxmox Download](https://hackmd.io/_uploads/BJaKHRTueg.png)

4. **Jalankan Rufus dan flash disk Proxmox ke USB**  
   - Pilih flashdisk
   - Masukkan file ISO Proxmox
   - Klik Start dan tunggu sampai selesai

5. **Cek hasilnya**  
   Kalau flashdisk udah gak bisa diakses dan isinya berubah → berarti ISO Proxmox sudah ter-flash.  
   Eject flashdisk dan colok ke server.

---

# 🖥️ Instalasi Proxmox di Server

1. **Nyalakan server** dan tunggu layar biru muncul.  
   Spam tekan `F11` sampai muncul tampilan seperti ini:
   ![Boot Manager](https://hackmd.io/_uploads/rJBeegA_gg.jpg)

2. Tekan `Enter` untuk masuk ke **Boot Manager** → pilih `One-shot UEFI Boot Menu` → tekan enter  
   ![UEFI Boot Menu](https://hackmd.io/_uploads/Hk2KweAuel.jpg)

3. Pilih flashdisk kalian, disini punyaku:  
   `"Disk connected to back USB 2: SanDisk 3.2Gen1"`  
   ![Select Boot Option](https://hackmd.io/_uploads/By9C_gCdge.jpg)

4. Masuk ke menu Proxmox → Pilih `Install Proxmox VE (Terminal UI)`  
   *(Pilih yang terminal biar ringan dan terlihat lebih “server”)*
   ![Install Proxmox](https://hackmd.io/_uploads/rkQnKxROxe.jpg)

5. Di tampilan EULA → Pilih `I Agree`  
   Di ‘Target Harddisk’ → Pilih harddisk kalian (aku cuma 1) → Tekan `Next`  
   ![EULA](https://hackmd.io/_uploads/rJ5JsxCuxe.jpg)  
   ![Target Harddisk](https://hackmd.io/_uploads/SJBeoxCuxe.jpg)

6. Pilih lokasi: Country, Timezone, dan Keyboard Layout → `Next`  
   ![Timezone](https://hackmd.io/_uploads/rJ5gsxRdel.jpg)

7. Di bagian `Management Interface` → isi Hostname, IP Address, Gateway, dan DNS Server  
   (Biasanya IP dan Gateway udah otomatis. Aku isi hostname: `cps.server`)  
   ![Management Interface](https://hackmd.io/_uploads/S1En2g0uxl.jpg)

8. Buat Password dan isi Email → Tekan `Next`  
   ![Password & Email](https://hackmd.io/_uploads/SkcfvWROgg.jpg)

9. Proxmox siap di-install → Pilih `Proxmox VE GNU/Linux` dan tekan `Enter`  
   ![Final Step](https://hackmd.io/_uploads/SJJ6DZAuex.jpg)

10. Setelah proses selesai → Masuk ke **terminal login**  
    Masukkan user (biasanya `root` dan password yang tadi kalian buat untuk login.

---

## ✅ Proxmox Berhasil Di-Install!

Kalau udah sampai sini, Proxmox kalian udah siap buat:
- Dijalankan lewat terminal
- Dipasang container buat database
- Lanjut ke setup **Cloudflare Tunnel**, **DNS**, dan **OpenVPN remote SSH**
---

# ☁️ Setup Cloudflare Tunnel untuk Server CPS

Dengan **Cloudflare Tunnel**, kamu bisa:
- Akses server lewat domain (disini DNS nya `cpsserver.my.id`)
- Sembunyiin IP publik asli (lebih aman)
- Gak perlu buka port di firewall/router
- Remote dari mana aja

---

## 📋 Prasyarat

Sebelum mulai, pastikan:
- Sudah punya akun Cloudflare
- Sudah punya domain (bisa beli di [cloudhost.id](https://cloudhost.id) atau lainnya)
- Domain sudah ditambahkan ke Cloudflare dan status DNS-nya **"Proxied"**
- Cek IP dengan  `ip a` untuk mengetahui IP yang kalian dapatkan

---

## 🧰 1. Install cloudflared

Login ke server kamu (via Proxmox atau SSH), lalu jalankan:

```bash
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb # kalau eror atau gabisa, jangan pake sudo ya
```

Cek versi untuk memastikan berhasil:
```bash
cloudflared --version
```

## 🔑 2. Login ke Cloudflare
```bash
cloudflared tunnel login
```

Salin URL yang muncul
- Buka lewat browser
- Login akun Cloudflare → pilih domain kamu
- Kalau berhasil, file auth akan tersimpan di:
```bash
/root/.cloudflared/cert.pem
```

## 🌐 3. Buat Tunnel Baru
```bash
cloudflared tunnel create cps-server
```

Setelah berhasil, akan muncul output seperti ini:
```bash
Created tunnel cps-server with ID 95931d2b-8144-4700-a577-9bb17aae0578
```
Catat Tunnel ID-nya.

## ⚙️ 4. Buat File Konfigurasi
Buat folder config:
```bash
mkdir -p /root/.cloudflared
```

Edit file konfigurasinya:
```bash
nano ~/.cloudflared/config.yml
```

Isi dengan:
```bash
tunnel: 95931d2b-8144-4700-a577-9bb17aae0578 # ganti dengan Tunnel ID
credentials-file: /root/.cloudflared/95931d2b-8144-4700-a577-9bb17aae0578.json # ganti dengan Tunnel ID (.json jangan dihapus)

ingress:
  - hostname: cpsserver.my.id # DNS kita
    service: https://192.168.0.117:8006 # IP yang didapat dari Tel-U, 8006 adalah port TCP yg biasa digunakan untuk Proxmox
    originRequest:
      insecureSkipVerify: true  # Karena Proxmox pakai self-signed cert
  - service: http_status:404
```

Pastikan kamu mengganti:

- `tunnel` → pakai nama tunnel kamu
- `credentials-file` → sesuai dengan nama file JSON yang muncul
- `hostname` → ganti dengan domain kamu sendiri
- `port:8006` → port service yang ingin kamu expose (misalnya: SSH)
![Screenshot 2025-08-16 230019](https://hackmd.io/_uploads/BykSsXAugl.png)

## 🌍 5. Buat DNS Record Otomatis
```bash
cloudflared tunnel route dns cps-server cpsserver.my.id
```

Kalau berhasil, akan otomatis muncul di dashboard DNS Cloudflare.

▶️ 6. Jalankan Tunnel
---
```bash
cloudflared tunnel run cps-server
```

Kalau berhasil, tunnel akan aktif dan domain kamu bisa digunakan untuk mengakses service di server (misalnya SSH).
![Screenshot 2025-08-16 225828](https://hackmd.io/_uploads/Bk4ls70uxx.png)

🔄 7. Autostart Saat Boot
---
Biar tunnel selalu aktif walaupun server di-reboot:
```bash
cloudflared service install
```

Tunnel akan otomatis jalan sebagai system service.
![Screenshot 2025-08-16 230230](https://hackmd.io/_uploads/Byfnsm0_ee.png)

Edit file di `nano /etc/systemd/system/cloudflared.service`. Isinya seperti ini:
```bash
[Unit]
Description=cloudflared
After=network.target

[Service]
TimeoutStartSec=0
Type=simple
ExecStart=/usr/bin/cloudflared --config /root/.cloudflared/config.yml tunnel run
Restart=always
RestartSec=5s
User=root
WorkingDirectory=/root/

[Install]
WantedBy=multi-user.target
```
![Screenshot (5)](https://hackmd.io/_uploads/SksUxsbYxx.png)


Setelah diganti, lakukan command ini:
```bash
systemctl daemon-reload
systemctl enable cloudflared
systemctl start cloudflared
systemctl status cloudflared
```

🧪 8. Tes SSH via Domain
---
Dari laptop/komputer yang kamu pakai (pastikan beda jaringan ya), buka browser dan ketik:
```bash
cpsserver.my.id
```
![Screenshot 2025-08-16 230342](https://hackmd.io/_uploads/ByiI2Q0_xe.png)

Pastikan:

- SSH aktif di server (port 8006 terbuka secara internal)
- Domain diarahkan ke service SSH di config

| Masalah | Solusi |
|------|------------|
| `cloudflared tunnel login` gagal | Pastikan server terkoneksi ke internet |
| Tunnel tidak aktif | Cek file config & path credentials |
| SSH gagal | Pastikan SSH service jalan, user & port benar |
| Port bentrok | Ubah port service di config atau SSH |

❗Hal yang Perlu di Perhatikan
---
1. Jalankan di browser https://www.cloudflare.com/ lalu login ke akun kalian. Pergi ke `Zero Trust` → `Networks` → `Tunnels`.
    - `Status` harus kategori `Healthy` dan `Connector ID` harus ada secara otomatis.
    - Tambahkan private networks, sesuaikan dengan IP kalian (cek di `ip a` pada terminal).
    - Tambahkan Public hostname atau DNS yang sudah kalian buat dan sesuaikan dengan IP dan prefix yang kalian inginkan.
    - Jangan lupa untuk mengubah status menjadi **ON** pada `additional application settings` → `TLS` → `No TLS Verify`.
2. Jalankan di browser https://www.cloudflare.com/ lalu login ke akun kalian. Pergi ke `Zero Trust` → `Networks` → `Routes`.
    - Sesuaikan dengan IP dan Prefix kalian.
    - Pada bagian `Tunnel`, sesuaikan dengan `cps-server` (atau tunnel yang kalian buat).
![Screenshot 2025-08-16 230800](https://hackmd.io/_uploads/SyNE6m0_gg.png)
![Screenshot 2025-08-16 230838](https://hackmd.io/_uploads/Syi46XRdlx.png)
![Screenshot 2025-08-16 230849](https://hackmd.io/_uploads/ByDHpmAOgx.png)
![Screenshot 2025-08-16 231220](https://hackmd.io/_uploads/rylfCQCulx.png)
![Screenshot 2025-08-16 232150](https://hackmd.io/_uploads/rJSVgVRdle.png)

# 🌐 Setup DNS di Cloudhost untuk Server

Setelah tunnel Cloudflare berhasil dibuat dan berjalan, kita akan **mendaftarkan domain** atau **subdomain** menggunakan layanan dari [Cloudhost.id](https://cloudhost.id), supaya server bisa diakses dari mana saja menggunakan nama domain (`cpsserver.my.id`).

---

## 📋 Yang Perlu Disiapkan

- Akun aktif di [https://cloudhost.id](https://cloudhost.id)
- Sudah membeli domain (misalnya: `cpsserver.my.id`)
- Akses ke dashboard Cloudflare
- Tunnel sudah aktif dengan `cloudflared`
- Subdomain atau domain yang ingin diarahkan

---

## 🛠️ 1. Login ke Dashboard Cloudhost

1. Kunjungi [https://cloudhost.id/login](https://cloudhost.id/login)
2. Masuk menggunakan akun yang telah didaftarkan
3. Buka menu **Layanan** > **Domain Saya**
4. Pilih domain yang ingin kamu kelola (misal: `cps-lab.site`)

---

## 🔧 2. Arahkan Nameserver ke Cloudflare

Kalau belum dilakukan:
1. Masuk ke **Kelola Domain**
2. Pilih tab **Nameserver**
3. Ganti nameserver default menjadi:
