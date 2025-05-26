# 🌡️ Rust-Based Modbus Sensor Monitoring System

Sistem pemantauan suhu dan kelembaban berbasis **sensor SHT20 Modbus RTU** dengan pengolahan data menggunakan **Rust**, pengiriman melalui **TCP**, dan penyimpanan di **InfluxDB**.

## 🔧 Fitur Utama

- Membaca data dari sensor **SHT20** melalui protokol **Modbus RTU**
- Mengirim data berupa **JSON** ke server melalui **TCP Socket**
- Parsing dan simpan data ke **InfluxDB 2.0** menggunakan **Line Protocol**
- Data lengkap: `timestamp`, `sensor_id`, `location`, `process_stage`, `temperature`, `humidity`

## 📁 Struktur Proyek

.
├── clientmodbus.rs # Program Rust client untuk baca sensor dan kirim via TCP
├── tcpserver.rs # Program Rust server untuk terima data dan simpan ke InfluxDB
├── Cargo.toml # Konfigurasi dependencies
├── README.md # Deskripsi proyek

shell
Copy
Edit

## 🖥️ Arsitektur Sistem

[SHT20 Sensor] --(Modbus RTU)--> [Rust Client] --(TCP)--> [Rust TCP Server] --(HTTP)--> [InfluxDB]

markdown
Copy
Edit

## 📦 Dependencies

- `tokio`
- `tokio-modbus`
- `tokio-serial`
- `serde`, `serde_json`
- `reqwest`
- `chrono`

## 🚀 Cara Menjalankan

### 1. Jalankan InfluxDB
Pastikan InfluxDB 2.x berjalan di `localhost:8086` dan sudah membuat:
- Bucket: `monitoring`
- Org: `instrumentasi`
- Token: disesuaikan di dalam kode `tcpserver.rs`

### 2. Jalankan TCP Server

```bash
cargo run --bin tcpserver
3. Jalankan Modbus Client
bash
Copy
Edit
cargo run --bin clientmodbus
Pastikan sensor SHT20 terhubung ke /dev/ttyUSB0.

📊 Contoh Data Masuk ke InfluxDB
yaml
Copy
Edit
measurement: monitoring
tags:
  sensor_id: SHT20-PascaPanen-001
  location: Gudang Fermentasi 1
  process_stage: Fermentasi
fields:
  temperature: 26.7
  humidity: 54.9
timestamp:
  2025-05-26T03:13:51Z
🔍 Query Sample di InfluxDB UI
flux
Copy
Edit
from(bucket: "monitoring")
  |> range(start: -1h)
  |> filter(fn: (r) => r["_measurement"] == "monitoring")
  |> filter(fn: (r) => r["_field"] == "temperature" or r["_field"] == "humidity")
  |> filter(fn: (r) => r["sensor_id"] == "SHT20-PascaPanen-001")
  |> aggregateWindow(every: 1m, fn: mean)
🧠 Catatan
Data dikirim setiap 5 detik (dapat diubah di clientmodbus.rs)

Format waktu menggunakan RFC3339 dan dikonversi ke UNIX epoch

JSON dikirim baris per baris dengan \n agar bisa dibaca oleh TCP server
