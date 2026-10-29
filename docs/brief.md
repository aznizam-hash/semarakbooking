# Brief: Sistem Tempahan Bilik SMK Mengkarak (Semarak Booking)

**Versi:** 0.3 (dikemas kini mengikut review)
**Tarikh:** 29/10/2026
**Pemilik:** Guru Sains Komputer

---

## 1. Tujuan

Menggantikan tempahan bilik melalui Telegram dengan sistem yang lebih teratur. Guru boleh tempah bilik, dan melihat slot kosong serta siapa yang tempah dalam kalendar mingguan dan bulanan. Sistem ini untuk kegunaan dalaman sekolah sahaja.

## 2. Masalah yang Diselesaikan

- Sukar tahu bilik mana yang kosong.
- Tempahan bertindih.
- Format masa dan tarikh tidak konsisten dalam Telegram.
- Ada tempahan tanpa masa tamat.

## 3. Skop Versi Pertama

**Dalam skop**

- Google Form sebagai satu-satunya pintu masuk tempahan.
- Dua bilik: Bilik Seminar dan Bilik Teori PPKI. Bilik lain ditambah kemudian.
- Semakan bertindih dan masa tidak sah oleh Apps Script.
- Emel kepada guru untuk setiap keputusan (Disahkan, Bertindih, Masa tidak sah).
- Emel notifikasi kepada dua admin bila ada tempahan baharu atau bertindih.
- Kalendar mingguan dan bulanan dalam fail berasingan, papar slot kosong dan nama guru yang tempah.

**Di luar skop (versi pertama)**

- Log masuk, akaun, atau kata laluan.
- Aplikasi mudah alih atau web app.
- Telegram, WhatsApp, atau bot. Telegram kekal digunakan sementara, tetapi bukan untuk tempahan baharu.
- Tempahan berbilang tarikh dalam satu borang. Guru hantar berasingan untuk setiap tarikh.
- Tempahan berulang setiap minggu.
- Kelulusan pentadbir. Slot kosong terus disahkan, siapa cepat dia dapat.
- Fungsi batal tempahan oleh guru.
- Semakan semula automatik bila admin edit atau padam baris.
- Peringatan esok melalui trigger harian. Ditangguhkan ke versi kedua.
- Medan sasaran, penceramah, atau kategori program. Cukup dimasukkan dalam medan Tujuan.
- Tempahan untuk pentadbiran (contoh: penempatan sementara pelajar). Ini bukan tempahan bilik.

## 4. Pengguna dan Peranan

| Peranan | Bilangan | Boleh buat | Tidak boleh buat |
|---|---|---|---|
| Guru | Semua guru | Isi borang tempahan, lihat Fail B (Kalendar), terima emel | Edit atau padam tempahan, buka Fail A, tukar peraturan |
| Admin | Dua orang | Semua yang guru buat, terima notifikasi, padam atau betulkan baris, tambah bilik, tukar had masa dalam tab `Tetapan`, jalankan `pasangTrigger` | Tiada had dari segi akses data |

Guru tidak perlu akaun. Peranan dikawal melalui akses kepada fail, bukan dalam aplikasi.

## 5. Peraturan Tempahan

- Siapa cepat dapat. Slot kosong terus disahkan, tiada kelulusan pentadbir.
- Tempahan pada hari yang sama dibenarkan selagi slot kosong.
- Tarikh yang sudah lepas ditolak (status `Masa tidak sah`).
- Sabtu dan Ahad dibenarkan.
- Masa dipilih dari dropdown slot 30 minit:
  - Mula: 07:30 hingga 16:30
  - Tamat: 08:00 hingga 17:00
- Tempoh minimum sesi 30 minit. Masa tamat mesti lebih lewat daripada masa mula.
- Tiada tempahan yang tamat selepas **17:00** (*andaian, perlu disahkan*).
- Tiada had tempoh maksimum sesi (*andaian*).
- Hari (Isnin hingga Ahad) dikira automatik daripada tarikh, tidak ditanya dalam borang.

**Definisi bertindih**

Dua tempahan bertindih jika bilik sama, tarikh sama, dan:

