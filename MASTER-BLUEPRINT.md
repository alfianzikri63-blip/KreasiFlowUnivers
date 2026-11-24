# KreasiFlow Universal AI — Master Blueprint V1.0

Status: rancangan teknis, bukan APK, bukan integrasi cloud yang sudah teruji. Target: Android, penggunaan dari HP, proyek terpisah dari KreasiFlow lama. Prinsip: offline-first, cloud-powered; satu APK, banyak adapter yang terverifikasi.

## 1. Scope dan keputusan V1
- Dashboard, Skill Library, Project Manager, Studio, Render Center, Settings. UI dark, Bahasa Indonesia, mobile portrait, aksesibilitas dan status yang jelas.
- Mode manual terlebih dahulu: buat project, salin prompt/kirim ke Codex melalui browser, simpan hasil, buka GitHub Actions, catat link run, impor MP4 melalui pemilih file. Tidak perlu API AI berbayar untuk alur manual; kuota ChatGPT/Codex dan GitHub berlaku terpisah.
- Mode otomatis (fase berikutnya): layanan backend untuk provider AI, adapter build/render dan autentikasi, status pekerjaan dan unduhan. Tidak mengasumsikan bahwa akun Plus menyediakan API atau bahwa Codex Cloud memiliki endpoint publik yang bisa dipanggil dari APK.
- Jangan menyatakan skill berhasil dirender sebelum ada artifact MP4 yang valid. Jangan membuat semua modul menjadi fitur siap render hanya karena paketnya dapat diimpor.
- Repo aplikasi baru, usulan `KreasiFlowUniversal` (private). Repo `alfianzikri63-blip/BedahKlasik` tetap menjadi renderer pilot; tidak mengubah workflow yang sedang diuji.

## 2. Modul sumber (berdasarkan paket yang diberikan)
| ID | Paket | Format asli | Engine / batas kesiapan |
|---|---|---|---|
| bedah-klasik | Bedah_Klasik_Codex_V1.zip | 9:16, ±40 dtk, 95–130 kata, 10 beat, HUD, karaoke per kata | Python + Three.js + HyperFrames 0.8.214; ada new_episode.py, tts.py, build.py, check_video.py. Workflow GitHub sudah ada, hasil render belum dikonfirmasi. |
| science3d-petir | science3d-render-kit-petir.zip | 9:16, 37 dtk, 9 scene, tanpa SFX pada contoh | WebGL2/Playwright/FFmpeg; render.js memakai batas scene tetap, parts/sN.mp4; perlu input VO, penggabungan scene dan adapter. Tidak ada SKILL.md; impor sebagai render kit, bukan skill universal. |
| psych-parable | psych-parable-upload.zip | 9:16, 60–80 dtk, 9 beat, 170–190 kata untuk 70 dtk; mode pendek tanpa VO 12–20 dtk | SVG silhouette rig + HyperFrames; template, contoh build, audio dan font ada. Perlu adapter produksi episode dan sinkronisasi cue frasa. |
| bedah-medis-mitos-3d | bedah-medis-mitos-3d.zip | 9:16, ±60 dtk, 115–125 kata, anatomi dan label AMAN/WASPADA | SKILL.md dan CARA-PASANG.txt saja. Mode paket prompt terlebih dahulu; engine Remotion/Canvas+FFmpeg adalah opsi sumber, belum dibangun. |

Pertahankan desain per modul, bukan paksa semua ke template yang sama. Jangan tampilkan hasil contoh sebagai hasil produksi baru. Periksa lisensi sumber, font, audio, model dan distribusi sebelum publikasi atau memasukkan aset ke repo publik. Untuk medis: sumber kredibel, tidak diagnosis atau rekomendasi dosis; red flags dan tindakan yang tepat sesuai topik, disclaimer edukasi. Untuk psikologi: sumber ilmiah nyata, jangan mengarang persentase, bukan diagnosis. Untuk sains: satuan dan asumsi diverifikasi ulang; jangan mengulang klaim paket secara otomatis.

