# lumbung-data-pipeline

Pipeline data Lumbung: menarik data timbangan, trip, dan harga pasar induk ke basis data `lumbung`.

## Sumber Data

- Timbangan gudang (USB/serial) — berat terima per lot.
- Trip kebun ke gudang — asal, tujuan, durasi, kendaraan.
- Harga pasar induk — harga per komoditas per pasar.

## Tujuan

Basis data `lumbung` (milik `lumbung-backend-service`) menjadi satu-satunya sumber kebenaran untuk ops, web, dan model AI.

## Struktur

```text
lumbung-data-pipeline/
├── pipelines/README.md   # daftar pipeline ingest
└── docs/schedule.md      # jadwal jalan pipeline
```

## Prinsip

- Idempoten: ingest ulang dengan kunci alami yang sama tidak menggandakan baris.
- Auditabel: setiap baris menyimpan sumber dan waktu ingest.
- Terpisah: pipeline tidak menulis ke basis data divisi lain.

## Keterkaitan

- Skema DB: [lumbung-backend-service](https://github.com/Coding-Skuy/lumbung-backend-service) (`migrations/`)
- Konsumen: [lumbung-ai-models](https://github.com/Coding-Skuy/lumbung-ai-models), [lumbung-web](https://github.com/Coding-Skuy/lumbung-web)