```
mulaBaru < tamatLama  DAN  tamatBaru > mulaLama
```

Tempahan yang bersentuhan di sempadan **tidak** bertindih. Contoh: 10:00–11:00 dan 11:00–12:00 kedua-duanya boleh disahkan.

Semakan hanya dibuat terhadap tempahan berstatus `Disahkan`. Tempahan `Bertindih` atau `Masa tidak sah` tidak menghalang tempahan lain.

## 6. Aliran Pengguna

1. Guru buka pautan Google Form.
2. Guru isi: nama, emel, bilik, tarikh, masa mula, masa tamat, dan tujuan. Semua medan wajib.
3. Apps Script semak masa dan bertindih secara automatik.
4. Jika masa tidak sah (tamat ≤ mula, tamat selepas had, atau tarikh lepas): status `Masa tidak sah`. Guru dapat emel dengan sebab.
5. Jika kosong: status `Disahkan`. Guru dapat emel pengesahan, dua admin dapat notifikasi.
6. Jika bertindih: status `Bertindih`. Guru dapat emel memaklumkan slot telah ditempah, dua admin dapat notifikasi.
7. Guru lihat slot kosong dalam Fail B (Kalendar).
8. Jika ada kesilapan atau nak batal, guru emel admin. Admin betulkan dalam Fail A.

## 7. Struktur Fail

Dua fail Google Sheets:

**Fail A: Tempahan (admin sahaja)**
- Form dihubungkan ke fail ini. Apps Script tinggal di sini.
- Tab `Tempahan`: data mentah dari Form.
- Tab `Tetapan`: nilai yang admin boleh ubah tanpa sentuh code.
- Dikongsi kepada dua admin sahaja sebagai Editor.

**Fail B: Kalendar (guru lihat sahaja)**
- Tab `Mingguan` dan `Bulanan`. Tiada data asal, hanya paparan.
- Tarik data dari Fail A menggunakan `QUERY(IMPORTRANGE(...))`.
- Hanya lajur Nama, Bilik, Tarikh, Mula, Tamat, Tujuan diimport, dan hanya baris `Disahkan`.
- Lajur Emel **tidak diimport langsung**. Sorok lajur tidak cukup, kerana guru boleh muat turun fail dan nampak lajur tersorok.
- Dikongsi kepada semua guru sebagai Viewer.

Fail A mesti dimiliki akaun rasmi sekolah. Kalau akaun pemilik keluar sekolah, Kalendar akan rosak.

**Tetapan fail (kedua-dua fail dan Apps Script)**
- Locale: Malaysia
- Timezone: `Asia/Kuala_Lumpur`

**Tetapan Google Form**
- Matikan "Allow response editing". Tempahan yang diedit selepas hantar tidak akan disemak semula.
- Semua soalan wajib (required).
- Medan emel ditaip oleh guru (bukan "Collect email addresses"), supaya admin boleh masukkan tempahan bagi pihak guru semasa migrasi.

## 8. Struktur Data

**Tab `Tempahan`** (Fail A)

| Lajur | Medan | Keterangan |
|---|---|---|
| A | Timestamp | Automatik dari Form |
| B | Nama | Nama guru |
| C | Emel | Untuk notifikasi, tidak diimport ke Fail B |
| D | Bilik | Dropdown: Bilik Seminar, Bilik Teori PPKI |
| E | Tarikh | DD/MM/YYYY |
| F | Mula | Dropdown 07:30–16:30 |
| G | Tamat | Dropdown 08:00–17:00 |
| H | Tujuan | Teks bebas |
| I | Status | Apps Script: Disahkan, Bertindih, atau Masa tidak sah |
| J | Catatan | Apps Script: sebab status (contoh: "Bertindih dengan Cikgu Ali 10:00–11:00") |

Apps Script baca lajur ikut **nama header**, bukan kedudukan. Masa dibaca sama ada dalam bentuk teks ("07:30") atau nilai Time.

**Tab `Tetapan`** (Fail A)

