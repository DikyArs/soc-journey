
**Windows Event Log punya 3 kategori utama:**

| Log Channel   | Isi                                    | Relevansi SOC                         |
| ------------- | -------------------------------------- | ------------------------------------- |
| `Security`    | Login, logout, privilege, audit policy | Paling penting — semua aktivitas user |
| `System`      | Service start/stop, driver, hardware   | Deteksi service aneh yang dipasang    |
| `Application` | Error aplikasi, crash, update          | Malware yang crash atau berulah       |

**Struktur satu Event Log:**

```
Event ID   : 4625
Level      : Audit Failure
Date/Time  : 2024-04-17 14:23:01
Source     : Microsoft-Windows-Security-Auditing
Computer   : WORKSTATION-01
User       : Administrator

Description:
An account failed to log on.
Account Name: administrator
Failure Reason: Unknown user name or bad password
Source IP: 192.168.1.100
```


| Field           | Arti & Relevansi untuk SOC                                                                                                                                                                                                                                                                                              |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Event ID**    | Kode numerik unik yang menunjukkan jenis kejadian. Misal `4625` = gagal login. SOC pakai Event ID untuk filtering, korelasi, dan deteksi anomali (brute force, privilege escalation, dll).                                                                                                                              |
| **Level**       | Tingkat keparahan/status: `Audit Failure` (gagal), `Audit Success` (berhasil), `Error`, `Warning`, `Information`. Di SOC, Audit Failure sering mengindikasikan percobaan akses tidak sah.                                                                                                                               |
| **Date/Time**   | Waktu kejadian tercatat. Penting untuk _timeline analysis_ dan rekonstruksi serangan (misal: jam berapa brute force terjadi).                                                                                                                                                                                           |
| **Source**      | Komponen atau layanan Windows yang menghasilkan log. `Security-Auditing` adalah sumber paling krusial untuk aktivitas login/privilege. SOC perlu tahu sumber terpercaya vs sumber aneh.                                                                                                                                 |
| **Computer**    | Nama host/komputer tempat event terjadi. Berguna untuk identifikasi aset terdampak, terutama dalam lingkungan dengan banyak endpoint.                                                                                                                                                                                   |
| **User**        | Akun user yang terkait dengan event. Bisa `SYSTEM`, `Administrator`, atau user biasa. SOC perhatikan _impersonation_ atau user tidak dikenal.                                                                                                                                                                           |
| **Description** | Teks rinci menjelaskan kejadian. Sering berisi sub-field penting seperti:  <br>- `Account Name`: akun yang dicoba dipakai.  <br>- `Failure Reason`: penyebab gagal (password salah, user tidak ada, account locked).  <br>- `Source IP`: alamat IP asal percobaan. Kunci untuk deteksi serangan dari luar atau insider. |


## EVENT ID

**Authentication & Account:**

| Event ID | Nama                            | Artinya                     | Kenapa Penting                                          |
| -------- | ------------------------------- | --------------------------- | ------------------------------------------------------- |
| `4624`   | Successful logon                | Ada yang berhasil login     | Normal, tapi curigai kalau jam aneh atau user tak biasa |
| `4625`   | Failed logon                    | Login gagal                 | Brute force kalau banyak dalam waktu singkat            |
| `4634`   | Logoff                          | User logout                 | Tracking sesi                                           |
| `4648`   | Logon with explicit credentials | Pakai credential orang lain | Sering muncul saat lateral movement                     |
| `4672`   | Special privileges assigned     | User dapat privilege tinggi | Privilege escalation                                    |
| `4720`   | User account created            | Akun baru dibuat            | Backdoor account oleh attacker                          |
| `4726`   | User account deleted            | Akun dihapus                | Attacker hapus jejak                                    |

**Process & Execution:**

| Event ID | Nama            | Artinya               | Kenapa Penting                     |
| -------- | --------------- | --------------------- | ---------------------------------- |
| `4688`   | Process created | Ada proses baru jalan | Deteksi malware, command execution |
| `4689`   | Process exited  | Proses selesai        | Tracking durasi proses             |

**System & Service:**

