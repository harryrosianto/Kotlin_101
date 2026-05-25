

## Daftar Isi

1. [Fragment](#1-fragment)
   - [Konsep Fragment](#konsep-fragment)
   - [Lifecycle Fragment](#lifecycle-fragment)
   - [Membuat Fragment](#membuat-fragment)
   - [Fragment Manager & Transaction](#fragment-manager--transaction)
   - [Kirim Data ke Fragment](#kirim-data-ke-fragment)
   - [Komunikasi Fragment ke Activity](#komunikasi-fragment-ke-activity)
   - [Fragment dengan Bottom Navigation](#fragment-dengan-bottom-navigation)
   - [Case: Aplikasi Multi-Halaman](#case-aplikasi-multi-halaman)
2. [Firebase](#2-firebase)
   - [Konsep Firebase](#konsep-firebase)
   - [Setup & Koneksi Firebase](#setup--koneksi-firebase)
   - [Firebase Authentication](#firebase-authentication)
   - [Firebase Firestore](#firebase-firestore)
   - [Firebase Realtime Database](#firebase-realtime-database)
   - [Firebase Storage](#firebase-storage)
   - [Case: Aplikasi Chat Sederhana](#case-aplikasi-chat-sederhana)
3. [MongoDB Atlas](#3-mongodb-atlas)
   - [Konsep MongoDB Atlas](#konsep-mongodb-atlas)
   - [Arsitektur Koneksi Android → MongoDB](#arsitektur-koneksi-android--mongodb)
   - [Setup Backend API (Node.js/Express)](#setup-backend-api-nodejs--express)
   - [Koneksi dari Android dengan Retrofit](#koneksi-dari-android-dengan-retrofit)
   - [CRUD Lengkap](#crud-lengkap)
   - [Case: Aplikasi Katalog Produk](#case-aplikasi-katalog-produk)

---

# 1. Fragment

## Konsep Fragment

### Apa itu Fragment?
Fragment adalah **"sub-activity"** — potongan UI yang bisa ditempel ke dalam Activity. Bayangkan layar HP dibagi-bagi jadi beberapa bagian yang bisa diganti-ganti tanpa harus pindah Activity.

```
Tanpa Fragment:              Dengan Fragment:
┌──────────────┐            ┌──────────────┐
│  Activity A  │            │   Activity   │
│  (halaman 1) │            │  ┌────────┐  │
└──────────────┘            │  │Frag  A │  │
        ↓ intent            │  └────────┘  │
┌──────────────┐            │     ↓ replace │
│  Activity B  │            │  ┌────────┐  │
│  (halaman 2) │            │  │Frag  B │  │
└──────────────┘            │  └────────┘  │
                            └──────────────┘
```

### Kapan Pakai Fragment vs Activity?

| Situasi | Gunakan |
|---------|---------|
| Navigasi tab (Bottom Nav) | Fragment |
| Layar penuh terpisah (login, onboarding) | Activity |
| Panel kiri/kanan di tablet | Fragment |
| Wizard / step-by-step form | Fragment |
| Dialog kompleks | Fragment (DialogFragment) |
| Halaman utama app | Fragment di dalam 1 Activity |

### Keuntungan Fragment
- Bisa **reuse** — satu fragment bisa dipasang di banyak activity
- **Back stack** — bisa kembali ke fragment sebelumnya
- **Modular** — setiap fitur dalam fragment-nya sendiri
- Lebih efisien dari membuat Activity baru

---

## Lifecycle Fragment

Fragment punya lifecycle lebih panjang dari Activity:

```
Activity onCreate()
      │
      ▼
onAttach()       ← Fragment terhubung ke Activity
      │
      ▼
onCreate()       ← Fragment dibuat (belum ada View)
      │
      ▼
onCreateView()   ← Inflate layout fragment
      │
      ▼
onViewCreated()  ← View sudah siap, setup listener di sini
      │
      ▼
onStart()        ← Fragment terlihat
      │
      ▼
onResume()       ← User bisa berinteraksi
      │
      ▼
   [AKTIF]
      │
      ▼
onPause()
      │
      ▼
onStop()
      │
      ▼
onDestroyView()  ← View dihancurkan (Fragment masih ada)
      │
      ▼
onDestroy()      ← Fragment dihancurkan
      │
      ▼
onDetach()       ← Fragment lepas dari Activity
```

> ⚠️ **Penting:** Jangan akses View di luar rentang `onViewCreated()` sampai `onDestroyView()` — View bisa null!

---

## Membuat Fragment

### Tambah Dependency
```kotlin
// build.gradle (app)
dependencies {
    implementation("androidx.fragment:fragment-ktx:1.8.5")
    implementation("androidx.navigation:navigation-fragment-ktx:2.8.5")
    implementation("androidx.navigation:navigation-ui-ktx:2.8.5")
}
```

### Layout Fragment: `fragment_home.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp"
    tools:context=".fragment.HomeFragment">

    <TextView
        android:id="@+id/tvJudul"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Halaman Home"
        android:textSize="24sp"
        android:textStyle="bold"
        android:layout_marginBottom="16dp" />

    <TextView
        android:id="@+id/tvKonten"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textSize="16sp"
        tools:text="Konten akan muncul di sini" />

    <com.google.android.material.button.MaterialButton
        android:id="@+id/btnKirimData"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="Kirim Data ke Activity" />

</LinearLayout>
```

### HomeFragment.kt
```kotlin
package com.example.app.fragment

import android.content.Context
import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import androidx.fragment.app.Fragment
import com.example.app.databinding.FragmentHomeBinding

class HomeFragment : Fragment() {

    // ── View Binding ──────────────────────────────────────────────────────────
    // Binding dideklarasikan nullable karena View bisa null di luar lifecycle
    private var _binding: FragmentHomeBinding? = null
    private val binding get() = _binding!!   // shortcut non-null

    // ── Lifecycle ─────────────────────────────────────────────────────────────

    // 1. Inflate layout
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = FragmentHomeBinding.inflate(inflater, container, false)
        return binding.root
    }

    // 2. View sudah siap — setup di sini
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        // Ambil argumen yang dikirim ke fragment ini
        val nama = arguments?.getString("NAMA") ?: "Tamu"
        binding.tvKonten.text = "Selamat datang, $nama!"

        // Setup listener
        binding.btnKirimData.setOnClickListener {
            // Kirim data ke Activity lewat interface
            callback?.onDataDikirim("Data dari HomeFragment")
        }
    }

    // 3. WAJIB: set binding ke null saat view dihancurkan (cegah memory leak)
    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }

    // ── Komunikasi ke Activity (via Interface) ────────────────────────────────

    interface HomeCallback {
        fun onDataDikirim(data: String)
    }

    private var callback: HomeCallback? = null

    override fun onAttach(context: Context) {
        super.onAttach(context)
        // Activity harus mengimplementasikan HomeCallback
        if (context is HomeCallback) {
            callback = context
        }
    }

    override fun onDetach() {
        super.onDetach()
        callback = null
    }

    // ── Companion Object (Factory Method) ─────────────────────────────────────
    companion object {
        // Cara buat instance fragment dengan argumen (cara idiomatik)
        fun newInstance(nama: String): HomeFragment {
            return HomeFragment().apply {
                arguments = Bundle().apply {
                    putString("NAMA", nama)
                }
            }
        }
    }
}
```

---

## Fragment Manager & Transaction

Fragment Manager adalah yang mengatur semua fragment di dalam Activity.

### Container di Activity Layout: `activity_main.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <com.google.android.material.appbar.MaterialToolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        app:title="Fragment Demo" />

    <!-- Container tempat fragment akan ditampilkan -->
    <FrameLayout
        android:id="@+id/fragmentContainer"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
```

### Operasi Fragment di Activity
```kotlin
class MainActivity : AppCompatActivity(), HomeFragment.HomeCallback {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        // Tampilkan fragment pertama saat activity dibuat
        // Cek savedInstanceState agar tidak double-load saat rotasi layar
        if (savedInstanceState == null) {
            tampilkanFragment(HomeFragment.newInstance("Budi"))
        }
    }

    // ── Operasi Fragment ──────────────────────────────────────────────────────

    // ADD — tambahkan fragment (bisa ditumpuk)
    private fun tambahFragment(fragment: Fragment) {
        supportFragmentManager.beginTransaction()
            .add(R.id.fragmentContainer, fragment)
            .commit()
    }

    // REPLACE — ganti fragment (yang lama hilang dari view)
    private fun tampilkanFragment(fragment: Fragment, addToBackStack: Boolean = true) {
        supportFragmentManager.beginTransaction()
            .replace(R.id.fragmentContainer, fragment)
            .apply {
                if (addToBackStack) {
                    addToBackStack(null)   // bisa kembali dengan tombol back
                }
            }
            .setCustomAnimations(
                android.R.anim.fade_in,    // animasi masuk
                android.R.anim.fade_out    // animasi keluar
            )
            .commit()
    }

    // REMOVE — hapus fragment tertentu
    private fun hapusFragment(fragment: Fragment) {
        supportFragmentManager.beginTransaction()
            .remove(fragment)
            .commit()
    }

    // HIDE / SHOW — sembunyikan tanpa hapus
    private fun sembunyikanFragment(fragment: Fragment) {
        supportFragmentManager.beginTransaction()
            .hide(fragment)
            .commit()
    }

    // Cek apakah ada fragment di back stack
    private fun adaBackStack(): Boolean {
        return supportFragmentManager.backStackEntryCount > 0
    }

    // Navigasi ke fragment lain
    fun keHalamanProfil() {
        tampilkanFragment(ProfilFragment.newInstance())
    }

    // ── Implementasi Callback dari Fragment ───────────────────────────────────
    override fun onDataDikirim(data: String) {
        Toast.makeText(this, "Diterima: $data", Toast.LENGTH_SHORT).show()
    }
}
```

---

## Kirim Data ke Fragment

### Cara 1 — Bundle Arguments (Direkomendasikan)
```kotlin
// Di Activity / Fragment pengirim:
val fragment = DetailFragment().apply {
    arguments = Bundle().apply {
        putString("JUDUL", "Produk A")
        putInt("ID", 42)
        putDouble("HARGA", 150_000.0)
        putBoolean("TERSEDIA", true)
        putParcelable("USER", userObject)        // untuk data class (perlu @Parcelize)
        putSerializable("LIST", arrayListOf(1,2)) // untuk list
    }
}
supportFragmentManager.beginTransaction()
    .replace(R.id.container, fragment)
    .commit()

// Di DetailFragment — terima data:
override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    super.onViewCreated(view, savedInstanceState)

    val judul    = arguments?.getString("JUDUL") ?: ""
    val id       = arguments?.getInt("ID") ?: 0
    val harga    = arguments?.getDouble("HARGA") ?: 0.0
    val tersedia = arguments?.getBoolean("TERSEDIA") ?: false

    binding.tvJudul.text = judul
    binding.tvHarga.text = "Rp ${harga.toLong()}"
}
```

### Cara 2 — ViewModel Bersama (Shared ViewModel)
Untuk berbagi data antara beberapa fragment dalam satu Activity:

```kotlin
// SharedViewModel.kt — data dibagi antar fragment
class SharedViewModel : ViewModel() {
    val produkDipilih = MutableLiveData<Produk>()
    val keranjang = MutableLiveData<MutableList<Produk>>(mutableListOf())

    fun pilihProduk(produk: Produk) {
        produkDipilih.value = produk
    }

    fun tambahKeKeranjang(produk: Produk) {
        keranjang.value?.add(produk)
        keranjang.notifyObserver()
    }
}

// Di Fragment A — set data
class ListProdukFragment : Fragment() {
    // activityViewModels() → shared dengan semua fragment di Activity yang sama
    private val viewModel: SharedViewModel by activityViewModels()

    fun onProdukDiklik(produk: Produk) {
        viewModel.pilihProduk(produk)
        // navigasi ke fragment B
    }
}

// Di Fragment B — observe data
class DetailProdukFragment : Fragment() {
    private val viewModel: SharedViewModel by activityViewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        // Otomatis update saat data berubah
        viewModel.produkDipilih.observe(viewLifecycleOwner) { produk ->
            binding.tvNama.text  = produk.nama
            binding.tvHarga.text = "Rp ${produk.harga}"
        }
    }
}
```

---

## Komunikasi Fragment ke Activity

Ada tiga cara:

### Cara 1 — Interface (seperti di atas)
```kotlin
// Di Fragment
interface OnFormSelesai {
    fun onSimpan(nama: String, email: String)
    fun onBatal()
}

// Activity implementasikan:
class MainActivity : AppCompatActivity(), FormFragment.OnFormSelesai {
    override fun onSimpan(nama: String, email: String) { /* ... */ }
    override fun onBatal() { supportFragmentManager.popBackStack() }
}
```

### Cara 2 — setFragmentResult (Modern, API 1.3+)
```kotlin
// Fragment pengirim — tidak perlu tahu siapa penerimanya
binding.btnSimpan.setOnClickListener {
    val result = Bundle().apply {
        putString("NAMA", binding.etNama.text.toString())
        putInt("UMUR", 25)
    }
    setFragmentResult("formKey", result)   // kirim dengan key
    parentFragmentManager.popBackStack()   // kembali
}

// Fragment / Activity penerima
supportFragmentManager.setFragmentResultListener("formKey", this) { key, bundle ->
    val nama = bundle.getString("NAMA")
    val umur = bundle.getInt("UMUR")
    Toast.makeText(this, "Data diterima: $nama, $umur", Toast.LENGTH_SHORT).show()
}
```

### Cara 3 — ViewModel (sudah dibahas di atas)

---

## Fragment dengan Bottom Navigation

### Layout: `activity_main.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <FrameLayout
        android:id="@+id/fragmentContainer"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toTopOf="@id/bottomNav"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottomNav"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        app:menu="@menu/bottom_nav_menu"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

### Menu: `res/menu/bottom_nav_menu.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">

    <item
        android:id="@+id/nav_home"
        android:icon="@drawable/ic_home"
        android:title="Home" />

    <item
        android:id="@+id/nav_explore"
        android:icon="@drawable/ic_explore"
        android:title="Jelajah" />

    <item
        android:id="@+id/nav_profil"
        android:icon="@drawable/ic_person"
        android:title="Profil" />

</menu>
```

### MainActivity.kt dengan Bottom Navigation
```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    // Simpan referensi fragment agar tidak dibuat ulang saat pindah tab
    private val homeFragment    = HomeFragment.newInstance("Budi")
    private val exploreFragment = ExploreFragment()
    private val profilFragment  = ProfilFragment()
    private var activeFragment: Fragment = homeFragment

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        setupFragments()
        setupBottomNav()
    }

    private fun setupFragments() {
        // Tambahkan semua fragment sekaligus, tapi sembunyikan yang tidak aktif
        supportFragmentManager.beginTransaction()
            .add(R.id.fragmentContainer, profilFragment, "profil").hide(profilFragment)
            .add(R.id.fragmentContainer, exploreFragment, "explore").hide(exploreFragment)
            .add(R.id.fragmentContainer, homeFragment, "home")  // home aktif duluan
            .commit()
    }

    private fun setupBottomNav() {
        binding.bottomNav.setOnItemSelectedListener { item ->
            val targetFragment = when (item.itemId) {
                R.id.nav_home    -> homeFragment
                R.id.nav_explore -> exploreFragment
                R.id.nav_profil  -> profilFragment
                else             -> return@setOnItemSelectedListener false
            }
            gantiFragment(targetFragment)
            true
        }
    }

    // Sembunyikan yang aktif, tampilkan yang baru — tanpa recreate fragment
    private fun gantiFragment(target: Fragment) {
        supportFragmentManager.beginTransaction()
            .hide(activeFragment)
            .show(target)
            .commit()
        activeFragment = target
    }
}
```

---

## Case: Aplikasi Multi-Halaman

Aplikasi sederhana dengan 3 tab: **Home** (daftar artikel), **Cari** (search), **Profil** (info user).

### Struktur
```
app/
├── fragment/
│   ├── HomeFragment.kt         ← daftar artikel
│   ├── CariFragment.kt         ← search
│   └── ProfilFragment.kt       ← profil user
├── adapter/
│   └── ArtikelAdapter.kt
├── model/
│   └── Artikel.kt
└── MainActivity.kt
```

### Model
```kotlin
data class Artikel(
    val id: Int,
    val judul: String,
    val isi: String,
    val penulis: String,
    val tanggal: String
)
```

### HomeFragment.kt dengan RecyclerView
```kotlin
class HomeFragment : Fragment() {

    private var _binding: FragmentHomeBinding? = null
    private val binding get() = _binding!!
    private lateinit var adapter: ArtikelAdapter

    // Data dummy
    private val daftarArtikel = listOf(
        Artikel(1, "Belajar Kotlin", "Kotlin adalah bahasa...", "Budi", "25 Mei 2026"),
        Artikel(2, "Android Studio Tips", "Tips produktif...", "Ani", "24 Mei 2026"),
        Artikel(3, "Jetpack Compose", "Cara baru bikin UI...", "Citra", "23 Mei 2026")
    )

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View {
        _binding = FragmentHomeBinding.inflate(inflater, container, false)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        adapter = ArtikelAdapter(daftarArtikel) { artikel ->
            // Navigasi ke detail
            val detail = DetailArtikelFragment.newInstance(artikel.id, artikel.judul)
            parentFragmentManager.beginTransaction()
                .replace(R.id.fragmentContainer, detail)
                .addToBackStack("detail")
                .commit()
        }

        binding.rvArtikel.apply {
            layoutManager = LinearLayoutManager(requireContext())
            adapter = this@HomeFragment.adapter
        }
    }

    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
}
```

---

# 2. Firebase

## Konsep Firebase

Firebase adalah platform **Backend-as-a-Service (BaaS)** dari Google yang menyediakan:

```
Firebase Platform
├── Authentication    ← Login (email, Google, dll)
├── Firestore         ← Database NoSQL modern (dokumen/koleksi)
├── Realtime Database ← Database JSON real-time (lebih lama)
├── Storage           ← Simpan file (foto, video, dll)
├── Cloud Functions   ← Server-side logic
├── FCM               ← Push notification
└── Analytics         ← Statistik pengguna
```

### Firestore vs Realtime Database
| | Firestore | Realtime Database |
|-|-----------|-------------------|
| Struktur | Dokumen & Koleksi | JSON Tree |
| Query | Powerful (filter, sort) | Terbatas |
| Offline | ✅ Otomatis | ✅ |
| Skalabilitas | Lebih baik | OK untuk kecil |
| Harga | Lebih mahal di scale | Lebih murah |
| **Rekomendasi** | **Proyek baru** | Real-time sederhana |

---

## Setup & Koneksi Firebase

### Langkah 1 — Buat Project di Firebase Console
```
1. Buka console.firebase.google.com
2. Klik "Add project"
3. Isi nama project → Continue
4. Enable/disable Google Analytics → Create project
```

### Langkah 2 — Daftarkan App Android
```
1. Di dashboard Firebase → klik ikon Android
2. Isi package name (harus sama dengan di build.gradle!)
   contoh: com.example.myapp
3. Isi nickname app (opsional)
4. Download file google-services.json
5. Taruh google-services.json di folder app/ (bukan root project!)
```

```
MyApp/
├── app/
│   ├── google-services.json   ← taruh di sini ✅
│   └── src/...
└── build.gradle
```

### Langkah 3 — Tambah Plugin & Dependency
```kotlin
// build.gradle (Project level)
plugins {
    id("com.google.gms.google-services") version "4.4.2" apply false
}
```

```kotlin
// build.gradle (App level)
plugins {
    id("com.android.application")
    id("com.google.gms.google-services")   // ← tambahkan ini
    id("org.jetbrains.kotlin.android")
}

dependencies {
    // Firebase BOM — satu versi untuk semua library Firebase
    implementation(platform("com.google.firebase:firebase-bom:33.7.0"))

    // Pilih sesuai yang dipakai
    implementation("com.google.firebase:firebase-auth-ktx")
    implementation("com.google.firebase:firebase-firestore-ktx")
    implementation("com.google.firebase:firebase-database-ktx")
    implementation("com.google.firebase:firebase-storage-ktx")

    // Coroutines untuk Firebase
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-play-services:1.8.1")
}
```

### Langkah 4 — Verifikasi Koneksi
```kotlin
// Tidak perlu inisialisasi manual — google-services.json sudah mengurus semuanya
// Cukup akses langsung:

val db   = Firebase.firestore
val auth = Firebase.auth
val rtdb = Firebase.database
val storage = Firebase.storage
```

---

## Firebase Authentication

### Aktifkan di Firebase Console
```
Firebase Console → Authentication → Sign-in method → Enable:
  ✅ Email/Password
  ✅ Google (opsional)
```

### Layout Login: `activity_login.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    android:padding="24dp">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Login"
        android:textSize="32sp"
        android:textStyle="bold"
        android:layout_marginBottom="32dp" />

    <com.google.android.material.textfield.TextInputLayout
        android:id="@+id/tilEmail"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:hint="Email"
        android:layout_marginBottom="12dp">
        <com.google.android.material.textfield.TextInputEditText
            android:id="@+id/etEmail"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="textEmailAddress" />
    </com.google.android.material.textfield.TextInputLayout>

    <com.google.android.material.textfield.TextInputLayout
        android:id="@+id/tilPassword"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:hint="Password"
        app:passwordToggleEnabled="true"
        android:layout_marginBottom="24dp">
        <com.google.android.material.textfield.TextInputEditText
            android:id="@+id/etPassword"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="textPassword" />
    </com.google.android.material.textfield.TextInputLayout>

    <com.google.android.material.button.MaterialButton
        android:id="@+id/btnLogin"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Login"
        android:layout_marginBottom="8dp" />

    <com.google.android.material.button.MaterialButton
        android:id="@+id/btnDaftar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Daftar Akun Baru"
        style="@style/Widget.MaterialComponents.Button.OutlinedButton" />

    <ProgressBar
        android:id="@+id/progressBar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:visibility="gone" />

</LinearLayout>
```

### LoginActivity.kt
```kotlin
class LoginActivity : AppCompatActivity() {

    private lateinit var binding: ActivityLoginBinding
    private val auth = Firebase.auth

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityLoginBinding.inflate(layoutInflater)
        setContentView(binding.root)

        // Kalau sudah login, langsung masuk
        if (auth.currentUser != null) {
            masukKeApp()
            return
        }

        setupListeners()
    }

    private fun setupListeners() {
        binding.btnLogin.setOnClickListener { login() }
        binding.btnDaftar.setOnClickListener { daftar() }
    }

    // ── LOGIN ─────────────────────────────────────────────────────────────────
    private fun login() {
        val email    = binding.etEmail.text.toString().trim()
        val password = binding.etPassword.text.toString()

        // Validasi lokal
        if (email.isEmpty()) { binding.tilEmail.error = "Email wajib diisi"; return }
        if (password.length < 6) { binding.tilPassword.error = "Password minimal 6 karakter"; return }
        binding.tilEmail.error = null
        binding.tilPassword.error = null

        tampilkanLoading(true)

        auth.signInWithEmailAndPassword(email, password)
            .addOnSuccessListener {
                tampilkanLoading(false)
                masukKeApp()
            }
            .addOnFailureListener { e ->
                tampilkanLoading(false)
                val pesan = when {
                    e.message?.contains("password") == true -> "Password salah"
                    e.message?.contains("user") == true     -> "Email tidak terdaftar"
                    else                                    -> "Login gagal: ${e.message}"
                }
                Snackbar.make(binding.root, pesan, Snackbar.LENGTH_LONG).show()
            }
    }

    // ── DAFTAR ────────────────────────────────────────────────────────────────
    private fun daftar() {
        val email    = binding.etEmail.text.toString().trim()
        val password = binding.etPassword.text.toString()

        if (email.isEmpty() || password.length < 6) {
            Toast.makeText(this, "Isi email dan password (min 6 karakter)", Toast.LENGTH_SHORT).show()
            return
        }

        tampilkanLoading(true)

        auth.createUserWithEmailAndPassword(email, password)
            .addOnSuccessListener { result ->
                tampilkanLoading(false)
                // Simpan profil awal ke Firestore
                simpanProfilUser(result.user?.uid ?: "", email)
                masukKeApp()
            }
            .addOnFailureListener { e ->
                tampilkanLoading(false)
                Toast.makeText(this, "Gagal daftar: ${e.message}", Toast.LENGTH_LONG).show()
            }
    }

    // Simpan data user ke Firestore setelah register
    private fun simpanProfilUser(uid: String, email: String) {
        val db = Firebase.firestore
        val user = hashMapOf(
            "uid"       to uid,
            "email"     to email,
            "nama"      to "",
            "createdAt" to FieldValue.serverTimestamp()
        )
        db.collection("users").document(uid).set(user)
    }

    // ── LOGOUT ────────────────────────────────────────────────────────────────
    // Panggil dari menu atau tombol logout
    private fun logout() {
        auth.signOut()
        recreate()   // restart activity → kembali ke form login
    }

    // ── HELPER ───────────────────────────────────────────────────────────────
    private fun masukKeApp() {
        startActivity(Intent(this, MainActivity::class.java))
        finish()
    }

    private fun tampilkanLoading(show: Boolean) {
        binding.progressBar.visibility = if (show) View.VISIBLE else View.GONE
        binding.btnLogin.isEnabled     = !show
        binding.btnDaftar.isEnabled    = !show
    }
}
```

### Cek User yang Sedang Login
```kotlin
val auth = Firebase.auth

// Cek apakah sudah login
val userSaatIni = auth.currentUser
if (userSaatIni != null) {
    val uid   = userSaatIni.uid            // ID unik user
    val email = userSaatIni.email          // email
    val nama  = userSaatIni.displayName    // nama (jika diset)
    println("Login sebagai: $email (uid: $uid)")
} else {
    println("Belum login")
}

// Pantau perubahan status login secara real-time
auth.addAuthStateListener { firebaseAuth ->
    if (firebaseAuth.currentUser != null) {
        println("User login")
    } else {
        println("User logout")
    }
}
```

---

## Firebase Firestore

### Konsep Struktur Firestore
```
Firestore
└── Collection "users"              ← seperti tabel
    ├── Document "uid_abc123"       ← seperti baris
    │   ├── nama: "Budi"
    │   ├── email: "budi@mail.com"
    │   └── umur: 25
    └── Document "uid_def456"
        ├── nama: "Ani"
        └── ...

└── Collection "artikel"
    ├── Document "artikel_001"
    │   ├── judul: "Belajar Kotlin"
    │   ├── isi: "..."
    │   ├── penulis: "Budi"
    │   └── createdAt: Timestamp
    └── Document (auto-id)
        └── ...
```

### CRUD Firestore Lengkap
```kotlin
class FirestoreManager {

    private val db = Firebase.firestore

    // ── CREATE ────────────────────────────────────────────────────────────────

    // Tambah dengan ID otomatis
    fun tambahArtikel(judul: String, isi: String, penulis: String) {
        val artikel = hashMapOf(
            "judul"     to judul,
            "isi"       to isi,
            "penulis"   to penulis,
            "createdAt" to FieldValue.serverTimestamp(),
            "likes"     to 0
        )

        db.collection("artikel")
            .add(artikel)          // ID otomatis
            .addOnSuccessListener { docRef ->
                println("Artikel ditambahkan dengan ID: ${docRef.id}")
            }
            .addOnFailureListener { e ->
                println("Gagal tambah: ${e.message}")
            }
    }

    // Tambah dengan ID manual
    fun tambahUserDenganId(uid: String, nama: String, email: String) {
        val user = mapOf(
            "uid"   to uid,
            "nama"  to nama,
            "email" to email
        )

        db.collection("users").document(uid)
            .set(user)
            .addOnSuccessListener { println("User $uid disimpan") }
            .addOnFailureListener { println("Gagal: ${it.message}") }
    }

    // ── READ — Sekali Baca ────────────────────────────────────────────────────

    // Baca satu dokumen
    fun bacaUser(uid: String) {
        db.collection("users").document(uid)
            .get()
            .addOnSuccessListener { doc ->
                if (doc.exists()) {
                    val nama  = doc.getString("nama")
                    val email = doc.getString("email")
                    println("User: $nama <$email>")

                    // Atau langsung konversi ke data class
                    val user = doc.toObject(User::class.java)
                    println(user)
                } else {
                    println("Dokumen tidak ditemukan")
                }
            }
            .addOnFailureListener { println("Gagal baca: ${it.message}") }
    }

    // Baca semua dokumen di koleksi
    fun bacaSemuaArtikel() {
        db.collection("artikel")
            .orderBy("createdAt", Query.Direction.DESCENDING)  // urutkan terbaru
            .limit(20)                                         // maks 20 item
            .get()
            .addOnSuccessListener { querySnapshot ->
                val daftarArtikel = querySnapshot.documents.map { doc ->
                    Artikel(
                        id      = doc.id,
                        judul   = doc.getString("judul") ?: "",
                        isi     = doc.getString("isi") ?: "",
                        penulis = doc.getString("penulis") ?: ""
                    )
                }
                println("Jumlah artikel: ${daftarArtikel.size}")
            }
    }

    // Query dengan filter
    fun cariArtikelOleh(penulis: String) {
        db.collection("artikel")
            .whereEqualTo("penulis", penulis)
            .whereGreaterThan("likes", 10)
            .orderBy("likes", Query.Direction.DESCENDING)
            .get()
            .addOnSuccessListener { result ->
                for (doc in result) {
                    println("${doc.id} → ${doc.getString("judul")}")
                }
            }
    }

    // ── READ — Real-time (Listener) ───────────────────────────────────────────
    // Data otomatis update saat berubah di Firestore

    private var listenerRegistration: ListenerRegistration? = null

    fun dengarArtikelRealtime(onUpdate: (List<Artikel>) -> Unit) {
        listenerRegistration = db.collection("artikel")
            .orderBy("createdAt", Query.Direction.DESCENDING)
            .addSnapshotListener { snapshot, error ->
                if (error != null) {
                    println("Error listener: ${error.message}")
                    return@addSnapshotListener
                }

                val daftar = snapshot?.documents?.mapNotNull { doc ->
                    doc.toObject(Artikel::class.java)?.copy(id = doc.id)
                } ?: emptyList()

                onUpdate(daftar)   // panggil callback dengan data terbaru
            }
    }

    // PENTING: hentikan listener saat tidak dipakai (cegah memory leak)
    fun hentikanListener() {
        listenerRegistration?.remove()
    }

    // ── UPDATE ────────────────────────────────────────────────────────────────

    // Update field tertentu saja (tidak menghapus field lain)
    fun updateJudul(artikelId: String, judulBaru: String) {
        db.collection("artikel").document(artikelId)
            .update("judul", judulBaru)
            .addOnSuccessListener { println("Judul diupdate") }
    }

    // Update beberapa field sekaligus
    fun updateArtikel(artikelId: String, data: Map<String, Any>) {
        db.collection("artikel").document(artikelId)
            .update(data)
    }

    // Increment nilai (atomic — aman untuk counter)
    fun tambahLikes(artikelId: String) {
        db.collection("artikel").document(artikelId)
            .update("likes", FieldValue.increment(1))
    }

    // Tambah item ke array
    fun tambahTag(artikelId: String, tag: String) {
        db.collection("artikel").document(artikelId)
            .update("tags", FieldValue.arrayUnion(tag))
    }

    // ── DELETE ────────────────────────────────────────────────────────────────

    // Hapus dokumen
    fun hapusArtikel(artikelId: String) {
        db.collection("artikel").document(artikelId)
            .delete()
            .addOnSuccessListener { println("Artikel dihapus") }
    }

    // Hapus field tertentu saja
    fun hapusField(artikelId: String) {
        db.collection("artikel").document(artikelId)
            .update("draftField", FieldValue.delete())
    }
}
```

### Firestore dengan Coroutines (cara modern)
```kotlin
// Tambah dependency:
// implementation("org.jetbrains.kotlinx:kotlinx-coroutines-play-services:1.8.1")

class ArtikelRepository {

    private val db = Firebase.firestore

    // Dengan await() — bisa dipakai di coroutine
    suspend fun ambilSemuaArtikel(): List<Artikel> {
        return try {
            val snapshot = db.collection("artikel")
                .orderBy("createdAt", Query.Direction.DESCENDING)
                .get()
                .await()   // tunggu hasil tanpa block UI thread

            snapshot.toObjects(Artikel::class.java)
        } catch (e: Exception) {
            println("Error: ${e.message}")
            emptyList()
        }
    }

    suspend fun simpanArtikel(artikel: Artikel): Boolean {
        return try {
            db.collection("artikel").add(artikel).await()
            true
        } catch (e: Exception) {
            false
        }
    }
}

// Di ViewModel
class ArtikelViewModel : ViewModel() {

    private val repo = ArtikelRepository()
    val daftarArtikel = MutableLiveData<List<Artikel>>()
    val isLoading = MutableLiveData<Boolean>()

    fun muatArtikel() {
        viewModelScope.launch {
            isLoading.value = true
            daftarArtikel.value = repo.ambilSemuaArtikel()
            isLoading.value = false
        }
    }
}

// Di Fragment/Activity
viewModel.daftarArtikel.observe(viewLifecycleOwner) { daftar ->
    adapter.updateData(daftar)
}
viewModel.isLoading.observe(viewLifecycleOwner) { loading ->
    binding.progressBar.isVisible = loading
}
viewModel.muatArtikel()
```

---

## Firebase Realtime Database

Cocok untuk data yang benar-benar butuh **real-time** seperti chat, status online, game skor.

### Struktur JSON di Realtime DB
```json
{
  "pesan": {
    "room_001": {
      "-Nabc123": {
        "teks": "Halo semua!",
        "pengirim": "Budi",
        "waktu": 1716624000000
      },
      "-Ndef456": {
        "teks": "Hai Budi!",
        "pengirim": "Ani",
        "waktu": 1716624060000
      }
    }
  },
  "users": {
    "uid_abc": {
      "nama": "Budi",
      "online": true,
      "lastSeen": 1716624000000
    }
  }
}
```

### Setup Rules di Firebase Console
```json
{
  "rules": {
    "pesan": {
      "$roomId": {
        ".read": "auth != null",
        ".write": "auth != null"
      }
    },
    "users": {
      "$uid": {
        ".read": "auth != null",
        ".write": "$uid === auth.uid"
      }
    }
  }
}
```

### Operasi Realtime Database
```kotlin
class RealtimeDBManager {

    private val db = Firebase.database.reference

    // ── TULIS DATA ────────────────────────────────────────────────────────────

    fun kirimPesan(roomId: String, teks: String, pengirim: String) {
        val pesan = mapOf(
            "teks"     to teks,
            "pengirim" to pengirim,
            "waktu"    to ServerValue.TIMESTAMP   // waktu server
        )

        // push() = buat key unik otomatis
        db.child("pesan").child(roomId).push()
            .setValue(pesan)
            .addOnSuccessListener { println("Pesan terkirim") }
            .addOnFailureListener { println("Gagal: ${it.message}") }
    }

    fun updateStatusOnline(uid: String, online: Boolean) {
        db.child("users").child(uid).child("online").setValue(online)
    }

    // ── BACA SEKALI ───────────────────────────────────────────────────────────

    fun bacaPesanSekali(roomId: String) {
        db.child("pesan").child(roomId)
            .get()
            .addOnSuccessListener { snapshot ->
                for (child in snapshot.children) {
                    val teks     = child.child("teks").getValue(String::class.java)
                    val pengirim = child.child("pengirim").getValue(String::class.java)
                    println("$pengirim: $teks")
                }
            }
    }

    // ── DENGARKAN REAL-TIME ───────────────────────────────────────────────────

    private var pesanListener: ValueEventListener? = null

    fun dengarPesan(roomId: String, onPesan: (List<Pesan>) -> Unit) {
        val ref = db.child("pesan").child(roomId)

        pesanListener = object : ValueEventListener {
            override fun onDataChange(snapshot: DataSnapshot) {
                val daftarPesan = snapshot.children.mapNotNull { child ->
                    Pesan(
                        id       = child.key ?: "",
                        teks     = child.child("teks").getValue(String::class.java) ?: "",
                        pengirim = child.child("pengirim").getValue(String::class.java) ?: "",
                        waktu    = child.child("waktu").getValue(Long::class.java) ?: 0L
                    )
                }
                onPesan(daftarPesan)
            }

            override fun onCancelled(error: DatabaseError) {
                println("Error: ${error.message}")
            }
        }

        ref.addValueEventListener(pesanListener!!)
    }

    // Dengarkan hanya pesan BARU (lebih efisien untuk chat)
    fun dengarPesanBaru(roomId: String, onPesanBaru: (Pesan) -> Unit) {
        db.child("pesan").child(roomId)
            .limitToLast(1)   // hanya 1 data terbaru
            .addChildEventListener(object : ChildEventListener {
                override fun onChildAdded(snapshot: DataSnapshot, prev: String?) {
                    val pesan = Pesan(
                        id       = snapshot.key ?: "",
                        teks     = snapshot.child("teks").getValue(String::class.java) ?: "",
                        pengirim = snapshot.child("pengirim").getValue(String::class.java) ?: ""
                    )
                    onPesanBaru(pesan)
                }
                override fun onChildChanged(s: DataSnapshot, p: String?) {}
                override fun onChildRemoved(s: DataSnapshot) {}
                override fun onChildMoved(s: DataSnapshot, p: String?) {}
                override fun onCancelled(e: DatabaseError) {}
            })
    }

    fun hentikanListener(roomId: String) {
        pesanListener?.let {
            db.child("pesan").child(roomId).removeEventListener(it)
        }
    }

    // ── HAPUS ─────────────────────────────────────────────────────────────────

    fun hapusPesan(roomId: String, pesanId: String) {
        db.child("pesan").child(roomId).child(pesanId).removeValue()
    }
}
```

---

## Firebase Storage

Untuk upload/download file (foto profil, dokumen, dll).

```kotlin
class StorageManager(private val context: Context) {

    private val storage = Firebase.storage.reference

    // ── UPLOAD FOTO ───────────────────────────────────────────────────────────

    fun uploadFotoProfil(
        uid: String,
        uri: Uri,
        onProgress: (Int) -> Unit,
        onSelesai: (String) -> Unit,
        onGagal: (String) -> Unit
    ) {
        val ref = storage.child("profil/$uid.jpg")

        ref.putFile(uri)
            .addOnProgressListener { task ->
                val progress = (100.0 * task.bytesTransferred / task.totalByteCount).toInt()
                onProgress(progress)   // update progress bar
            }
            .addOnSuccessListener {
                // Ambil URL download setelah upload selesai
                ref.downloadUrl.addOnSuccessListener { url ->
                    onSelesai(url.toString())
                }
            }
            .addOnFailureListener { e ->
                onGagal(e.message ?: "Upload gagal")
            }
    }

    // ── DOWNLOAD URL ─────────────────────────────────────────────────────────

    fun ambilUrlFoto(uid: String, onUrl: (String) -> Unit) {
        storage.child("profil/$uid.jpg")
            .downloadUrl
            .addOnSuccessListener { uri ->
                onUrl(uri.toString())
            }
            .addOnFailureListener {
                println("Foto tidak ditemukan")
            }
    }

    // ── HAPUS FILE ────────────────────────────────────────────────────────────

    fun hapusFoto(uid: String) {
        storage.child("profil/$uid.jpg")
            .delete()
            .addOnSuccessListener { println("File dihapus") }
    }
}

// Cara pakai di Activity:
// 1. Buka galeri
val launcherGaleri = registerForActivityResult(ActivityResultContracts.GetContent()) { uri ->
    uri?.let { imageUri ->
        val storageManager = StorageManager(this)
        storageManager.uploadFotoProfil(
            uid        = Firebase.auth.uid ?: return@let,
            uri        = imageUri,
            onProgress = { persen -> binding.progressUpload.progress = persen },
            onSelesai  = { url ->
                // Simpan URL ke Firestore
                Firebase.firestore.collection("users").document(Firebase.auth.uid!!)
                    .update("fotoUrl", url)
                Toast.makeText(this, "Foto berhasil diupload!", Toast.LENGTH_SHORT).show()
            },
            onGagal = { pesan -> Toast.makeText(this, pesan, Toast.LENGTH_SHORT).show() }
        )
    }
}
// 2. Panggil launcher
binding.btnGantiFoto.setOnClickListener {
    launcherGaleri.launch("image/*")
}
```

---

## Case: Aplikasi Chat Sederhana

Gabungan Firebase Auth + Realtime Database:

```
app/
├── LoginActivity.kt
├── DaftarRoomActivity.kt
├── ChatActivity.kt
├── model/
│   ├── Pesan.kt
│   └── ChatRoom.kt
└── adapter/
    └── PesanAdapter.kt
```

### Model
```kotlin
data class Pesan(
    val id: String = "",
    val teks: String = "",
    val pengirim: String = "",
    val pengirimUid: String = "",
    val waktu: Long = 0L
)

data class ChatRoom(
    val id: String = "",
    val nama: String = "",
    val terakhirPesan: String = ""
)
```

### ChatActivity.kt
```kotlin
class ChatActivity : AppCompatActivity() {

    private lateinit var binding: ActivityChatBinding
    private lateinit var adapter: PesanAdapter
    private val dbManager = RealtimeDBManager()
    private val auth = Firebase.auth

    private val roomId by lazy { intent.getStringExtra("ROOM_ID") ?: "" }
    private val daftarPesan = mutableListOf<Pesan>()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityChatBinding.inflate(layoutInflater)
        setContentView(binding.root)

        setupRecyclerView()
        dengarPesan()
        setupTombolKirim()
    }

    private fun setupRecyclerView() {
        adapter = PesanAdapter(daftarPesan, auth.uid ?: "")
        binding.rvChat.apply {
            layoutManager = LinearLayoutManager(this@ChatActivity).apply {
                stackFromEnd = true   // mulai dari bawah (seperti WhatsApp)
            }
            adapter = this@ChatActivity.adapter
        }
    }

    private fun dengarPesan() {
        dbManager.dengarPesan(roomId) { pesan ->
            daftarPesan.clear()
            daftarPesan.addAll(pesan)
            adapter.notifyDataSetChanged()
            // Scroll ke pesan terbaru
            if (daftarPesan.isNotEmpty()) {
                binding.rvChat.smoothScrollToPosition(daftarPesan.size - 1)
            }
        }
    }

    private fun setupTombolKirim() {
        binding.btnKirim.setOnClickListener {
            val teks = binding.etPesan.text.toString().trim()
            if (teks.isEmpty()) return@setOnClickListener

            val namaUser = auth.currentUser?.displayName ?: "Anonim"
            dbManager.kirimPesan(roomId, teks, namaUser)
            binding.etPesan.text?.clear()
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        dbManager.hentikanListener(roomId)
    }
}
```

### PesanAdapter.kt
```kotlin
class PesanAdapter(
    private val daftar: List<Pesan>,
    private val myUid: String
) : RecyclerView.Adapter<RecyclerView.ViewHolder>() {

    companion object {
        const val TYPE_SAYA   = 1
        const val TYPE_MEREKA = 2
    }

    override fun getItemViewType(position: Int): Int {
        // Tentukan tipe bubble: kanan (saya) atau kiri (orang lain)
        return if (daftar[position].pengirimUid == myUid) TYPE_SAYA else TYPE_MEREKA
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        val inflater = LayoutInflater.from(parent.context)
        return if (viewType == TYPE_SAYA) {
            val binding = ItemPesanSayaBinding.inflate(inflater, parent, false)
            PesanSayaViewHolder(binding)
        } else {
            val binding = ItemPesanMerekaBinding.inflate(inflater, parent, false)
            PesanMerekaViewHolder(binding)
        }
    }

    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        val pesan = daftar[position]
        when (holder) {
            is PesanSayaViewHolder   -> holder.bind(pesan)
            is PesanMerekaViewHolder -> holder.bind(pesan)
        }
    }

    override fun getItemCount() = daftar.size

    inner class PesanSayaViewHolder(val binding: ItemPesanSayaBinding) :
        RecyclerView.ViewHolder(binding.root) {
        fun bind(pesan: Pesan) {
            binding.tvPesan.text = pesan.teks
            binding.tvWaktu.text = SimpleDateFormat("HH:mm", Locale.getDefault())
                .format(Date(pesan.waktu))
        }
    }

    inner class PesanMerekaViewHolder(val binding: ItemPesanMerekaBinding) :
        RecyclerView.ViewHolder(binding.root) {
        fun bind(pesan: Pesan) {
            binding.tvNama.text  = pesan.pengirim
            binding.tvPesan.text = pesan.teks
            binding.tvWaktu.text = SimpleDateFormat("HH:mm", Locale.getDefault())
                .format(Date(pesan.waktu))
        }
    }
}
```

---

# 3. MongoDB Atlas

## Konsep MongoDB Atlas

MongoDB Atlas adalah layanan **cloud database MongoDB** yang di-manage sepenuhnya — tidak perlu setup server sendiri.

### Perbedaan dengan Firebase Firestore
| | MongoDB Atlas | Firebase Firestore |
|-|---------------|-------------------|
| Provider | MongoDB Inc. | Google |
| Query | Sangat powerful (aggregation) | Menengah |
| Kontrol | Lebih bebas | Lebih opinionated |
| Backend | Butuh server/API sendiri | Firebase Rules |
| Harga | Free tier tersedia | Free tier tersedia |
| Cocok untuk | App kompleks, enterprise | App cepat / startup |

---

## Arsitektur Koneksi Android → MongoDB

> ⚠️ **PENTING:** Android **TIDAK BISA** langsung konek ke MongoDB Atlas!
> Koneksi langsung dari app ke database berbahaya karena credential bisa dibaca dari APK.

```
Arsitektur yang BENAR:

┌─────────────┐     HTTP/REST      ┌─────────────┐      ┌───────────────┐
│   Android   │ ──────────────────► │  Backend    │ ────► │  MongoDB      │
│    App      │ ◄────────────────── │  API        │ ◄──── │  Atlas        │
│  (Retrofit) │    JSON Response    │ (Node.js /  │      │  (Cloud DB)   │
└─────────────┘                     │  Express)   │      └───────────────┘
                                    └─────────────┘
                                    Deployed di:
                                    - Railway
                                    - Render
                                    - VPS sendiri
```

---

## Setup Backend API (Node.js/Express)

### Inisialisasi Project
```bash
mkdir backend-api && cd backend-api
npm init -y
npm install express mongoose dotenv cors bcryptjs jsonwebtoken
npm install -D nodemon
```

### `.env` — Konfigurasi Rahasia
```env
MONGODB_URI=mongodb+srv://username:password@cluster0.xxxxx.mongodb.net/myapp?retryWrites=true&w=majority
PORT=3000
JWT_SECRET=rahasia_jwt_super_panjang_dan_aman_ganti_ini
```

### `server.js` — Entry Point
```javascript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
require('dotenv').config();

const app = express();

// Middleware
app.use(cors());
app.use(express.json());

// Routes
app.use('/api/auth',   require('./routes/auth'));
app.use('/api/produk', require('./routes/produk'));
app.use('/api/users',  require('./routes/users'));

// Health check
app.get('/', (req, res) => {
    res.json({ status: 'OK', message: 'API berjalan!' });
});

// Koneksi MongoDB
mongoose.connect(process.env.MONGODB_URI)
    .then(() => {
        console.log('✅ Terhubung ke MongoDB Atlas');
        app.listen(process.env.PORT || 3000, () => {
            console.log(`🚀 Server berjalan di port ${process.env.PORT}`);
        });
    })
    .catch(err => console.error('❌ Gagal konek MongoDB:', err));
```

### `models/Produk.js` — Schema MongoDB
```javascript
const mongoose = require('mongoose');

const produkSchema = new mongoose.Schema({
    nama: {
        type: String,
        required: [true, 'Nama produk wajib diisi'],
        trim: true,
        maxlength: 100
    },
    deskripsi: {
        type: String,
        default: ''
    },
    harga: {
        type: Number,
        required: true,
        min: [0, 'Harga tidak boleh negatif']
    },
    stok: {
        type: Number,
        default: 0,
        min: 0
    },
    kategori: {
        type: String,
        enum: ['elektronik', 'pakaian', 'makanan', 'lainnya'],
        default: 'lainnya'
    },
    gambarUrl: String,
    aktif: {
        type: Boolean,
        default: true
    }
}, {
    timestamps: true   // otomatis tambah createdAt & updatedAt
});

module.exports = mongoose.model('Produk', produkSchema);
```

### `models/User.js`
```javascript
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema({
    nama: { type: String, required: true },
    email: {
        type: String,
        required: true,
        unique: true,
        lowercase: true
    },
    password: { type: String, required: true, minlength: 6 },
    role: { type: String, enum: ['user', 'admin'], default: 'user' }
}, { timestamps: true });

// Hash password sebelum disimpan
userSchema.pre('save', async function(next) {
    if (!this.isModified('password')) return next();
    this.password = await bcrypt.hash(this.password, 12);
    next();
});

// Method untuk cek password
userSchema.methods.cocokPassword = function(password) {
    return bcrypt.compare(password, this.password);
};

module.exports = mongoose.model('User', userSchema);
```

### `middleware/auth.js` — JWT Middleware
```javascript
const jwt = require('jsonwebtoken');
const User = require('../models/User');

module.exports = async (req, res, next) => {
    try {
        const token = req.headers.authorization?.split(' ')[1]; // "Bearer <token>"
        if (!token) return res.status(401).json({ message: 'Token tidak ada' });

        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        req.user = await User.findById(decoded.id).select('-password');
        next();
    } catch (err) {
        res.status(401).json({ message: 'Token tidak valid' });
    }
};
```

### `routes/auth.js` — Endpoint Login & Register
```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const User = require('../models/User');
const router = express.Router();

const buatToken = (id) => jwt.sign({ id }, process.env.JWT_SECRET, { expiresIn: '7d' });

// POST /api/auth/register
router.post('/register', async (req, res) => {
    try {
        const { nama, email, password } = req.body;

        const sudahAda = await User.findOne({ email });
        if (sudahAda) return res.status(400).json({ message: 'Email sudah terdaftar' });

        const user = await User.create({ nama, email, password });
        const token = buatToken(user._id);

        res.status(201).json({
            success: true,
            token,
            user: { id: user._id, nama: user.nama, email: user.email }
        });
    } catch (err) {
        res.status(500).json({ message: err.message });
    }
});

// POST /api/auth/login
router.post('/login', async (req, res) => {
    try {
        const { email, password } = req.body;

        const user = await User.findOne({ email });
        if (!user) return res.status(401).json({ message: 'Email tidak ditemukan' });

        const cocok = await user.cocokPassword(password);
        if (!cocok) return res.status(401).json({ message: 'Password salah' });

        const token = buatToken(user._id);
        res.json({
            success: true,
            token,
            user: { id: user._id, nama: user.nama, email: user.email, role: user.role }
        });
    } catch (err) {
        res.status(500).json({ message: err.message });
    }
});

module.exports = router;
```

### `routes/produk.js` — CRUD Produk
```javascript
const express = require('express');
const Produk = require('../models/Produk');
const auth = require('../middleware/auth');
const router = express.Router();

// GET /api/produk — ambil semua produk (dengan filter & pagination)
router.get('/', async (req, res) => {
    try {
        const { kategori, cari, halaman = 1, limit = 10, minHarga, maxHarga } = req.query;

        const filter = { aktif: true };
        if (kategori) filter.kategori = kategori;
        if (cari) filter.nama = { $regex: cari, $options: 'i' };  // case-insensitive search
        if (minHarga || maxHarga) {
            filter.harga = {};
            if (minHarga) filter.harga.$gte = Number(minHarga);
            if (maxHarga) filter.harga.$lte = Number(maxHarga);
        }

        const total  = await Produk.countDocuments(filter);
        const produk = await Produk.find(filter)
            .sort({ createdAt: -1 })
            .skip((halaman - 1) * limit)
            .limit(Number(limit));

        res.json({
            success: true,
            data: produk,
            pagination: {
                total,
                halaman: Number(halaman),
                totalHalaman: Math.ceil(total / limit)
            }
        });
    } catch (err) {
        res.status(500).json({ message: err.message });
    }
});

// GET /api/produk/:id — ambil satu produk
router.get('/:id', async (req, res) => {
    try {
        const produk = await Produk.findById(req.params.id);
        if (!produk) return res.status(404).json({ message: 'Produk tidak ditemukan' });
        res.json({ success: true, data: produk });
    } catch (err) {
        res.status(500).json({ message: err.message });
    }
});

// POST /api/produk — tambah produk (perlu login)
router.post('/', auth, async (req, res) => {
    try {
        const produk = await Produk.create(req.body);
        res.status(201).json({ success: true, data: produk });
    } catch (err) {
        res.status(400).json({ message: err.message });
    }
});

// PUT /api/produk/:id — update produk
router.put('/:id', auth, async (req, res) => {
    try {
        const produk = await Produk.findByIdAndUpdate(
            req.params.id,
            req.body,
            { new: true, runValidators: true }   // return data baru + validasi
        );
        if (!produk) return res.status(404).json({ message: 'Tidak ditemukan' });
        res.json({ success: true, data: produk });
    } catch (err) {
        res.status(400).json({ message: err.message });
    }
});

// DELETE /api/produk/:id — hapus produk
router.delete('/:id', auth, async (req, res) => {
    try {
        // Soft delete — tandai tidak aktif daripada hapus beneran
        await Produk.findByIdAndUpdate(req.params.id, { aktif: false });
        res.json({ success: true, message: 'Produk dihapus' });
    } catch (err) {
        res.status(500).json({ message: err.message });
    }
});

module.exports = router;
```

---

## Koneksi dari Android dengan Retrofit

Retrofit adalah library HTTP client paling populer di Android.

### Tambah Dependency
```kotlin
// build.gradle (app)
dependencies {
    // Retrofit
    implementation("com.squareup.retrofit2:retrofit:2.11.0")
    implementation("com.squareup.retrofit2:converter-gson:2.11.0")

    // OkHttp (logging request/response)
    implementation("com.squareup.okhttp3:okhttp:4.12.0")
    implementation("com.squareup.okhttp3:logging-interceptor:4.12.0")

    // Coroutines
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.9.0")

    // ViewModel & LiveData
    implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.8.7")
    implementation("androidx.lifecycle:lifecycle-livedata-ktx:2.8.7")
}
```

```xml
<!-- AndroidManifest.xml — izin internet -->
<uses-permission android:name="android.permission.INTERNET" />
```

### Data Models (Kotlin)
```kotlin
// data/model/Produk.kt
data class Produk(
    @SerializedName("_id")         val id: String = "",
    @SerializedName("nama")        val nama: String = "",
    @SerializedName("deskripsi")   val deskripsi: String = "",
    @SerializedName("harga")       val harga: Double = 0.0,
    @SerializedName("stok")        val stok: Int = 0,
    @SerializedName("kategori")    val kategori: String = "",
    @SerializedName("gambarUrl")   val gambarUrl: String? = null,
    @SerializedName("createdAt")   val createdAt: String = ""
)

// Wrapper response API
data class ApiResponse<T>(
    @SerializedName("success") val success: Boolean,
    @SerializedName("data")    val data: T?,
    @SerializedName("message") val message: String? = null
)

data class PaginatedResponse<T>(
    @SerializedName("success")    val success: Boolean,
    @SerializedName("data")       val data: List<T>,
    @SerializedName("pagination") val pagination: Pagination
)

data class Pagination(
    @SerializedName("total")        val total: Int,
    @SerializedName("halaman")      val halaman: Int,
    @SerializedName("totalHalaman") val totalHalaman: Int
)

// Request body
data class LoginRequest(
    @SerializedName("email")    val email: String,
    @SerializedName("password") val password: String
)

data class RegisterRequest(
    @SerializedName("nama")     val nama: String,
    @SerializedName("email")    val email: String,
    @SerializedName("password") val password: String
)

data class AuthResponse(
    @SerializedName("success") val success: Boolean,
    @SerializedName("token")   val token: String,
    @SerializedName("user")    val user: UserInfo
)

data class UserInfo(
    @SerializedName("id")    val id: String,
    @SerializedName("nama")  val nama: String,
    @SerializedName("email") val email: String,
    @SerializedName("role")  val role: String = "user"
)
```

### API Interface
```kotlin
// network/ApiService.kt
interface ApiService {

    // Auth
    @POST("auth/login")
    suspend fun login(@Body request: LoginRequest): AuthResponse

    @POST("auth/register")
    suspend fun register(@Body request: RegisterRequest): AuthResponse

    // Produk
    @GET("produk")
    suspend fun getProduk(
        @Query("halaman")  halaman: Int = 1,
        @Query("limit")    limit: Int = 10,
        @Query("kategori") kategori: String? = null,
        @Query("cari")     cari: String? = null,
        @Query("minHarga") minHarga: Int? = null,
        @Query("maxHarga") maxHarga: Int? = null
    ): PaginatedResponse<Produk>

    @GET("produk/{id}")
    suspend fun getProdukById(@Path("id") id: String): ApiResponse<Produk>

    @POST("produk")
    suspend fun tambahProduk(
        @Header("Authorization") token: String,
        @Body produk: Produk
    ): ApiResponse<Produk>

    @PUT("produk/{id}")
    suspend fun updateProduk(
        @Header("Authorization") token: String,
        @Path("id") id: String,
        @Body produk: Produk
    ): ApiResponse<Produk>

    @DELETE("produk/{id}")
    suspend fun hapusProduk(
        @Header("Authorization") token: String,
        @Path("id") id: String
    ): ApiResponse<Unit>
}
```

### Retrofit Client
```kotlin
// network/RetrofitClient.kt
object RetrofitClient {

    // Ganti dengan URL backend yang sudah di-deploy
    private const val BASE_URL = "https://api-saya.railway.app/api/"

    // OkHttp client dengan logging
    private val httpClient: OkHttpClient by lazy {
        val logging = HttpLoggingInterceptor().apply {
            level = HttpLoggingInterceptor.Level.BODY   // log semua request & response
        }

        OkHttpClient.Builder()
            .addInterceptor(logging)
            .connectTimeout(30, TimeUnit.SECONDS)
            .readTimeout(30, TimeUnit.SECONDS)
            .build()
    }

    // Retrofit instance
    val instance: ApiService by lazy {
        Retrofit.Builder()
            .baseUrl(BASE_URL)
            .client(httpClient)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(ApiService::class.java)
    }
}
```

### Repository Pattern
```kotlin
// repository/ProdukRepository.kt
class ProdukRepository {

    private val api = RetrofitClient.instance

    // Sealed class untuk wrap hasil operasi
    sealed class Result<out T> {
        data class Success<T>(val data: T) : Result<T>()
        data class Error(val pesan: String) : Result<Nothing>()
        object Loading : Result<Nothing>()
    }

    suspend fun ambilSemuaProduk(
        halaman: Int = 1,
        kategori: String? = null,
        cari: String? = null
    ): Result<PaginatedResponse<Produk>> {
        return try {
            val response = api.getProduk(halaman = halaman, kategori = kategori, cari = cari)
            Result.Success(response)
        } catch (e: HttpException) {
            val pesan = when (e.code()) {
                401  -> "Sesi berakhir, silakan login ulang"
                403  -> "Tidak punya izin"
                404  -> "Data tidak ditemukan"
                500  -> "Server sedang bermasalah"
                else -> "Terjadi kesalahan (${e.code()})"
            }
            Result.Error(pesan)
        } catch (e: IOException) {
            Result.Error("Tidak ada koneksi internet")
        } catch (e: Exception) {
            Result.Error("Terjadi kesalahan: ${e.message}")
        }
    }

    suspend fun tambahProduk(token: String, produk: Produk): Result<Produk> {
        return try {
            val response = api.tambahProduk("Bearer $token", produk)
            if (response.success && response.data != null) {
                Result.Success(response.data)
            } else {
                Result.Error(response.message ?: "Gagal menyimpan produk")
            }
        } catch (e: IOException) {
            Result.Error("Tidak ada koneksi internet")
        } catch (e: Exception) {
            Result.Error(e.message ?: "Terjadi kesalahan")
        }
    }

    suspend fun hapusProduk(token: String, id: String): Result<Boolean> {
        return try {
            api.hapusProduk("Bearer $token", id)
            Result.Success(true)
        } catch (e: Exception) {
            Result.Error(e.message ?: "Gagal menghapus")
        }
    }
}
```

### ViewModel
```kotlin
// viewmodel/ProdukViewModel.kt
class ProdukViewModel : ViewModel() {

    private val repo = ProdukRepository()

    // State yang diobserve oleh UI
    val daftarProduk  = MutableLiveData<List<Produk>>()
    val isLoading     = MutableLiveData<Boolean>(false)
    val pesanError    = MutableLiveData<String?>()
    val totalHalaman  = MutableLiveData<Int>(1)

    private var halamanSaatIni = 1
    private var sedangMuat = false

    fun muatProduk(reset: Boolean = false, kategori: String? = null, cari: String? = null) {
        if (sedangMuat) return
        if (reset) halamanSaatIni = 1

        sedangMuat = true
        isLoading.value = true

        viewModelScope.launch {
            when (val result = repo.ambilSemuaProduk(halamanSaatIni, kategori, cari)) {
                is ProdukRepository.Result.Success -> {
                    val response = result.data
                    if (reset) {
                        daftarProduk.value = response.data
                    } else {
                        val currentList = daftarProduk.value?.toMutableList() ?: mutableListOf()
                        currentList.addAll(response.data)
                        daftarProduk.value = currentList
                    }
                    totalHalaman.value = response.pagination.totalHalaman
                    pesanError.value = null
                }
                is ProdukRepository.Result.Error -> {
                    pesanError.value = result.pesan
                }
                else -> {}
            }
            isLoading.value = false
            sedangMuat = false
        }
    }

    fun muatHalamanBerikutnya() {
        if (halamanSaatIni < (totalHalaman.value ?: 1)) {
            halamanSaatIni++
            muatProduk()
        }
    }

    fun tambahProduk(token: String, produk: Produk) {
        viewModelScope.launch {
            isLoading.value = true
            when (val result = repo.tambahProduk(token, produk)) {
                is ProdukRepository.Result.Success -> {
                    val currentList = daftarProduk.value?.toMutableList() ?: mutableListOf()
                    currentList.add(0, result.data)   // tambah di awal list
                    daftarProduk.value = currentList
                }
                is ProdukRepository.Result.Error -> {
                    pesanError.value = result.pesan
                }
                else -> {}
            }
            isLoading.value = false
        }
    }
}
```

---

## CRUD Lengkap

### Fragment List Produk
```kotlin
class ListProdukFragment : Fragment() {

    private var _binding: FragmentListProdukBinding? = null
    private val binding get() = _binding!!
    private val viewModel: ProdukViewModel by viewModels()
    private lateinit var adapter: ProdukAdapter

    override fun onCreateView(inflater: LayoutInflater, container: ViewGroup?, savedInstanceState: Bundle?): View {
        _binding = FragmentListProdukBinding.inflate(inflater, container, false)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        setupRecyclerView()
        setupObservers()
        setupSearch()
        setupSwipeRefresh()

        viewModel.muatProduk(reset = true)
    }

    private fun setupRecyclerView() {
        adapter = ProdukAdapter { produk ->
            // Navigasi ke detail
        }

        binding.rvProduk.apply {
            layoutManager = GridLayoutManager(requireContext(), 2)  // 2 kolom
            adapter = this@ListProdukFragment.adapter

            // Infinite scroll — muat halaman berikutnya saat scroll ke bawah
            addOnScrollListener(object : RecyclerView.OnScrollListener() {
                override fun onScrolled(rv: RecyclerView, dx: Int, dy: Int) {
                    val layoutManager = rv.layoutManager as GridLayoutManager
                    val total = layoutManager.itemCount
                    val terakhirTerlihat = layoutManager.findLastVisibleItemPosition()
                    if (terakhirTerlihat >= total - 3) {  // muat saat 3 item dari bawah
                        viewModel.muatHalamanBerikutnya()
                    }
                }
            })
        }
    }

    private fun setupObservers() {
        viewModel.daftarProduk.observe(viewLifecycleOwner) { produk ->
            adapter.submitList(produk)
            binding.tvKosong.isVisible = produk.isEmpty()
        }

        viewModel.isLoading.observe(viewLifecycleOwner) { loading ->
            binding.progressBar.isVisible = loading && adapter.itemCount == 0
            binding.swipeRefresh.isRefreshing = loading && adapter.itemCount > 0
        }

        viewModel.pesanError.observe(viewLifecycleOwner) { pesan ->
            pesan?.let {
                Snackbar.make(binding.root, it, Snackbar.LENGTH_LONG)
                    .setAction("Coba Lagi") { viewModel.muatProduk(reset = true) }
                    .show()
            }
        }
    }

    private fun setupSearch() {
        binding.etCari.addTextChangedListener { editable ->
            val keyword = editable.toString().trim()
            // Debounce — tunggu 500ms sebelum request
            binding.root.removeCallbacks(searchRunnable)
            searchRunnable = Runnable { viewModel.muatProduk(reset = true, cari = keyword) }
            binding.root.postDelayed(searchRunnable, 500)
        }
    }

    private var searchRunnable: Runnable = Runnable {}

    private fun setupSwipeRefresh() {
        binding.swipeRefresh.setOnRefreshListener {
            viewModel.muatProduk(reset = true)
        }
    }

    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null
    }
}
```

---

## Case: Aplikasi Katalog Produk

Aplikasi lengkap dengan autentikasi + CRUD produk dari MongoDB Atlas:

### Alur Aplikasi
```
SplashActivity
    ├── Ada token? ──► MainActivity (Fragment: Home, Kategori, Profil)
    └── Tidak? ──────► LoginActivity
                            └── Berhasil? ──► MainActivity
```

### Session Manager (Simpan Token)
```kotlin
// utils/SessionManager.kt
class SessionManager(context: Context) {

    private val prefs = context.getSharedPreferences("session", Context.MODE_PRIVATE)

    var token: String
        get() = prefs.getString("token", "") ?: ""
        set(value) = prefs.edit().putString("token", value).apply()

    var userId: String
        get() = prefs.getString("user_id", "") ?: ""
        set(value) = prefs.edit().putString("user_id", value).apply()

    var namaUser: String
        get() = prefs.getString("nama", "") ?: ""
        set(value) = prefs.edit().putString("nama", value).apply()

    val isLogin: Boolean get() = token.isNotEmpty()

    fun logout() = prefs.edit().clear().apply()
}
```

### LoginActivity lengkap dengan MongoDB
```kotlin
class LoginActivity : AppCompatActivity() {

    private lateinit var binding: ActivityLoginBinding
    private val session by lazy { SessionManager(this) }
    private val api = RetrofitClient.instance

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityLoginBinding.inflate(layoutInflater)
        setContentView(binding.root)

        if (session.isLogin) masukKeApp()

        binding.btnLogin.setOnClickListener { login() }
        binding.btnDaftar.setOnClickListener { daftar() }
    }

    private fun login() {
        val email    = binding.etEmail.text.toString().trim()
        val password = binding.etPassword.text.toString()

        if (!validasi(email, password)) return

        setLoading(true)
        lifecycleScope.launch {
            try {
                val response = api.login(LoginRequest(email, password))
                session.token    = response.token
                session.userId   = response.user.id
                session.namaUser = response.user.nama
                masukKeApp()
            } catch (e: HttpException) {
                val pesan = when (e.code()) {
                    401  -> "Email atau password salah"
                    else -> "Login gagal"
                }
                tampilkanError(pesan)
            } catch (e: IOException) {
                tampilkanError("Tidak ada koneksi internet")
            } finally {
                setLoading(false)
            }
        }
    }

    private fun daftar() {
        val nama     = binding.etNama.text.toString().trim()
        val email    = binding.etEmail.text.toString().trim()
        val password = binding.etPassword.text.toString()

        setLoading(true)
        lifecycleScope.launch {
            try {
                val response = api.register(RegisterRequest(nama, email, password))
                session.token    = response.token
                session.namaUser = response.user.nama
                masukKeApp()
            } catch (e: Exception) {
                tampilkanError("Gagal daftar: ${e.message}")
            } finally {
                setLoading(false)
            }
        }
    }

    private fun validasi(email: String, password: String): Boolean {
        if (email.isEmpty() || !android.util.Patterns.EMAIL_ADDRESS.matcher(email).matches()) {
            binding.tilEmail.error = "Email tidak valid"; return false
        }
        if (password.length < 6) {
            binding.tilPassword.error = "Password minimal 6 karakter"; return false
        }
        binding.tilEmail.error = null
        binding.tilPassword.error = null
        return true
    }

    private fun setLoading(show: Boolean) {
        binding.progressBar.isVisible = show
        binding.btnLogin.isEnabled    = !show
        binding.btnDaftar.isEnabled   = !show
    }

    private fun tampilkanError(pesan: String) {
        Snackbar.make(binding.root, pesan, Snackbar.LENGTH_LONG).show()
    }

    private fun masukKeApp() {
        startActivity(Intent(this, MainActivity::class.java))
        finish()
    }
}
```

---

## Ringkasan Perbandingan

| Aspek | Firebase | MongoDB Atlas |
|-------|----------|---------------|
| **Setup** | Mudah (file JSON) | Perlu backend sendiri |
| **Query** | Menengah | Sangat powerful |
| **Auth bawaan** | ✅ | ❌ (buat sendiri/JWT) |
| **Real-time** | ✅ Native | ❌ Perlu WebSocket |
| **Cocok untuk** | MVP, startup cepat | Enterprise, kontrol penuh |
| **Skill dibutuhkan** | Android saja | Android + Backend |
| **Biaya** | Free tier OK | Free tier OK (M0 Atlas) |

---

##  Checklist Sebelum Deploy

### Firebase
- [ ] `google-services.json` sudah di folder `app/`
- [ ] Firestore Rules sudah dikonfigurasi (bukan test mode)
- [ ] Auth method sudah diaktifkan
- [ ] SHA-1 fingerprint sudah didaftarkan (untuk Google Sign-In)

### MongoDB Atlas
- [ ] IP address backend sudah diwhitelist di Atlas
- [ ] Environment variables (`.env`) tidak ikut commit ke Git
- [ ] JWT secret panjang dan acak
- [ ] HTTPS digunakan di production (bukan HTTP)
- [ ] Token disimpan aman di SharedPreferences (bukan hardcode)

---