| Kunci | Contoh nilai | Keterangan |
|---|---|---|
| `MULA_AWAL` | 07:30 | Masa mula paling awal |
| `TAMAT_AKHIR` | 17:00 | Masa tamat paling lewat |
| `EMEL_ADMIN` | admin1@..., admin2@... | Penerima notifikasi admin, dipisah koma |

Bila had masa ditukar, admin perlu kemas kini **dua tempat**: dropdown dalam Form dan tab `Tetapan`.

## 9. Paparan dan Reka Bentuk

**Palet warna**

| Fungsi | Latar | Teks | Digunakan di |
|---|---|---|---|
| Header utama | `#0E5E63` | `#FFFFFF` | Fail A dan B |
| Latar belakang | `#E8F4F5` | — | Fail B |
| Kosong | `#FFFFFF` | `#1E6B3E` | Fail B |
| Ditempah | `#F8D7DE` | `#9B2C4A` | Fail B |
| Bertindih | `#F59E0B` | `#1F2937` | Fail A sahaja |
| Bilik Seminar | `#3B6FB6` | `#FFFFFF` | Header bilik |
| Bilik Teori PPKI | `#7C5CBF` | `#FFFFFF` | Header bilik |

Semua pasangan warna di atas dianggarkan melepasi kontras 4.5:1 (WCAG AA). Teks putih tidak digunakan atas hijau atau oren.

**Prinsip**

- Warna bilik hanya pada header bilik. Warna status hanya pada sel slot.
- Setiap sel ada teks ("Kosong" atau nama guru). Jangan bergantung pada warna sahaja.
- Bertindih tidak dipaparkan dalam Kalendar, hanya dalam Fail A untuk rujukan admin.

**Kalendar mingguan** (tab `Mingguan`)
- Satu dropdown pilih bilik, satu sel tarikh pilih minggu.
- Baris: slot 30 minit dari 07:30 hingga 17:00 (19 baris).
- Lajur: Isnin hingga Ahad.
- Sel: nama guru dan tujuan ringkas. Tempahan yang meliputi beberapa slot dipaparkan pada setiap slot.

**Kalendar bulanan** (tab `Bulanan`)
- Satu sel tarikh pilih bulan.
- Setiap hari papar jumlah tempahan per bilik (contoh: "Seminar: 2, PPKI: 1").

## 10. Notifikasi (Emel)

| Penerima | Bila | Kandungan |
|---|---|---|
| Guru | Disahkan | Bilik, tarikh, hari, masa, tujuan |
| Guru | Bertindih | Slot telah ditempah, minta pilih masa lain dan rujuk Kalendar |
| Guru | Masa tidak sah | Sebab ditolak, minta hantar semula |
| Dua admin | Disahkan | Nama, bilik, tarikh, masa, tujuan |
| Dua admin | Bertindih | Butiran tempahan yang gagal |

Admin tidak diberi notifikasi untuk `Masa tidak sah` bagi mengurangkan emel.

## 11. Keselamatan dan PDPA

- Sistem untuk kegunaan dalaman sekolah sahaja.
- Fail A tidak dikongsi kepada guru. Fail B tidak mengandungi emel.
- Fail tidak diterbitkan ke web.
- Nama guru dipaparkan dalam Kalendar untuk kegunaan dalaman. Emel guru tidak dipaparkan.
- Tiada data guru dihantar ke pihak ketiga atau group chat.
- Deskripsi Form mengandungi notis ringkas: data yang dikumpul, tujuan, siapa boleh akses, dan tempoh simpanan.
- Apps Script guna `LockService` supaya dua tempahan serentak tidak boleh kedua-duanya disahkan.
- Tempoh simpanan (*cadangan, perlu disahkan*): data tahun semasa disimpan. Pada hujung tahun persekolahan, baris tahun lepas dipindah ke fail arkib milik sekolah, dan dipadam selepas setahun.

**Notis PDPA dalam Form (draf)**

> Maklumat anda (nama, emel, butiran tempahan) digunakan untuk urusan tempahan bilik sahaja. Nama dan butiran tempahan dipaparkan dalam kalendar dalaman untuk guru SMK Mengkarak. Emel anda hanya dilihat oleh admin sistem. Data disimpan hingga hujung tahun persekolahan berikutnya.

