# Brief: Sistem Tempahan Bilik SMK Mengkarak (Semarak Booking)

**Versi:** 0.4 (dikemas kini mengikut keputusan pemilik)
**Tarikh:** 29/10/2026
**Pemilik:** Guru Sains Komputer

---

## 1. Tujuan

Menggantikan tempahan bilik melalui Telegram dengan sistem yang lebih teratur. Guru dan staf boleh tempah bilik, dan melihat slot kosong serta siapa yang tempah dalam kalendar. Sistem ini untuk kegunaan dalaman sekolah sahaja.

## 2. Masalah yang Diselesaikan

- Sukar tahu bilik mana yang kosong.
- Tempahan bertindih.
- Format masa dan tarikh tidak konsisten dalam Telegram.
- Ada tempahan tanpa masa tamat.

## 3. Skop Versi Pertama

**Dalam skop**

- Google Form sebagai pintu masuk tempahan untuk guru dan staf.
- Dua bilik: Bilik Seminar dan Bilik Teori PPKI. Bilik lain ditambah kemudian.
- Semakan bertindih dan masa tidak sah oleh Apps Script.
- Emel kepada penempah untuk setiap keputusan (Disahkan, Bertindih, Masa tidak sah).
- Emel notifikasi kepada dua admin bila ada tempahan bertindih.
- Fail Kalendar berasingan dengan tiga paparan: `Hari Ini` (untuk telefon), `Mingguan`, dan `Bulanan`.
- Admin boleh sekat bilik (peperiksaan, majlis, penyelenggaraan).

**Di luar skop (versi pertama)**

- Log masuk, akaun, atau kata laluan.
- Aplikasi mudah alih atau web app.
- Integrasi Google Calendar. Dipertimbangkan untuk versi kedua jika paparan `Hari Ini` tidak mencukupi.
- Telegram, WhatsApp, atau bot. Telegram kekal untuk sebar pautan sahaja, bukan untuk tempahan.
- Tempahan berbilang tarikh dalam satu borang. Penempah hantar berasingan untuk setiap tarikh.
- Tempahan berulang setiap minggu. Tiada kelas tetap yang guna bilik ini.
- Kelulusan pentadbir. Slot kosong terus disahkan, siapa cepat dia dapat.
- Fungsi batal tempahan oleh penempah.
- Peraturan no-show.
- Semakan semula automatik bila admin edit atau padam baris.
- Peringatan esok melalui trigger harian. Ditangguhkan ke versi kedua.
- Medan sasaran, penceramah, atau kategori program. Cukup dimasukkan dalam medan Tujuan.
- Tempahan untuk pentadbiran (contoh: penempatan sementara pelajar). Ini bukan tempahan bilik.

## 4. Pengguna dan Peranan

| Peranan | Bilangan | Boleh buat | Tidak boleh buat |
|---|---|---|---|
| Guru dan staf | Semua guru dan staf | Isi borang tempahan, lihat Fail B (Kalendar), terima emel | Edit atau padam tempahan, buka Fail A, tukar peraturan, tempah lebih 60 hari ke depan |
| Admin | Dua orang (*belum diputuskan*) | Semua yang guru buat, terima notifikasi bertindih, padam atau betulkan baris, sekat bilik, tambah bilik, tukar had dalam tab `Tetapan`, jalankan `pasangTrigger` | Tiada had dari segi akses data |
| Penyelaras bilik | Seorang setiap bilik | Pegang kunci bilik. Tiada akses khas dalam sistem | — |

Semua guru dan staf boleh tempah semua bilik. Tiada keutamaan untuk bilik tertentu.

Penempah tidak perlu akaun. Peranan dikawal melalui akses kepada fail, bukan dalam aplikasi.

## 5. Peraturan Tempahan

- Siapa cepat dapat. Slot kosong terus disahkan, tiada kelulusan pentadbir.
- Tempahan pada hari yang sama dibenarkan selagi slot kosong.
- Tarikh yang sudah lepas ditolak.
- Tarikh lebih **60 hari** dari hari ini ditolak.
- Sabtu, Ahad, cuti umum, dan cuti sekolah dibenarkan. Sistem tidak semak hari cuti.
- Masa dipilih dari dropdown slot 30 minit:
  - Mula: 07:30 hingga 21:30
  - Tamat: 08:00 hingga 22:00
