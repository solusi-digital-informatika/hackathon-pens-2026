# Design Tokens — Teman UMKM

Format implementasi: CSS Custom Properties (`:root`).
Frontend mengimpor token ini di layer global (`index.css` atau `tokens.css`).

---

## 1. Color Palette & Semantic Roles

> **Aturan Desain Anti-Slop:**
> - Hanya 1 warna aksen: **Emerald Forest** (melambangkan ketenangan, stabilitas, dan pertumbuhan bisnis terpercaya).
> - Palet netral ditinting halus ke arah aksen (slate-sage) untuk nuansa visual kohesif.
> - Tidak ada bayangan (shadow) pada kartu istirahat (resting cards); diferensiasi menggunakan kontras border dan permukaan.
> - Seluruh kombinasi teks dan latar memenuhi standar kontras **WCAG AA (minimal 4.5:1 untuk teks normal, 3:1 untuk elemen UI grafis)**.

```css
:root {
  /* === Brand & Primary Accent (Emerald Forest) === */
  --color-primary-50:  #F0FDF9;
  --color-primary-100: #CCFBF1;
  --color-primary-200: #99F6E4;
  --color-primary-300: #5EEAD4;
  --color-primary-400: #2DD4BF;
  --color-primary-500: #14B8A6;
  --color-primary-600: #0D9488;
  --color-primary-700: #0F766E; /* Default interactive primary */
  --color-primary-800: #115E59; /* Hover / Pressed primary */
  --color-primary-900: #134E48;

  /* Semantic Primary */
  --color-primary:       var(--color-primary-700);
  --color-primary-hover: var(--color-primary-800);
  --color-primary-subtle: var(--color-primary-50);
  --color-primary-border: var(--color-primary-200);
  --color-primary-text:   #FFFFFF; /* Contrast on primary: 5.6:1 */

  /* === Surfaces & Backgrounds === */
  --color-bg-app:        #F7F9F8; /* Latar kanvas aplikasi utama */
  --color-bg-surface:    #FFFFFF; /* Kartu, panel, dan kontainer utama */
  --color-bg-subtle:     #EFF2F1; /* Baris zebra tabel, header kolom, input read-only */
  --color-bg-hover:      #E6EBE9; /* Interaksi hover baris tabel atau item menu */
  --color-bg-active:     #DCE3E0; /* Keadaan ditekan (active) */

  /* === Borders (Surface Contrast) === */
  --color-border-subtle:  #E2E8E5; /* Pemisah baris tabel dan batas kartu standar */
  --color-border-default: #CBD5D1; /* Border input formulir & komponen kontrol */
  --color-border-strong:  #94A39F; /* Border fokus & pembatas seksi utama */
  --color-border-focus:   var(--color-primary-700); /* Ring fokus aksesibilitas */

  /* === Typography & Content === */
  --color-text-main:      #131F1C; /* Teks judul, angka metrik, isi utama (kontras 15.2:1) */
  --color-text-muted:     #4B5E59; /* Label formulir, teks sekunder, petunjuk (kontras 5.8:1) */
  --color-text-tertiary:  #6D827C; /* Placeholder & status non-kritis (kontras 4.5:1) */
  --color-text-disabled:  #9EADA8; /* Elemen non-aktif */
  --color-text-inverse:   #FFFFFF; /* Teks di atas warna gelap atau primer */

  /* === Functional / Feedback States === */
  /* Sukses (Transaksi tercatat, data cocok 100%, impor berhasil) */
  --color-success-bg:     #F0FDF4;
  --color-success-border: #BBF7D0;
  --color-success-text:   #166534; /* Kontras 5.5:1 */
  --color-success-icon:   #15803D;

  /* Peringatan (Estimasi laba, HPP belum lengkap, transaksi tanpa pelanggan) */
  --color-warning-bg:     #FFFBEB;
  --color-warning-border: #FDE68A;
  --color-warning-text:   #854D0E; /* Kontras 4.8:1 */
  --color-warning-icon:   #B45309;

  /* Bahaya / Error (Validasi gagal, transaksi dibatalkan, harga jual di bawah HPP) */
  --color-error-bg:       #FEF2F2;
  --color-error-border:   #FECACA;
  --color-error-text:     #991B1B; /* Kontras 6.1:1 */
  --color-error-icon:     #DC2626;

  /* Info / Edukasi (Panduan rumus deterministik, petunjuk pemetaan kolom) */
  --color-info-bg:        #F0F9FF;
  --color-info-border:    #BAE6FD;
  --color-info-text:      #075985; /* Kontras 5.3:1 */
  --color-info-icon:      #0284C7;
}
```