## 12. Had Teknikal

- Kuota emel Apps Script: sekitar 100 penerima sehari untuk Gmail biasa, lebih tinggi untuk Workspace. Setiap tempahan guna sehingga 3 penerima (guru + dua admin).
- Form terbuka kepada sesiapa yang ada pautan jika bukan Workspace. Hadkan kepada domain sekolah jika ada Workspace.
- `IMPORTRANGE` perlu kebenaran sekali pada kali pertama.
- Apps Script perlu dijalankan sekali (`pasangTrigger`) untuk pasang trigger `onFormSubmit`.
- Trigger hanya jalan bila borang dihantar. Bila admin edit masa atau padam baris:
  - Semakan bertindih tidak dibuat semula. Admin perlu semak sendiri.
  - Tempahan `Bertindih` untuk slot yang sama tidak naik jadi `Disahkan` secara automatik.
- Jangan tambah, buang, atau tukar susunan soalan Form selepas sistem digunakan. Tambah pilihan dropdown (contoh bilik baharu) tidak mengapa.

## 13. Migrasi dari Telegram

- Telegram masih digunakan untuk tempahan sekarang. Sistem baharu bermula bila tarikh mula ditetapkan.
- Tempahan sedia ada yang masih akan datang dimasukkan oleh admin melalui Form, menggunakan nama dan emel guru asal. Guru akan terima emel pengesahan seperti biasa.
- Jika ada pertindihan antara tempahan Telegram, admin selesaikan dengan guru terlibat sebelum masukkan.
- *Tarikh mula dan siapa yang masukkan belum ditetapkan.*

## 14. Kriteria Siap

- Tempahan kosong disahkan dan emel dihantar dalam masa beberapa saat.
- Tempahan bertindih ditandakan dengan betul dan emel dihantar.
- Masa tamat ≤ masa mula, atau tarikh lepas, ditandakan `Masa tidak sah` dan guru terima emel dengan sebab.
- Kalendar mingguan dan bulanan betul untuk kedua-dua bilik.
- Warna dan teks status boleh dibezakan tanpa bergantung pada warna.
- Guru tidak boleh buka Fail A. Fail B, termasuk bila dimuat turun, tidak mengandungi emel.

**Kes ujian minimum**

| # | Kes | Jangkaan |
|---|---|---|
| 1 | Slot kosong | Disahkan |
| 2 | Bertindih separa dengan tempahan sedia ada | Bertindih |
| 3 | Bersentuhan di sempadan (10:00–11:00 lepas tu 11:00–12:00) | Kedua-duanya Disahkan |
| 4 | Tamat sama atau lebih awal dari mula | Masa tidak sah |
| 5 | Tarikh lepas | Masa tidak sah |
| 6 | Slot terakhir (16:30–17:00) | Disahkan |
| 7 | Masa sama, bilik berbeza | Kedua-duanya Disahkan |
| 8 | Dua borang dihantar serentak untuk slot sama | Satu Disahkan, satu Bertindih |
| 9 | Muat turun Fail B sebagai .xlsx | Tiada emel dalam fail |

## 15. Soalan Terbuka

1. Adakah had masa tamat betul-betul 17:00?
2. Akaun Workspace for Education atau Gmail biasa?
3. Siapa dua admin, dan siapa pemilik Fail A?
4. Setuju dengan cadangan tempoh simpanan di bahagian 11?
5. Bila tarikh mula sistem baharu, dan siapa masukkan tempahan yang sedia ada?

## 16. Langkah Seterusnya

1. Sahkan soalan terbuka di bahagian 15.
2. Sediakan Fail A: Form, tab `Tempahan`, tab `Tetapan`, locale dan timezone.
3. Tulis Apps Script: semakan masa, semakan bertindih dengan `LockService`, emel, dan `pasangTrigger`.
4. Bina Fail B dengan tab `Mingguan` dan `Bulanan`, uji dengan data contoh.
5. Uji dengan sembilan kes di bahagian 14.
6. Semak dengan dua admin sebelum guru guna.
