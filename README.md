📱 HP Screen Mirroring ke Laptop di CachyOS / Arch Linux
Panduan simpel menampilkan layar HP Android ke laptop menggunakan scrcpy – baik lewat kabel USB maupun Wi‑Fi. Dibuat khusus untuk CachyOS (Arch Linux) dan sudah diuji dengan Vivo V2116 (Android 13).

🔧 Instalasi Awal (Cukup Sekali)
Buka terminal dan jalankan:
```
sudo pacman -Syu
sudo pacman -S scrcpy android-tools android-udev
sudo usermod -aG adbusers $USER
```
Kemudian logout dan login ulang agar grup adbusers aktif.

⚡ A. Koneksi Kabel (USB) – Paling Cepat
Aktifkan USB Debugging di HP:

Settings > Additional settings > Developer options > USB debugging (ON).
Cara membuka Developer options: tap Build number 8x di About phone.

Hubungkan HP ke laptop dengan kabel data.

Izinkan USB debugging saat muncul popup di HP (centang Always allow).

Jalankan perintah ajaib ini di terminal:
```
scrcpy --video-bit-rate=16M --no-audio --turn-screen-off --render-driver=opengl --screen-off-timeout=600 --stay-awake
```
Selesai. Layar HP langsung muncul, kualitas tajam, dan minim lag. Cocok untuk gaming.

📶 B. Koneksi Wireless (Wi‑Fi) – Tanpa Kabel

Gunakan jika ingin bebas kabel. Pastikan laptop dan HP terhubung ke jaringan Wi‑Fi yang sama.

1. Aktifkan Wireless Debugging di HP

Buka Settings > Additional settings > Developer options

Aktifkan Wireless debugging

2. Skrip Koneksi Manual (Direkomendasikan)

Simpan file berikut dengan nama connect-wifi.sh:
```
#!/bin/bash
# Wireless ADB – input IP, port, dan pairing code manual
# Untuk HP yang tidak terdeteksi otomatis oleh mDNS

echo "========================================="
echo "   📱 Koneksi Wireless ADB Manual"
echo "========================================="
echo ""
echo "📶 Pastikan HP & laptop di Wi‑Fi yang sama."
echo "🔧 Di HP: Developer Options > Wireless debugging = ON"
echo ""

# Langkah 1: Pairing
echo -e "\e[33m🔢 Langkah 1: Pairing\e[0m"
echo "   Di HP, masuk ke:"
echo "   Wireless debugging > Pair device with pairing code"
echo "   Akan muncul alamat IP, port, dan kode 6 digit."
echo ""
read -p "   Masukkan IP:port pairing (contoh 192.168.1.10:41235): " PAIR_ADDR
read -p "   Masukkan kode pairing 6 digit (contoh 123456): " PAIR_CODE

echo -e "\n\e[36m🔄 Melakukan pairing...\e[0m"
adb pair $PAIR_ADDR $PAIR_CODE
if [ $? -ne 0 ]; then
    echo -e "\e[31m❌ Gagal pairing. Periksa kembali IP, port, dan kode.\e[0m"
    echo "   Coba ulangi langkah 1 di HP untuk mendapatkan kode baru."
    exit 1
fi

# Langkah 2: Connect
echo ""
echo -e "\e[33m🌐 Langkah 2: Menghubungkan\e[0m"
echo "   Sekarang lihat kembali menu Wireless debugging di HP."
echo "   Di bagian atas akan ada 'IP address & Port' (contoh: 192.168.1.10:38417)."
echo "   Biasanya port-nya BERBEDA dari port pairing tadi."
read -p "   Masukkan IP:port untuk connect: " CONNECT_ADDR

echo -e "\n\e[36m🌐 Menghubungkan ke $CONNECT_ADDR...\e[0m"
adb connect $CONNECT_ADDR
sleep 2

# Verifikasi
if adb devices | grep -q "$CONNECT_ADDR.*device"; then
    echo -e "\e[32m✅ Berhasil terhubung!\e[0m"
    echo ""
    echo "   ✨ Jalankan perintah ini untuk mirroring:"
    echo -e "\e[1m   scrcpy --video-bit-rate=16M --no-audio --turn-screen-off --render-driver=opengl --screen-off-timeout=600 --stay-awake\e[0m"
else
    echo -e "\e[31m❌ Koneksi gagal.\e[0m"
    echo "   Pastikan IP:port untuk connect sudah benar."
    echo "   Jika masih gagal, coba restart Wireless debugging di HP."
fi
```
Cara menggunakan:
```
chmod +x connect-wifi.sh
./connect-wifi.sh
```
Saat diminta, lihat HP: buka Wireless debugging > Pair device with pairing code → masukkan IP:port dan kode 6 digit.

Setelah pairing berhasil, lihat lagi menu utama Wireless debugging → masukkan IP:port yang tertera di sana (port biasanya berbeda).

Jika sukses, salin dan jalankan perintah scrcpy yang ditampilkan.