---

## 2. Typography Scale

> **Aturan Desain:**
> - Maksimal 2 keluarga huruf:
>   - **Plus Jakarta Sans**: Tampilan (Display), Heading, dan label navigasi/antarmuka umum.
>   - **Inter**: Data finansial, tabel transaksi, harga Rupiah, dan perhitungan numerik (mengaktifkan `tabular-nums`).
> - Tidak ada label "eyebrow" berlebihan (small uppercase badges di atas judul).
> - Judul halaman maksimal 2 baris pada desktop.

```css
:root {
  /* Font Families */
  --font-family-sans:    "Plus Jakarta Sans", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
  --font-family-tabular: "Inter", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;

  /* Font Weights */
  --font-weight-regular: 400;
  --font-weight-medium:  500;
  --font-weight-semibold: 600;
  --font-weight-bold:    700;

  /* Type Sizes & Line Heights (1.25 Modular Scale) */
  --text-xs-size:        0.75rem;   /* 12px */
  --text-xs-line-height: 1rem;      /* 16px */

  --text-sm-size:        0.875rem;  /* 14px - Standar body tabel & input */
  --text-sm-line-height: 1.25rem;   /* 20px */

  --text-base-size:       1rem;      /* 16px - Standar body teks & tombol */
  --text-base-line-height: 1.5rem;   /* 24px */

  --text-md-size:        1.125rem;  /* 18px - Subheading & kartu ringkasan */
  --text-md-line-height: 1.625rem;  /* 26px */

  --text-lg-size:        1.25rem;   /* 20px - Section heading */
  --text-lg-line-height: 1.75rem;   /* 28px */

  --text-xl-size:        1.5rem;    /* 24px - Judul halaman */
  --text-xl-line-height: 2rem;      /* 32px */

  --text-2xl-size:       1.875rem;  /* 30px - Metrik angka utama / Laba Kotor */
  --text-2xl-line-height: 2.25rem;  /* 36px */

  --text-3xl-size:       2.25rem;   /* 36px - Hero display metrik */
  --text-3xl-line-height: 2.5rem;   /* 40px */
}

/* Helper Class untuk Angka Keuangan */
.tnum,
.financial-figure {
  font-family: var(--font-family-tabular);
  font-feature-settings: "tnum" 1, "cv05" 1;
  font-variant-numeric: tabular-nums;
  letter-spacing: -0.01em;
}
```

---

## 3. Spacing Scale (4-pt Grid)

> **Aturan Desain:**
> - Hirarki pengelompokan menggunakan jarak (spacing) terlebih dahulu sebelum menambah garis border.
> - Skala 4-pt ketat tanpa nilai sembarang (*no arbitrary values*).

```css
:root {
  --space-0:   0px;
  --space-1:   4px;   /* 0.25rem - Jarak mikro: badge padding, icon gap */
  --space-2:   8px;   /* 0.5rem  - Jarak kompak: elemen inline, tombol padding-y */
  --space-3:   12px;  /* 0.75rem - Jarak kontrol formulir, input padding */
  --space-4:   16px;  /* 1rem    - Standar grid gutter, card padding dalam */
  --space-6:   24px;  /* 1.5rem  - Jarak antar seksi kartu, layout panel */
  --space-8:   32px;  /* 2rem    - Jarak antar blok halaman */
  --space-12:  48px;  /* 3rem    - Jarak vertikal seksi utama desktop */
  --space-16:  64px;  /* 4rem    - Maksimal pemisah halaman/onboarding */
}
```