- Tempoh minimum sesi 30 minit. Masa tamat mesti lebih lewat daripada masa mula.
- Tiada had tempoh maksimum sesi. Tempahan sehari suntuk dibenarkan.
- Hari (Isnin hingga Ahad) dikira automatik daripada tarikh, tidak ditanya dalam borang.

Semua tempahan yang melanggar peraturan di atas ditandakan `Masa tidak sah`.

**Definisi bertindih**

Dua tempahan bertindih jika bilik sama, tarikh sama, dan:

```
mulaBaru < tamatLama  DAN  tamatBaru > mulaLama
```

Tempahan yang bersentuhan di sempadan **tidak** bertindih. Contoh: 10:00–11:00 dan 11:00–12:00 kedua-duanya boleh disahkan.

Semakan hanya dibuat terhadap tempahan berstatus `Disahkan`. Tempahan `Bertindih` atau `Masa tidak sah` tidak menghalang tempahan lain.

**Sekatan oleh admin**

- Dalam tempoh 60 hari: admin hantar Form seperti biasa, dengan nama `ADMIN` dan tujuan `Disekat: <sebab>`.
- Lebih 60 hari: admin taip terus dalam tab `Tempahan` dan isi status `Disahkan` sendiri. Baris ini tetap dikira dalam semakan bertindih.
- Admin tidak dikecualikan melalui emel dalam Form. Emel ditaip bebas, jadi sesiapa boleh menyamar emel admin.

## 6. Aliran Pengguna

1. Penempah buka pautan Google Form (dipin dalam group Telegram guru).
2. Penempah isi: nama, emel, bilik, tarikh, masa mula, masa tamat, dan tujuan. Semua medan wajib.
3. Apps Script semak masa dan bertindih secara automatik.
4. Jika masa tidak sah: status `Masa tidak sah`. Penempah dapat emel dengan sebab.
5. Jika kosong: status `Disahkan`. Penempah dapat emel pengesahan.
6. Jika bertindih: status `Bertindih`. Penempah dapat emel dengan nama guru yang sudah tempah slot itu. Dua admin dapat notifikasi.
7. Penempah lihat slot kosong dalam Fail B (Kalendar). Di telefon, guna tab `Hari Ini`.
8. Jika ada kesilapan atau nak batal, penempah emel admin. Admin betulkan dalam Fail A.

## 7. Struktur Fail

Dua fail Google Sheets, dimiliki oleh **akaun Gmail khas sistem** (contoh: `tempahan.smkmengkarak@gmail.com`). Bukan akaun peribadi sesiapa. Akaun ini yang diserahkan bila admin bertukar.

**Fail A: Tempahan (admin sahaja)**
- Form dihubungkan ke fail ini. Apps Script tinggal di sini.
- Tab `Tempahan`: data mentah dari Form.
- Tab `Tetapan`: nilai yang admin boleh ubah tanpa sentuh code.
- Dikongsi kepada dua admin sahaja sebagai Editor.

**Fail B: Kalendar (guru dan staf lihat sahaja)**
- Tab `Hari Ini`, `Mingguan`, dan `Bulanan`. Tiada data asal, hanya paparan.
- Tarik data dari Fail A menggunakan `QUERY(IMPORTRANGE(...))`.
- Hanya lajur Nama, Bilik, Tarikh, Mula, Tamat, Tujuan diimport, dan hanya baris `Disahkan`.
- Lajur Emel **tidak diimport langsung**. Sorok lajur tidak cukup, kerana sesiapa yang boleh lihat fail boleh muat turun dan nampak lajur tersorok.
- Perkongsian: "Anyone with the link can view". Pautan dipin dalam group Telegram guru sahaja.

