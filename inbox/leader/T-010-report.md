# Laporan Penyelesaian Tugas T-010: UI Spec for Auth & Business Setup

**Kepada:** Leader  
**Dari:** Designer  
**Status Tugas:** Selesai (`done`)  
**File Dihasilkan:** `contracts/ui-auth-business.md`

---

## 1. Ringkasan Eksekusi

Sesuai instruksi pada `inbox/designer/T-010-ui-spec-auth-business.md`, PRD (§5.1, §6.1), TRD (§5 TD-05 & TD-06), dan pedoman SOUL Designer:

1. **Spesifikasi Lengkap UI/UX (`contracts/ui-auth-business.md`)**:
   - **Feature → Page Map**:
     - Registrasi Pengguna (`/register`)
     - Login Pengguna (`/login`)
     - Setup Profil Usaha / Onboarding (`/onboarding`)
     - Pengaturan Profil Bisnis & Preferensi (`/settings`)
     - Penanganan Sesi Kedaluwarsa (*Session Expired*)
   - **Komponen Antarmuka Bersama**:
     - `AuthCard`: Kontainer form tanpa bayangan istirahat (*resting box-shadow*), mengandalkan kontras border dan background permukaan.
     - `FormInput`: Label eksplisit (*visible label*), pembantu teks, status validasi, dan toggle buka-tutup kata sandi dengan target sentuh ≥ 44×44px.
     - `SelectDropdown`, `PrimaryButton`, `SecondaryButton`, dan `AlertBanner` lengkap dengan matriks status (*default, hover, focus-visible, active, disabled, validation error*).

2. **Spesifikasi Halaman & Responsif (390 / 768 / 1440)**:
   - Setiap halaman memuat: tujuan, pengguna & tugas utama, aksi primer, alur masuk/keluar, data ditampilkan, breakdown responsif mobile/tablet/desktop, states, spesifikasi gerak (*motion specs*), dan catatan aksesibilitas.
   - Mengadopsi prinsip Anti-Slop ketat: tanpa gradien dekoratif AI, tanpa bayangan kartu istirahat, salinan teks Bahasa Indonesia yang fungsional dan lugas.

3. **Keamanan & Aksesibilitas (Security UX & A11y)**:
   - **Generic Auth Error**: Pesan error seragam `"Email atau password salah."` pada kegagalan autentikasi untuk mencegah enumerasi akun (mengikuti TRD TD-06).
   - **Session Expired UX**: Penanganan token kedaluwarsa dengan penyimpanan URL asal (`redirect_to`) dan alert banner penjelasan transparan saat dialihkan ke login.
   - **A11y**: Kepatuhan WCAG AA untuk kontras teks/UI, navigasi keyboard penuh, pemenuhan Fitts's law (target sentuh ≥ 44×44px), serta pemenuhan preferensi gerak berkurang (*reduced motion*).

4. **Status Papan Kerja & Arsip**:
   - Status T-010 pada `board/board.md` telah diperbarui dari `doing` menjadi `done`.
   - File tugas `inbox/designer/T-010-ui-spec-auth-business.md` dipindahkan ke `inbox/designer/done/T-010-ui-spec-auth-business.md`.

Spesifikasi ini siap menjadi acuan langsung bagi implementasi Frontend (T-013) serta penyelarasan kontrak API Backend (T-011).