---

## 4. Radius Scale

> **Aturan Desain:**
> - Kepribadian konsisten dan lugas (clean modern craftsman).
> - Menghindari bentuk pill berlebihan untuk kartu atau kontainer formulir.

```css
:root {
  --radius-none: 0px;
  --radius-xs:   2px;   /* Bar progress tipis, indikator status mini */
  --radius-sm:   4px;   /* Badge status, tag kategori, tooltip */
  --radius-md:   6px;   /* Tombol, input teks, select dropdown, checkbox */
  --radius-lg:   8px;   /* Kartu metrik, panel data, kontainer tabel */
  --radius-xl:   12px;  /* Dialog modal, floating drawer, popup konfirmasi */
  --radius-full: 9999px;/* Avatar inisial pelanggan, status dot */
}
```

---

## 5. Elevation & Shadows (Raised Layers Only)

> **Aturan Anti-Slop:**
> - **DILARANG memberikan bayangan pada resting card**. Gunakan `border: 1px solid var(--color-border-subtle)` dan latar `var(--color-bg-surface)`.
> - Bayangan hanya digunakan saat lapisan terangkat secara fisik dari kanvas: menu melayang, dialog modal, toast, atau baris yang sedang ditarik (*drag*).

```css
:root {
  --shadow-none:     none;
  --shadow-dropdown: 0 4px 12px rgba(19, 31, 28, 0.08); /* Popover, select menu */
  --shadow-modal:    0 12px 32px rgba(19, 31, 28, 0.14); /* Dialog konfirmasi */
  --shadow-toast:    0 8px 24px rgba(19, 31, 28, 0.12);  /* Notifikasi sistem */
  --shadow-drag:     0 16px 40px rgba(19, 31, 28, 0.18); /* Card drag-and-drop */
}
```

---

## 6. Motion & Transitions

> **Aturan Desain:**
> - Tenang, responsif, tidak memblokir input pengguna (*motion never blocks input*).
> - Durasi:
>   - **Micro (100–150ms)**: Hover, press, toggle checkbox.
>   - **Standard (200–300ms)**: Dialog modal, dropdown enter/exit, tabs, alert toast.
>   - **Page (≤ 400ms)**: Transisi layar/rute, expand panel wizard.
> - Easing:
>   - Masuk (*enter*): Deceleration cepat (`cubic-bezier(0.22, 1, 0.36, 1)`).
>   - Keluar (*exit*): Akselerasi bersih (`cubic-bezier(0.4, 0, 1, 1)`).
>   - Tanpa efek membal (*no bounce/spring*) pada antarmuka kerja fungsional.
> - Properti yang dianimasikan: hanya `opacity`, `transform`, dan `color`/`background-color`. Jangan pernah menganimasikan `width`, `height`, `top`, `left`.