**Tetapan fail (kedua-dua fail dan Apps Script)**
- Locale: Malaysia
- Timezone: `Asia/Kuala_Lumpur`

**Tetapan Google Form**
- Matikan "Allow response editing". Tempahan yang diedit selepas hantar tidak akan disemak semula.
- Semua soalan wajib (required).
- Medan emel ditaip oleh penempah, dengan response validation jenis emel.
- Deskripsi Form mengandungi notis PDPA dan amaran tentang medan Tujuan (lihat bahagian 11).

## 8. Struktur Data

**Tab `Tempahan`** (Fail A)

| Lajur | Medan | Keterangan |
|---|---|---|
| A | Timestamp | Automatik dari Form. Kosong untuk baris yang admin taip sendiri |
| B | Nama | Ditaip bebas |
| C | Emel | Untuk notifikasi, tidak diimport ke Fail B |
| D | Bilik | Dropdown: Bilik Seminar, Bilik Teori PPKI |
| E | Tarikh | DD/MM/YYYY |
| F | Mula | Dropdown 07:30–21:30 |
| G | Tamat | Dropdown 08:00–22:00 |
| H | Tujuan | Teks bebas, dipaparkan dalam Kalendar |
| I | Status | Apps Script: Disahkan, Bertindih, atau Masa tidak sah |
| J | Catatan | Apps Script: sebab status (contoh: "Bertindih dengan Cikgu Ali 10:00–11:00") |

Apps Script baca lajur ikut **nama header**, bukan kedudukan. Masa dibaca sama ada dalam bentuk teks ("07:30") atau nilai Time.

**Tab `Tetapan`** (Fail A)

| Kunci | Nilai awal | Keterangan |
|---|---|---|
| `MULA_AWAL` | 07:30 | Masa mula paling awal |
| `TAMAT_AKHIR` | 22:00 | Masa tamat paling lewat |
| `HARI_KE_DEPAN` | 60 | Had berapa hari ke depan boleh tempah |
| `EMEL_ADMIN` | (belum diputuskan) | Penerima notifikasi admin, dipisah koma |
| `INFO_KUNCI` | Hubungi penyelaras bilik untuk kunci | Ayat dalam emel pengesahan |

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

- Kebanyakan guru buka Kalendar di telefon. `Hari Ini` ialah paparan utama, dan tab ini diletak paling kiri.
- Warna bilik hanya pada header bilik. Warna status hanya pada sel slot.
- Setiap sel ada teks ("Kosong" atau nama guru). Jangan bergantung pada warna sahaja.
- Bertindih tidak dipaparkan dalam Kalendar, hanya dalam Fail A untuk rujukan admin.

**`Hari Ini`** (untuk telefon)
- Senarai menegak tempahan `Disahkan` dari hari ini hingga 7 hari ke depan.
- Disusun ikut tarikh, kemudian masa mula.
- Lajur sedikit sahaja supaya muat skrin telefon: Tarikh/Hari, Masa, Bilik, Nama, Tujuan.
- Tiada input. Dikemas kini automatik ikut `TODAY()`.

**`Mingguan`** (untuk komputer)
- Satu dropdown pilih bilik, satu sel tarikh pilih minggu.
- Baris: slot 30 minit dari 07:30 hingga 22:00 (29 baris).
- Lajur: Isnin hingga Ahad.
- Sel: nama guru dan tujuan ringkas. Tempahan yang meliputi beberapa slot dipaparkan pada setiap slot.
- Baris 17:00 ke atas boleh dikumpulkan (row group) supaya grid tidak terlalu panjang bila tiada program malam.

**`Bulanan`**
- Satu sel tarikh pilih bulan.
- Setiap hari papar jumlah tempahan per bilik (contoh: "Seminar: 2, PPKI: 1").

## 10. Notifikasi (Emel)

| Penerima | Bila | Kandungan |
|---|---|---|
| Penempah | Disahkan | Bilik, tarikh, hari, masa, tujuan, `INFO_KUNCI` |
| Penempah | Bertindih | Slot telah ditempah oleh siapa dan pada masa apa, minta pilih masa lain dan rujuk Kalendar |
| Penempah | Masa tidak sah | Sebab ditolak, minta hantar semula |
| Dua admin | Bertindih | Butiran tempahan yang gagal dan tempahan sedia ada |

