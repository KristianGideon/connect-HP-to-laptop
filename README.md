📱 HP Screen Mirroring ke Laptop di CachyOS / Arch Linux
Panduan simpel menampilkan layar HP Android ke laptop menggunakan scrcpy – baik lewat kabel USB maupun Wi‑Fi. Dibuat khusus untuk CachyOS (Arch Linux) dan sudah diuji dengan Vivo V2116 (Android 13).

🔧 Instalasi Awal (Cukup Sekali)
Buka terminal dan jalankan:

sudo pacman -Syu

sudo pacman -S scrcpy android-tools android-udev

sudo usermod -aG adbusers $USER

Kemudian logout dan login ulang agar grup adbusers aktif.

⚡ A. Koneksi Kabel (USB) – Paling Cepat
Aktifkan USB Debugging di HP:

Settings > Additional settings > Developer options > USB debugging (ON).
Cara membuka Developer options: tap Build number 8x di About phone.

Hubungkan HP ke laptop dengan kabel data.

Izinkan USB debugging saat muncul popup di HP (centang Always allow).

Jalankan perintah ajaib ini di terminal:

scrcpy --video-bit-rate=16M --no-audio --turn-screen-off --render-driver=opengl --screen-off-timeout=600 --stay-awake

Selesai. Layar HP langsung muncul, kualitas tajam, dan minim lag. Cocok untuk gaming.

📶 B. Koneksi Wireless (Wi‑Fi) – Tanpa Kabel

Gunakan jika ingin bebas kabel. Pastikan laptop dan HP terhubung ke jaringan Wi‑Fi yang sama.

1. Aktifkan Wireless Debugging di HP

Buka Settings > Additional settings > Developer options

Aktifkan Wireless debugging

2. Skrip Koneksi Manual (Direkomendasikan)

Simpan file berikut dengan nama connect-wifi.sh:

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
