# ADR-0001 — Fondasi teknis lumbung-data-pipeline

- Status: Diterima
- Tanggal: 2026-10-14

## Konteks

lumbung-data-pipeline mengisi data timbangan, trip, dan harga referensi untuk web dan model AI, serta menjadi penghubung event Lumbung ke Titeny (KONTEKS §3).

## Keputusan

1. Ingest idempoten dengan kunci alami; setiap baris menyimpan sumber dan waktu ingest.
2. Menulis hanya lewat API `lumbung-backend-service` memakai `X-Service-Key` (least privilege); tidak ada koneksi langsung ke DB divisi lain.
3. Event keluar memakai nama terkunci registry Titeny dengan `source: lumbung`: `panen.dicatat`, `harga.diperbarui`, `stok.berubah`, `stok.snapshot_harian`; menambah `event_type` butuh bump versi + 30 hari dual-read.
4. Amplop wajib: `event_id`, `schema_version`, `source`, `occurred_at`/`received_at`, `batch_id`; kirim `POST /api/v1/event/masuk` dengan `X-Idempotency-Key`; retry 3x (5/30/300 dtk) lalu DLQ.
5. Titeny hanya membaca (read-only) ke DB sumber; pipeline tidak memberi akses tulis.
6. Pilihan bahasa implementasi pipeline diputuskan di ADR-0002 sebelum kode pertama.

## Konsekuensi

- Event domain FSD/30 (`lot.disortir`, `trip.dimuat`, dst.) harus dipetakan ke registry; pemetaan dicatat di dokumen kontrak event repo.
- Lembar kerja 3 referensi harga disimpan 6 bulan.