Admin tidak terima emel untuk tempahan `Disahkan` atau `Masa tidak sah`. Admin semak tab `Tempahan` secara berkala, terutama untuk kesan tempahan palsu.

## 11. Keselamatan dan PDPA

- Sistem untuk kegunaan dalaman sekolah sahaja.
- Fail A tidak dikongsi kepada guru. Fail B tidak mengandungi emel.
- Fail tidak diterbitkan ke web ("Publish to web" tidak digunakan).
- Nama dan tujuan dipaparkan dalam Kalendar. Emel tidak dipaparkan.
- Tiada data guru dihantar ke pihak ketiga atau group chat. Hanya pautan yang dikongsi dalam group.
- Apps Script guna `LockService` supaya dua tempahan serentak tidak boleh kedua-duanya disahkan.
- Tempoh simpanan (*cadangan, perlu disahkan*): data tahun semasa disimpan. Pada hujung tahun persekolahan, baris tahun lepas dipindah ke fail arkib milik akaun sistem, dan dipadam selepas setahun.

**Risiko yang diterima kerana guna Gmail biasa**

- Form terbuka kepada sesiapa yang ada pautan. Tempahan palsu tidak boleh dihalang sepenuhnya. Kawalan: pautan hanya dalam group guru, admin semak tab `Tempahan` secara berkala dan padam baris palsu.
- Fail B boleh dibuka sesiapa yang ada pautan. Kesan jika bocor: nama guru dan tujuan tempahan terdedah. Emel tidak terdedah.
- Emel ditaip bebas. Sesiapa boleh masukkan emel orang lain.

**Notis dalam Form (draf)**

> Maklumat anda (nama, emel, butiran tempahan) digunakan untuk urusan tempahan bilik sahaja. Nama, bilik, masa dan tujuan akan dipaparkan dalam kalendar yang dikongsi kepada guru dan staf SMK Mengkarak. Emel anda hanya dilihat oleh admin sistem. Data disimpan hingga hujung tahun persekolahan berikutnya.
>
> **Jangan tulis maklumat sensitif dalam medan Tujuan** (contoh: nama pelajar, kes disiplin, kaunseling).

## 12. Had Teknikal

- Kuota emel Apps Script untuk Gmail biasa sekitar 100 penerima sehari. Kebanyakan tempahan guna 1 emel, tempahan bertindih guna 3. Cukup untuk kegunaan sekolah.
- `IMPORTRANGE` perlu kebenaran sekali pada kali pertama.
- Apps Script perlu dijalankan sekali (`pasangTrigger`) untuk pasang trigger `onFormSubmit`.
- Trigger hanya jalan bila borang dihantar. Bila admin edit masa atau padam baris:
  - Semakan bertindih tidak dibuat semula. Admin perlu semak sendiri.
  - Tempahan `Bertindih` untuk slot yang sama tidak naik jadi `Disahkan` secara automatik.
- Jangan tambah, buang, atau tukar susunan soalan Form selepas sistem digunakan. Tambah pilihan dropdown (contoh bilik baharu) tidak mengapa.
- Google Sheets di telefon perlu app Google Sheets. Pautan dibuka melalui browser mungkin minta pasang app.

## 13. Migrasi dari Telegram

- Telegram masih digunakan untuk tempahan sekarang. Sistem baharu bermula selepas pilot.
- Tempahan sedia ada yang masih akan datang dimasukkan oleh admin melalui Form, menggunakan nama dan emel guru asal. Guru akan terima emel pengesahan seperti biasa.
- Jika ada pertindihan antara tempahan Telegram, admin selesaikan dengan guru terlibat sebelum masukkan.
- Pada tarikh mula, umumkan dalam group Telegram bahawa tempahan baharu hanya melalui Form. Pin pautan Form dan Kalendar.
- *Tarikh mula dan siapa yang masukkan belum ditetapkan.*