```css
:root {
  /* Durasi */
  --duration-micro:    120ms;
  --duration-standard: 240ms;
  --duration-page:     350ms;

  /* Kurva Easing */
  --ease-enter:    cubic-bezier(0.22, 1, 0.36, 1); /* Ease-out natural */
  --ease-exit:     cubic-bezier(0.4, 0, 1, 1);      /* Ease-in tegas */
  --ease-standard: cubic-bezier(0.2, 0, 0, 1);      /* Transisi seimbang */

  /* Transisi Siap Pakai */
  --transition-interactive: color var(--duration-micro) var(--ease-standard),
                            background-color var(--duration-micro) var(--ease-standard),
                            border-color var(--duration-micro) var(--ease-standard);

  --transition-overlay:     opacity var(--duration-standard) var(--ease-enter),
                            transform var(--duration-standard) var(--ease-enter);
}

/* Reduced Motion Override (Aksesibilitas Wajib) */
@media (prefers-reduced-motion: reduce) {
  :root {
    --duration-micro: 0ms;
    --duration-standard: 0ms;
    --duration-page: 0ms;
  }

  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

### Spesifikasi Gerak Antarmuka (Trigger → Property → Duration → Easing)

| Komponen | Pemicu (Trigger) | Properti Diubah | Durasi | Easing |
| :--- | :--- | :--- | :--- | :--- |
| Tombol / Nav Item | Hover | `background-color`, `border-color` | 120ms | `--ease-standard` |
| Tombol / Link | Press (Active) | `transform: scale(0.98)` | 100ms | `--ease-exit` |
| Input / Kontrol | Focus-visible | `outline-color`, `border-color` | 120ms | `--ease-standard` |
| Menu Dropdown | Open | `opacity: 0 → 1`, `transform: translateY(-4px → 0)` | 200ms | `--ease-enter` |
| Menu Dropdown | Close | `opacity: 1 → 0`, `transform: translateY(0 → -4px)` | 150ms | `--ease-exit` |
| Dialog Modal | Enter | `opacity: 0 → 1`, `transform: scale(0.96 → 1)` | 240ms | `--ease-enter` |
| Dialog Modal | Exit | `opacity: 1 → 0`, `transform: scale(1 → 0.96)` | 180ms | `--ease-exit` |
| Toast Notifikasi | Slide In | `opacity: 0 → 1`, `transform: translateY(8px → 0)` | 240ms | `--ease-enter` |
| Halaman / Tab | Switch | `opacity: 0 → 1`, `transform: translateY(6px → 0)` | 250ms | `--ease-enter` |

---

## 7. Z-Index Scale

```css
:root {
  --z-deep:     -1;  /* Latar belakang dekoratif / canvas base */
  --z-base:      0;  /* Elemen resting di halaman */
  --z-raised:    1;  /* Sticky table column / hovering row */
  --z-sticky:   10;  /* Sticky table header / filter bar */
  --z-header:   20;  /* App bar & navigation utama */
  --z-dropdown: 30;  /* Select popover, context menu */
  --z-backdrop: 40;  /* Modal backdrop & drawer overlay */
  --z-modal:    50;  /* Dialog box, drawer dialog */
  --z-toast:    60;  /* Toast notification banner */
  --z-tooltip:  70;  /* Tooltip penjelasan metrik */
}
```

---

## 8. Standar Format Angka & Mata Uang (IDR)

Sesuai PRD §8.5 dan TRD §3 / §5:

1. **Rupiah Standar**:
   - Format: `Rp X.XXX.XXX` (tanpa desimal untuk transaksi nominal bulat, koma desimal bila ada sen/HPP presisi).
   - Kode Implementasi Frontend:
     ```typescript
     export const formatRupiah = (amount: number | string): string => {
       const numeric = typeof amount === 'string' ? parseFloat(amount) : amount;
       if (isNaN(numeric)) return 'Rp 0';
       return new Intl.NumberFormat('id-ID', {
         style: 'currency',
         currency: 'IDR',
         minimumFractionDigits: 0,
         maximumFractionDigits: 2,
       }).format(numeric);
     };
     ```
2. **Kaidah Terminologi Finansial Wajib**:
   - Gunakan **"Laba Kotor"** (Dilarang hanya menuliskan "Laba").
   - Jika `cost_coverage < 100%`, sertakan badge **"Estimasi"** (`--color-warning-bg` + `--color-warning-text`) beserta jumlah produk tanpa HPP.
   - Jika data biaya operasional belum dicatat (MVP), tampilkan **"Tidak tersedia"** disertai penjelasan singkat (sesuai TRD A4).
3. **Standar Tanggal**:
   - Format tampilan: `DD/MM/YYYY` (misal: `18/10/2026`) atau format panjang lokal `18 Okt 2026`.
