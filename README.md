# Ansible Node Exporter Deployment

[![Ansible](https://img.shields.io/badge/Ansible-2.10+-black.svg?style=for-the-badge&logo=ansible)](https://www.ansible.com/)
[![Prometheus](https://img.shields.io/badge/Prometheus-Node__Exporter-orange.svg?style=for-the-badge&logo=prometheus)](https://prometheus.io/)
[![License](https://img.shields.io/badge/License-MIT-green.svg?style=for-the-badge)](LICENSE)

Repositori ini berisi infrastruktur otomatisasi berbasis **Ansible Playbook** untuk men-deploy, mengonfigurasi, dan menjalankan `node_exporter` (Prometheus metrics agent) pada server Linux target secara *idempotent* dan aman.

## 🚀 Fitur Utama
- **Idempotent Deployment**: Playbook aman dijalankan berulang kali tanpa merusak konfigurasi sistem yang sudah ada.
- **Security-First Approach**: `node_exporter` dijalankan menggunakan *isolated system user* khusus (`node_exporter`) tanpa akses shell (`nologin`) dan tanpa home directory.
- **Systemd Integration**: Otomatis mendaftarkan `node_exporter` sebagai *systemd service unit* yang berjalan otomatis saat server *boot/reboot*.
- **Native SSH Config Integration**: Memanfaatkan konfigurasi lokal `~/.ssh/config` untuk manajemen koneksi yang rapi.

## 📁 Struktur Direktori
```text
.
├── inventory.ini                # Definisi target server & variabel versi
├── deploy-node-exporter.yml     # Core Ansible Playbook
├── ansible.cfg                  # Konfigurasi Ansible environment (opsional)
└── templates/
    └── node_exporter.service.j2 # Template Systemd unit file (Jinja2)
```

## 🛠️ Prasyarat (Prerequisites)
Sebelum menjalankan playbook ini, pastikan Anda telah memenuhi kondisi berikut:

1. Ansible terinstall di mesin lokal/laptop Anda.

2. Koneksi SSH ke remote VPS sudah dikonfigurasi melalui kunci publik (Public Key Authentication) di file ~/.ssh/config. Contoh:

Plaintext

```
Host karavan
    HostName <IP_VPS_ANDA>
    User karavan
    IdentityFile ~/.ssh/id_rsa
   ``` 

3. User remote (karavan) memiliki hak akses sudo tanpa password (NOPASSWD: ALL) diletakkan di baris paling bawah /etc/sudoers agar otomatisasi berjalan lancar tanpa interupsi terminal.

## ⚙️ Konfigurasi & Variabel
1. Inventory (inventory.ini)
Sesuaikan nama host target dengan konfigurasi SSH Anda dan tentukan versi yang ingin di-deploy:

Ini, TOML

```
[monitoring_targets]
karavan

[monitoring_targets:vars]
node_exporter_version=1.11.1
```
2. Systemd Service Template (templates/node_exporter.service.j2)
Secara default, service diaktifkan dengan kolektor tambahan yang sangat berguna untuk pemantauan sistem:

```
ExecStart=/usr/local/bin/node_exporter --collector.systemd --collector.processes
```
## 💻 Cara Penggunaan
1. Clone Repositori Ini
```
git clone [https://github.com/username-anda/nama-repo.git](https://github.com/username-anda/nama-repo.git)
cd nama-repo
```
2. Uji Koneksi (Smoke Test)
Pastikan Ansible dapat berkomunikasi dengan target host menggunakan modul ping:
```
ansible monitoring_targets -i inventory.ini -m ping
```
3. Jalankan Playbook
Eksekusi perintah berikut untuk memulai proses instalasi otomatis:

```
ansible-playbook -i inventory.ini deploy-node-exporter.yml
```
## 🔍 Verifikasi Hasil Instalasi
Setelah proses selesai tanpa error, Anda dapat memastikan agensi monitoring berjalan dengan melakukan curl ke port default (9100) dari laptop Anda (atau langsung di dalam server):

```
curl http://<IP_VPS_ANDA>:9100/metrics
```
Jika mengembalikan tumpukan data metrik teks berupa # HELP node_cpu_seconds_total ..., maka node_exporter telah sukses mengirimkan metrik sistem dan siap ditarik (scraped) oleh Prometheus server.

## 🔒 Catatan Keamanan (Security Notes)
File konfigurasi systemd /etc/systemd/system/node_exporter.service secara ketat diatur kepemilikannya ke root:root dengan permission 0644 untuk mencegah eskalasi hak akses oleh user non-privilese.

Disarankan untuk membatasi akses port 9100 pada firewall VPS (UFW/iptables) Anda agar hanya bisa diakses oleh IP internal spesifik milik server Prometheus Anda.