## 14. Pilot

- 5 orang guru atau staf guna sistem selama seminggu, sambil Telegram masih berjalan.
- Pilih sekurang-kurangnya seorang yang biasa guna telefon sahaja.
- Perkara yang dinilai:
  - Borang mudah diisi di telefon?
  - Emel sampai, atau masuk folder Spam?
  - Tab `Hari Ini` mudah dibaca di telefon?
  - Ada kes yang brief ini tidak jangka?
- Selepas pilot, betulkan isu dan buka kepada semua.

## 15. Serahan Tugas

Bila pemilik sistem atau admin bertukar, lantik admin baharu dan serahkan:

- Kata laluan akaun Gmail khas sistem (tukar kata laluan selepas serahan).
- Dokumen ringkas: cara tambah bilik, tukar had masa, padam tempahan, sekat bilik, dan jalankan `pasangTrigger` semula.
- Kemas kini `EMEL_ADMIN` dalam tab `Tetapan`.
- Buang akses admin lama dari Fail A.

## 16. Kriteria Siap

- Tempahan kosong disahkan dan emel dihantar dalam masa beberapa saat.
- Tempahan bertindih ditandakan dengan betul, penempah dimaklumkan siapa yang sudah tempah, dan admin dapat notifikasi.
- Tempahan tidak sah ditandakan `Masa tidak sah` dan penempah terima emel dengan sebab.
- Paparan `Hari Ini`, `Mingguan`, dan `Bulanan` betul untuk kedua-dua bilik.
- `Hari Ini` boleh dibaca tanpa zoom di telefon.
- Warna dan teks status boleh dibezakan tanpa bergantung pada warna.
- Guru tidak boleh buka Fail A. Fail B, termasuk bila dimuat turun, tidak mengandungi emel.

**Kes ujian minimum**

| # | Kes | Jangkaan |
|---|---|---|
| 1 | Slot kosong | Disahkan |
| 2 | Bertindih separa dengan tempahan sedia ada | Bertindih, emel sebut nama guru asal |
| 3 | Bersentuhan di sempadan (10:00–11:00 lepas tu 11:00–12:00) | Kedua-duanya Disahkan |
| 4 | Tamat sama atau lebih awal dari mula | Masa tidak sah |
| 5 | Tarikh lepas | Masa tidak sah |
| 6 | Tarikh 61 hari ke depan | Masa tidak sah |
| 7 | Slot terakhir (21:30–22:00) | Disahkan |
| 8 | Masa sama, bilik berbeza | Kedua-duanya Disahkan |
| 9 | Dua borang dihantar serentak untuk slot sama | Satu Disahkan, satu Bertindih |
| 10 | Admin taip sekatan terus dalam Sheet (90 hari ke depan), kemudian guru tempah slot sama melalui Form bila sudah dalam 60 hari | Bertindih |
| 11 | Muat turun Fail B sebagai .xlsx | Tiada emel dalam fail |
| 12 | Buka `Hari Ini` di telefon | Boleh dibaca tanpa zoom |

## 17. Soalan Terbuka

1. Siapa dua admin? (Perlu sebelum pilot.)
2. Setuju dengan cadangan tempoh simpanan di bahagian 11?
3. Bila tarikh mula pilot dan sistem penuh, dan siapa masukkan tempahan yang sedia ada?
4. Siapa penyelaras bilik untuk setiap bilik?

## 18. Langkah Seterusnya

1. Cipta akaun Gmail khas sistem.
2. Sediakan Fail A: Form, tab `Tempahan`, tab `Tetapan`, locale dan timezone.
3. Tulis Apps Script: semakan masa, semakan bertindih dengan `LockService`, emel, dan `pasangTrigger`.
4. Bina Fail B dengan tab `Hari Ini`, `Mingguan`, dan `Bulanan`, uji dengan data contoh.
5. Uji dengan 12 kes di bahagian 16.
6. Lantik dua admin dan jalankan pilot (bahagian 14).
7. Migrasi tempahan Telegram dan buka kepada semua.