3. Skrip Otomatis (dengan Avahi/mDNS) – Alternatif
Hanya sebagai cadangan jika jaringan mendukung.
Fitur ini mencari HP secara otomatis lewat mDNS. Namun di beberapa jaringan, deteksi bisa gagal. Jika gagal, gunakan skrip manual di atas.

```
#!/bin/bash
# Wireless ADB - auto detect via mDNS (Android 11+ Wireless Debugging)
echo "🔍 Mencari perangkat Android dengan Wireless Debugging aktif..."
DEVICES=$(avahi-browse -rt _adb-tls-connect._tcp 2>/dev/null | awk '
/^=.*IPv4/ { iface=$2; }
/hostname =/ { gsub(/\[|\]/,""); host=$2; }
/address =/ { gsub(/\[|\]/,""); addr=$2; }
/port =/ { gsub(/\[|\]/,""); port=$2; }
/^txt =/ { printf "%s|%s|%s\n", host, addr, port; }
')

if [ -z "$DEVICES" ]; then
    echo "❌ Tidak ditemukan. Gunakan skrip manual."
    exit 1
fi

HOST=$(echo "$DEVICES" | head -1 | cut -d'|' -f1)
ADDR=$(echo "$DEVICES" | head -1 | cut -d'|' -f2)
PORT=$(echo "$DEVICES" | head -1 | cut -d'|' -f3)

echo "✅ Ditemukan: $HOST ($ADDR:$PORT)"
read -p "   Masukkan kode pairing 6 digit (dari HP): " PAIR_CODE

adb pair $ADDR:$PORT $PAIR_CODE
if [ $? -ne 0 ]; then
    echo "❌ Gagal pairing."
    exit 1
fi

adb connect $ADDR:$PORT
echo "Jalankan perintah scrcpy yang tersedia."
```
Instal paket pendukung:
```
sudo pacman -S avahi
```
dan pastikan avahi-daemon berjalan ```(sudo systemctl enable --now avahi-daemon).```

🎮 Perintah Gaming Optimal
Setelah terkoneksi (USB atau Wi‑Fi), gunakan perintah ini untuk performa terbaik dan hemat baterai:

```
scrcpy --video-bit-rate=16M --no-audio --turn-screen-off --render-driver=opengl --screen-off-timeout=600 --stay-awake
```
Penjelasan opsi:

--video-bit-rate=16M : Bitrate video tinggi → gambar lebih tajam (default 8M).

--no-audio : Tidak mengalirkan suara HP → fokus ke video.

--turn-screen-off : Layar HP mati → hemat baterai.

--render-driver=opengl : Performa rendering terbaik di Linux.

--screen-off-timeout=600 : Jaga layar mati hingga 10 menit.

--stay-awake : Mencegah HP masuk deep sleep.

Jika laptop kurang kuat, tambahkan --max-size=1920 untuk menurunkan resolusi.

Fitur ini sudah ada di scrcpy versi 1.16 ke atas.

## ⚠️ Troubleshooting Ringkas

| Masalah | Solusi |
|---|---|
| `adb devices` **kosong (USB)** | Cek kabel (harus data), ganti port USB, pastikan mode MTP/Transfer File. Jalankan `sudo udevadm trigger`. |
| Status `unauthorized` | Lihat layar HP, centang "Always allow" lalu izinkan. Jika tidak muncul, cabut kabel, revoke izin di Developer options. |
| `adb push` **gagal / Server connection failed** | Restart ADB: `adb kill-server && adb start-server`. Coba tanpa opsi `--turn-screen-off` dulu. |
| **HP tidak ditemukan di wireless** | Pastikan Wi-Fi sama, Wireless debugging ON. Gunakan skrip manual. |
| **Kode pairing salah** | Kode hanya berlaku sekali. Dapatkan kode baru dari HP. |
| **Gagal connect setelah pairing** | Pastikan port untuk connect benar (biasanya berbeda dari port pairing). Lihat di menu utama Wireless debugging. |
| **Koneksi putus-putus** | Gunakan Wi-Fi 5GHz, dekatkan perangkat, kurangi bitrate ke 8M, atau ganti codec ke `--video-codec=h264`. |

## 📝 Catatan

- **Koneksi wireless** tidak otomatis tersambung setelah restart HP. Cukup jalankan ulang skrip
  manual dan hanya lakukan langkah *connect* (karena pairing sudah permanen kecuali di-revoke).

- Untuk kenyamanan, bisa membuat alias di `.bashrc`:
```bash
  alias mirror='scrcpy --video-bit-rate=16M --no-audio --turn-screen-off --render-driver=opengl --screen-off-timeout=600 --stay-awake'
```

- Jika ingin mengembalikan koneksi USB setelah wireless, tinggal colok kabel dan jalankan `scrcpy` seperti biasa.

---

🎉 Selamat menikmati layar HP di laptop! Kalau ada kendala, silakan buka issue di repo GitHub ini.