## 3. UI / navigasi
- Home: jumlah skill, proyek, hasil; tombol New Project; status terakhir.
- Skills: daftar, detail deskripsi, versi, sumber, kebutuhan runtime, tahap kesiapan, import dan pemeriksaan; status Draft/Needs adapter/Needs test/Verified/Blocked. Label Verified hanya setelah pengujian tercatat.
- Projects: buat, cari, duplikat, ekspor/restore, hapus dengan konfirmasi; status berdasarkan tahap aktual.
- Studio: stepper dinamis mengikuti manifest, editor naskah, sumber fakta, scene/beat, preview suara/visual dan approval.
- Render: tautan workflow, run ID, status manual atau server verified (beri label sumber status), logs dan artifact; tombol ulang hanya bila tidak ada job aktif untuk versi yang sama.
- Settings: brand/handle milik user, bahasa, provider, koneksi GitHub, direktori ekspor, backup. API key bukan bagian project/backup.
- Export: MP4, naskah, caption, sumber, credit, cover; hanya file yang benar-benar tersedia. Android Storage Access Framework untuk impor/ekspor.

## 4. Arsitektur teknis
Pilot APK: Java + Android WebView untuk UI lokal yang dibundel di assets (HTML/CSS/JS), navigasi dan file picker native, SQLite/app-private storage. Tidak menjalankan HTML/JS impor dalam WebView yang memiliki native bridge. Link eksternal dibuka di browser. Preview V1 menggunakan gambar, audio dan MP4, bukan menjalankan script skill di HP. Build Gradle via GitHub Actions pada repo aplikasi baru. SDK dan versi Gradle/JDK diselaraskan saat implementasi.

Lapisan: UI -> Project/Skill services -> storage lokal -> adapter registry -> manual cloud handoff atau backend job API. Backend opsional setelah pilot: autentikasi, DB, secret store, job queue, worker terisolasi, object storage. GitHub Actions dipakai melalui workflow yang telah direview. Jangan menaruh PAT, key provider atau credential di APK, localStorage, source repo, log, maupun backup. Jangan memberi job yang menjalankan kode impor akses write repo/secrets. Gunakan izin minimum, repo private bila lisensi atau data belum jelas, dan hanya kirim data yang disetujui pengguna.

## 5. Kontrak modul (usulan manifest.json, dibuat oleh registry/importer, bukan format asli Claude)
```json
{
  "schemaVersion": 1,
  "id": "bedah-klasik",
  "version": "1.0.0",
  "title": "Bedah Klasik",
  "sourceType": "agent-skill",
  "entry": "SKILL.md",
  "adapterId": "bedah-klasik-hyperframes-v1",
  "readiness": "needs-test",
  "defaults": {"ratio": "9:16", "durationSeconds": 40, "language": "id"},
  "capabilities": ["script", "research", "tts", "preview", "mp4"],
  "stages": [
    {"id":"topic", "type":"editor", "approval":true},
    {"id":"facts", "type":"research", "approval":true},
    {"id":"script", "type":"editor", "approval":true},
    {"id":"model", "type":"cloud-build", "approval":true},
    {"id":"voice", "type":"audio", "approval":true},
    {"id":"timeline", "type":"editor", "approval":false},
    {"id":"preview", "type":"review", "approval":true},
    {"id":"render", "type":"cloud-render", "approval":false},
    {"id":"export", "type":"export", "approval":true}
  ]
}
```
Manifest deklaratif, bukan script executable. Validasi schema, tipe stage dan adapter ID dalam allowlist. Modul baru dapat diimpor tanpa rebuild APK untuk fungsi yang sudah didukung; kebutuhan engine/adapter yang belum ada memerlukan pengembangan dan deployment baru. Import GitHub pada V1 melalui download ZIP oleh user atau tautan sumber; bukan clone otomatis dengan token di HP.