| Event ID | Nama                  | Artinya                       | Kenapa Penting                 |
| -------- | --------------------- | ----------------------------- | ------------------------------ |
| `7034`   | Service crashed       | Service berhenti tidak normal | Malware yang gagal jalan       |
| `7045`   | New service installed | Service baru dipasang         | Persistence mechanism attacker |

**Policy & Audit:**

| Event ID | Nama                   | Artinya                            | Kenapa Penting                                 |
| -------- | ---------------------- | ---------------------------------- | ---------------------------------------------- |
| `4698`   | Scheduled task created | Task baru dibuat di Task Scheduler | Persistence mechanism                          |
| `4719`   | Audit policy changed   | Audit policy diubah                | Attacker matikan logging                       |
| `1102`   | Audit log cleared      | Log dihapus                        | Attacker hapus jejak — ini sangat mencurigakan |


### Skenario Deteksi dari Event ID

**Skenario 1 — Brute Force lalu Berhasil Masuk:**
```
Attacker coba login berkali-kali → akhirnya berhasil → langsung akses file sensitif
```

Event ID yang muncul berurutan: 4625 → 4624 → 4672

**Skenario 2 — Attacker Buat Backdoor:**
```
Attacker sudah di dalam sistem → buat user baru → kasih privilege admin → install service baru untuk persistence
```

Event ID yang muncul: 4720 → 4672 → 7045

**Skenario 3 — Attacker Hapus Jejak:**
```
Attacker selesai operasi → hapus event log → logout
```

Event ID yang muncul: 1102 → 4634



## Sysmon

Sysmon (System Monitor) adalah tool gratis dari Microsoft Sysinternals yang memperkaya log Windows jauh lebih detail dibandingkan Event Log bawaan. Di lingkungan enterprise, hampir semua perusahaan yang serius mengamankan endpoint mereka memasang Sysmon.

### Kenapa Sysmon Penting?

Event log standar Windows hanya memberikan informasi terbatas. Contohnya Event ID 4688 (process creation) hanya mencatat nama proses tanpa detail tambahan. Sementara itu, **Sysmon Event ID 1** mencatat:
- Nama proses
- Hash file (MD5, SHA1, SHA256)
- Command line lengkap
- Parent process (proses pemanggil)

Perbedaannya sangat krusial: Event ID 4688 hanya memberi tahu "ada proses cmd.exe berjalan". Sysmon Event ID 1 bisa menunjukkan "cmd.exe dipanggil oleh winword.exe dengan argumen `powershell -enc SQBFAFgA...`" — ini langsung mengindikasikan aktivitas mencurigakan seperti macro malware.

### Sysmon Event ID yang Wajib Diketahui

| Sysmon ID | Artinya |
|-----------|---------|
| 1 | Process created — detail lengkap parent process, command line, hash |
| 3 | Network connection — proses mana yang melakukan koneksi keluar ke IP/port berapa |
| 11 | File created — file baru dibuat di lokasi tertentu |
| 13 | Registry value set — perubahan nilai registry |
| 22 | DNS query — proses mana yang melakukan query domain tertentu |

### Mengapa Sysmon Event ID 3 (Network Connection) Sangat Berguna untuk Deteksi Malware?

Malware modern hampir selalu butuh komunikasi jaringan — entah untuk mengunduh stage kedua, mengirim data curian (exfiltration), atau menerima perintah dari C2 (Command & Control). Sysmon Event ID 3 mencatat setiap koneksi keluar beserta:
- Proses sumber (nama, PID, path lengkap)
- Alamat IP tujuan dan port
- Protokol (TCP/UDP)

Dengan log ini, analis keamanan bisa langsung menjawab pertanyaan kritis:
- "Apakah powershell.exe yang dipanggil oleh Outlook tiba-tiba konek ke IP asing port 4444?" — indikasi reverse shell.
- "Apakah file bernama svchost.exe yang berjalan dari folder %TEMP% mencoba terhubung ke domain yang baru terdaftar?" — itu jelas anomali.

Tanpa Sysmon ID 3, Anda hanya tahu ada koneksi jaringan dari komputer, tapi tidak tahu proses mana yang menyebabkannya. Dengan Sysmon, korelasi antara proses jahat dan aktivitas jaringannya menjadi sangat mudah. Ini adalah salah satu sumber data utama untuk hunting ancaman (threat hunting) dan deteksi dini di level endpoint.