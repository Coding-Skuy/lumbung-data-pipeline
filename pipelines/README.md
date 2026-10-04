# pipelines

Daftar pipeline ingest Lumbung (STUB awal — implementasi menyusul).

## Rencana Pipeline

- `ingest-timbangan` — baca berkas/serial timbangan gudang, upsert ke tabel `timbangan`.
- `ingest-trip` — baca log trip kebun-gudang, upsert ke tabel `trip`.
- `ingest-pasar-induk` — baca harga pasar induk harian, upsert ke tabel harga.

## Kontrak Umum

- Bahasa bebas (Python/Rust disarankan), satu folder satu pipeline.
- Setiap pipeline wajib punya `README.md` sendiri berisi: sumber, format input, kunci idempotensi, dan tabel tujuan.
- Kredensial hanya via environment (`DATABASE_URL` basis data `lumbung`); jangan commit kredensial.