## 6. Penyimpanan data lokal (logical schema)
- skills: id, version, title, description, source, contentHash, manifestPath, adapterId, readiness, createdAt.
- projects: id, skillId, skillVersion, title, topic, settingsJson, activeStage, status, createdAt, updatedAt.
- stage_runs: id, projectId, stageId, revision, status, inputHash, approvedAt, outputRefsJson, errorSummary.
- assets: id, projectId, kind, relativePath, mime, sizeBytes, sha256, sourceUrl, credit.
- render_jobs: id, projectId, revision, provider, repo, workflow, runId, runUrl, status, statusSource, artifactRef, errorSummary, updatedAt.
- preferences: brand, language, lastProvider, non-secret settings. Backup format versioned, tanpa key/token.
- Project version dikunci ke skill version; update skill tidak mengubah proyek lama tanpa migrasi. Edit tahap sebelumnya menandai hasil downstream stale; render lama tetap sebagai riwayat, bukan hasil versi baru.

## 7. Skill importer dan keamanan
1. Android file picker -> staging app-private. Batasi ukuran (usulan V1: 25 MB ZIP, 100 MB expanded, 1.000 entry; beri pesan jika melebihi).
2. Validasi path ZIP canonical, larang absolute path, traversal, symlink dan nested archive execution. Jangan mengeksekusi installer, shell, Python atau JS pada proses import.
3. Cari SKILL.md dan baca frontmatter name/description dengan parser aman tanpa custom tags; jika tidak ada, buat entri render kit manual. Tampilkan ringkasan, inventaris file, checksum, sumber dan izin yang dibutuhkan.
4. Cek duplikat dan konflik versi. Simpan skill dalam direktori terisolasi setelah konfirmasi; status imported/needs review. Scan statis bersifat indikatif, tidak menjamin keamanan.
5. Cocokkan adapter berdasarkan registry terpercaya, bukan perintah dari ZIP. Review kode, dependensi, lisensi dan izin sebelum run. Jangan mengirim secret ke AI atau mengikuti instruksi luar yang meminta credential.
6. Preview media hanya dengan parser yang aman dan tipe file allowlist. Import markdown sebagai teks/sanitized display. Update tidak menghapus versi proyek yang masih dipakai.

## 8. Alur produksi per modul
- Bedah Klasik: pilih topik -> fakta resmi/media/dimodelkan -> approve naskah -> model.js/state.js -> pilih take VO -> beats berbasis kata -> snapshot/QC -> render MP4 -> review & posting. Simpan episode di `episodes/<slug>/` dengan struktur SKILL.md. Contoh kloset dan pentil-ban sudah ada. Pilot workflow memilih mode `kloset`, `pentil-ban`, atau `custom` + slug; custom memerlukan file episode di repo, tidak hanya prompt.
- Science 3D: audit fakta dan satuan -> naskah/storyboard -> adaptasi scenes.js/hud.js/engine.js + timing -> render scene -> gabung dengan VO -> QC -> ekspor. V1 hanya template episode petir; episode bebas membutuhkan generalisasi timeline. Hindari flash berlebihan, uji visual.
- Psych Parable: topik dan 9 beat -> bukti ilmiah -> storyboard -> silhouette_rig -> VO dan cue frasa -> animasi/kartu -> motion review/QC -> render -> caption. Mode 12–20 detik tanpa VO dibuat terpisah. Pertahankan karakter original dan handle user.
- Bedah Medis: topik dan mitos -> fakta -> naskah -> peta caption 1–2 kata -> paket prompt visual/CapCut -> review medis -> ekspor materi. Render otomatis ditandai belum tersedia sampai adapter anatomi dibuat dan lulus tes.

