

| File                          | isi                                                   | Relevansi SOC                             |
| ----------------------------- | ----------------------------------------------------- | ----------------------------------------- |
| `/var/log/auth.log`           | Login (berhasil/gagal), sudo, SSH, PAM                | Deteksi brute force, privilege escalation |
| `/var/log/syslog`             | Semua pesan sistem kecuali auth (bisa berbeda distro) | Service crash, error tak wajar            |
| `/var/log/kern.log`           | Kernel ring buffer (hardware, driver, OOM)            | Hardware error, driver aneh               |
| `/var/log/dpkg.log`           | Install, upgrade, remove package (Debian/Ubuntu)      | Software yang dipasang tanpa izin         |
| `/var/log/apache2/access.log` | Setiap request HTTP ke web server                     | Web attack, scanning                      |
| `/var/log/faillog`            | Login gagal (format binary, gunakan `faillog -a`)     | Brute force tracking                      |


Jika sebuah alamat IP muncul **100 kali** atau lebih dalam log `auth.log` dengan pola `"Failed password"` hanya dalam rentang **5 menit**, maka itu merupakan indikasi kuat dari Serangan Brute Force.

Penyerang (atau malware) sedang mencoba menebak password secara sistematis dan cepat menggunakan skrip atau bot. Ini bisa berupa:

- Serangan terhadap **SSH** (port 22)
    
- Serangan terhadap **FTP, SMTP, atau layanan lain** yang loginnya tercatat di `auth.log`


### Contoh

#1
```
2026-04-23T19:49:22.956266+07:00 debian unix_chkpwd[66394]: password check failed for user (kaii)
```


| Bagian                                  | Artinya                                                                                                                                   |
| --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `2026-04-23T19:49:22.956266+07:00`      | Timestamp dengan format ISO 8601                                                                                                          |
| `debian`                                | Hostname mesin tempat log dibuat                                                                                                          |
| `unix_chkpwd[66394]`                    | Nama proses utilitas pengecekan password (`unix_chkpwd`) beserta PID (66394)                                                              |
| `password check failed for user (kaii)` | Pesan event: pengecekan password gagal untuk user `kaii` (biasanya terjadi saat login lokal, `su`, atau `sudo` memasukkan password salah) |

#2
```
2026-04-23T19:49:25.347202+07:00 debian sudo:     kaii : 3 incorrect password attempts ; TTY=pts/5 ; PWD=/home/kaii ; USER=root ; COMMAND=/usr/bin/apt update
```


| Bagian                             | Artinya                                                                                      |
| ---------------------------------- | -------------------------------------------------------------------------------------------- |
| `2026-04-23T19:49:25.347202+07:00` | Timestamp ISO 8601 (waktu kejadian)                                                          |
| `debian`                           | Hostname mesin                                                                               |
| `sudo`                             | Nama proses (sudo) – tanpa PID karena biasanya dicatat oleh syslog terpisah                  |
| `kaii`                             | Username yang menjalankan perintah sudo                                                      |
| `3 incorrect password attempts`    | Jumlah percobaan password salah yang dilakukan user `kaii` sebelum berhasil atau gagal total |
| `TTY=pts/5`                        | Terminal pseudo-terminal tempat perintah dijalankan                                          |
| `PWD=/home/kaii`                   | Working directory saat menjalankan sudo                                                      |
| `USER=root`                        | User target yang ingin diperoleh (dengan sudo menjadi root)                                  |
| `COMMAND=/usr/bin/apt update`      | Perintah yang dicoba dijalankan dengan hak akses root                                        |


# journalctl vs /var/log/* – Kapan Pakai yang Mana?

## 📖 `journalctl` (systemd journal)

**Kelebihan:**

- Filter berdasarkan service (`-u ssh`), priority (`-p err`), waktu (`--since "1 hour ago"`), boot (`-b`)
    
- Output JSON (`-o json`) untuk parsing otomatis
    
- Aman & cepat karena format binary terindeks
    

**Kekurangan:**

- Tidak bisa langsung `grep` / `tail` – harus lewat `journalctl`
    
- Butuh akses khusus (atau `sudo`)


**contoh :**
```
journalctl -u ssh -n 50            # 50 baris terakhir service ssh
journalctl --since "5 min ago" -f  # follow log 5 menit terakhir
journalctl -p err -b               # error selama boot terakhir
```

## 📁 File `/var/log/*`

**Kelebihan:**

- Format teks biasa → langsung `grep`, `awk`, `tail -f`
    
- Tidak tergantung systemd (cocok untuk container, WSL, atau sistem minimalis)
    
- `/var/log/auth.log` khusus menyimpan log otentikasi (SSH, sudo, login)
    

**Kekurangan:**

- Filter gabungan (waktu + priority + unit) repot tanpa tool tambahan
    
- Pencarian lambat pada file besar
    

**Contoh:**

```
sudo tail -f /var/log/auth.log | grep "Failed password"
sudo grep "error" /var/log/syslog | tail -20
```


> Kuasai keduanya. `journalctl` untuk filter terstruktur; file `/var/log` untuk grep manual dan kompatibilitas luas. 