## 9. Cloud job lifecycle / penghubung (usulan)
- Mode manual V1: simpan project -> copy prompt -> buka Codex -> user review perubahan repo -> buka Actions -> pilih workflow, mode/slug -> salin link run ke project -> unduh artifact ZIP dari halaman run -> import MP4 ke project. Status manual ditandai manual, bukan polling real-time.
- Mode otomatis mendatang: backend mengautentikasi user, membuat job idempotent per project+revision+adapter, mengirim workflow_dispatch dengan input allowlist (jangan membangun shell dari teks prompt), mengaitkan run dengan job lewat correlation ID, mengambil status via polling/webhook tervalidasi, menerbitkan URL unduhan singkat setelah verifikasi kepemilikan. API endpoints usulan: POST /jobs, GET /jobs/{id}, POST /jobs/{id}/cancel, GET /jobs/{id}/artifacts.
- State: draft -> awaiting_approval -> queued -> running -> succeeded/failed/cancelled -> exported. Stale/expired artifact adalah flag terpisah. Retry hanya bila error dapat diulang; beri batas biaya/timeout; tidak auto retry tanpa batas.
- Edge TTS memakai jaringan pihak ketiga; minta persetujuan kirim naskah. Alternatif audio rekaman user. API AI/TTS berbayar bersifat opsional dan butuh izin eksplisit. Video bukan sukses hanya karena workflow hijau; ffprobe, durasi, resolusi, audio dan review frame diperlukan.

## 10. Struktur repo aplikasi (rencana)
```
KreasiFlowUniversal/
  app/src/main/java/.../       # native Android, file picker, storage
  app/src/main/assets/ui/      # UI terpercaya lokal
  app/src/main/assets/registry/# manifest dan adapter metadata terpercaya
  docs/                       # blueprint, panduan
  tests/                      # import, project, state, keamanan
  .github/workflows/android.yml
  build.gradle / settings.gradle / gradle wrapper
```
ZIP pihak ketiga tidak dibundel publik sebelum lisensi diperiksa. Build app tidak menjalankan skill yang diimpor. Renderer BedahKlasik tetap repo terpisah; engine lain dapat ditambahkan sesuai adapter.

## 11. Milestone dan kriteria lulus
1. V1.0 Core APK: dapat dibuild, install HP, navigasi, buat/edit/hapus project, autosave setelah restart, import 4 paket, deteksi SKILL.md vs render kit, status adapter, backup/restore, manual handoff. Uji ZIP traversal dan duplikat versi; tidak ada secret/log sensitif. 
2. V1.1 Bedah Klasik: tes demo kloset dan pentil-ban, MP4 terverifikasi, lalu satu episode custom dengan review naskah/VO/visual, download dan import ke APK. Perbaiki workflow bila gagal.
3. V1.2 Psych Parable: adapter template dan cue, episode demo original serta QC gerak/audio.
4. V1.3 Science 3D: demo petir lengkap dengan audio yang disetujui, kemudian satu topik baru memakai timeline yang tidak hardcoded.
5. V1.4 Bedah Medis: paket prompt dulu, engine anatomi terverifikasi kemudian; QC sumber medis, caption dan red flags sesuai topik.
6. V2: backend otomatis dengan auth/secret management, provider API, job dispatch, status dan unduhan. Uji permission lintas user, timeout, retry, kuota dan biaya sebelum rilis.

## 12. Next action
Mulai source code Core APK V1.0 di proyek baru; jangan menunggu tes renderer untuk pekerjaan UI/storage/import. Jangan mengubah atau menjalankan ulang workflow BedahKlasik sampai status run saat ini diketahui. Belum ada repo aplikasi baru yang dibuat oleh blueprint ini.

## Basis paket yang diperiksa
- Bedah_Klasik_Codex_V1.zip: bedah-klasik/SKILL.md, daftar script dan examples.
- science3d-render-kit-petir.zip: PAKET_PRODUKSI.md, render.js, daftar file.
- psych-parable-upload.zip: psych-parable/SKILL.md, daftar asset/template.
- bedah-medis-mitos-3d.zip: bedah-medis-mitos-3d/SKILL.md, daftar file.
- Alur render yang telah disepakati di repo BedahKlasik. Seluruh desain sistem, manifest, schema dan milestone di atas adalah usulan pengembangan, bukan fitur yang sudah berfungsi.
