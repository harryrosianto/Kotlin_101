---

## Daftar Isi

1. [Pengenalan Android & Kotlin](#1-pengenalan-android--kotlin)
2. [Struktur Project Android](#2-struktur-project-android)
3. [Namespace XML: `android:`, `app:`, `tools:`](#3-namespace-xml-android-app-tools)
4. [Tipe Data & Deklarasi Variabel](#4-tipe-data--deklarasi-variabel)
5. [Manipulasi Data](#5-manipulasi-data)
6. [Function (Fungsi)](#6-function-fungsi)
7. [Kotlin Flow Control](#7-kotlin-flow-control)
8. [Class & Object](#8-class--object)
9. [Activity](#9-activity)
10. [Layout XML & View Components](#10-layout-xml--view-components)
11. [Intent & Navigasi Antar Activity](#11-intent--navigasi-antar-activity)
12. [RecyclerView & Adapter](#12-recyclerview--adapter)
13. [SharedPreferences & Penyimpanan Data](#13-sharedpreferences--penyimpanan-data)
14. [Proyek Lengkap: Aplikasi To-Do List](#14-proyek-lengkap-aplikasi-to-do-list)

---

## 1. Pengenalan Android & Kotlin

### Apa itu Android?
Android adalah sistem operasi mobile berbasis Linux yang dikembangkan oleh Google. Aplikasi Android ditulis dalam **Kotlin** (resmi sejak 2019) atau Java.

### Mengapa Kotlin?
| Fitur | Kotlin | Java |
|-------|--------|------|
| Null Safety | ✅ Built-in | ❌ Manual |
| Sintaks | Lebih ringkas | Verbose |
| Coroutines | ✅ Native | ❌ Perlu library |
| Data Class | ✅ 1 baris | ❌ Ratusan baris |
| Interoperability | ✅ 100% Java | - |

### Tools yang Dibutuhkan
- **Android Studio** (IDE resmi) — [download](https://developer.android.com/studio)
- **JDK 17+**
- **Android SDK**
- **Emulator** atau perangkat Android fisik

### Membuat Project Baru
```
File → New → New Project → Empty Views Activity
  Name        : MyFirstApp
  Package     : com.example.myfirstapp
  Save Location: (pilih folder)
  Language    : Kotlin
  Min SDK     : API 24 (Android 7.0)
```

---

## 2. Struktur Project Android

Setelah project dibuat, inilah struktur folder yang akan kamu lihat:

```
MyFirstApp/
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/
│   │   │   │   └── com.example.myfirstapp/
│   │   │   │       └── MainActivity.kt        ← Kotlin code
│   │   │   ├── res/
│   │   │   │   ├── layout/
│   │   │   │   │   └── activity_main.xml      ← Layout utama
│   │   │   │   ├── drawable/                  ← Gambar & ikon
│   │   │   │   ├── values/
│   │   │   │   │   ├── colors.xml             ← Definisi warna
│   │   │   │   │   ├── strings.xml            ← Teks/string
│   │   │   │   │   └── themes.xml             ← Tema aplikasi
│   │   │   │   ├── mipmap/                    ← Ikon launcher
│   │   │   │   └── menu/                      ← Menu XML
│   │   │   └── AndroidManifest.xml            ← Konfigurasi app
│   ├── build.gradle (app)                     ← Dependency modul
│   └── proguard-rules.pro
├── build.gradle (project)                     ← Konfigurasi global
├── settings.gradle
└── gradle.properties
```

### AndroidManifest.xml — Jantung Aplikasi
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- Izin aplikasi -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.CAMERA" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/Theme.MyFirstApp">

        <!-- Activity harus didaftarkan di sini -->
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <!-- Tandai sebagai activity pembuka app -->
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity android:name=".SecondActivity" />

    </application>
</manifest>
```

### build.gradle (app) — Konfigurasi & Dependency
```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
}

android {
    namespace = "com.example.myfirstapp"
    compileSdk = 35

    defaultConfig {
        applicationId = "com.example.myfirstapp"
        minSdk = 24
        targetSdk = 35
        versionCode = 1
        versionName = "1.0"
    }

    buildTypes {
        release {
            isMinifyEnabled = false
        }
    }

    // Aktifkan View Binding (wajib untuk akses view tanpa findViewById)
    buildFeatures {
        viewBinding = true
    }
}

dependencies {
    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.appcompat)
    implementation(libs.material)
    implementation(libs.androidx.activity)
    implementation(libs.androidx.constraintlayout)

    // RecyclerView
    implementation("androidx.recyclerview:recyclerview:1.3.2")

    // Lifecycle (ViewModel & LiveData)
    implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.8.7")
    implementation("androidx.lifecycle:lifecycle-livedata-ktx:2.8.7")
}
```

---

## 3. Namespace XML: `android:`, `app:`, `tools:`

Di dalam file XML layout, kamu akan sering menemukan tiga prefix berbeda. Ini sangat penting untuk dipahami!

```xml
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
```

### `android:` — Atribut Standar Android
- Berasal dari **Android SDK** resmi
- Berlaku pada semua versi Android yang didukung
- Berdampak nyata saat aplikasi **dijalankan**

```xml
<!-- Contoh atribut android: -->
android:id="@+id/btnSubmit"
android:layout_width="match_parent"
android:layout_height="wrap_content"
android:text="Submit"
android:textColor="#FFFFFF"
android:background="#6200EE"
android:visibility="visible"
android:padding="16dp"
android:gravity="center"
android:onClick="onButtonClick"
```

### `app:` — Atribut Custom / Library
- Berasal dari **library pihak ketiga** atau komponen AndroidX
- Diperlukan karena namespace `android:` tidak mencakup atribut dari library
- Contoh: ConstraintLayout, MaterialDesign, NavigationComponent

```xml
<!-- app: digunakan untuk constraint di ConstraintLayout -->
app:layout_constraintTop_toTopOf="parent"
app:layout_constraintStart_toStartOf="parent"
app:layout_constraintEnd_toEndOf="parent"
app:layout_constraintBottom_toBottomOf="parent"

<!-- app: untuk MaterialButton -->
app:cornerRadius="8dp"
app:backgroundTint="#6200EE"

<!-- app: untuk BottomNavigationView -->
app:menu="@menu/bottom_nav_menu"

<!-- app: untuk Toolbar -->
app:title="My App"
app:subtitle="Halaman Utama"
```

### `tools:` — Atribut untuk Preview & Debugging
- Hanya aktif di **Android Studio Editor** (preview layout)
- **TIDAK berpengaruh** saat aplikasi dijalankan di device
- Berguna untuk melihat tampilan tanpa data sebenarnya

```xml
<!-- Tampilkan teks di preview, tapi tidak di runtime -->
tools:text="Sample Text"

<!-- Sembunyikan view di preview (tapi tetap ada di runtime) -->
tools:visibility="invisible"

<!-- Tentukan activity untuk konteks preview -->
tools:context=".MainActivity"

<!-- Contoh data dummy di RecyclerView -->
tools:listitem="@layout/item_todo"
tools:itemCount="5"

<!-- Ignore lint warning tertentu -->
tools:ignore="HardcodedText"

<!-- Lihat layout di layar tertentu -->
tools:deviceIds="pixel_5"
```

### Perbandingan Lengkap
| Prefix | Sumber | Berpengaruh di Runtime? | Contoh Penggunaan |
|--------|--------|------------------------|-------------------|
| `android:` | Android SDK | ✅ Ya | `android:text`, `android:id` |
| `app:` | Library/AndroidX | ✅ Ya | `app:layout_constraint*` |
| `tools:` | Android Studio | ❌ Tidak | `tools:text` (preview saja) |

---

## 4. Tipe Data & Deklarasi Variabel

### `val` vs `var`
```kotlin
// val = immutable (tidak bisa diubah setelah diisi) — seperti 'final' di Java
val namaAplikasi: String = "MyApp"
val versi: Int = 1

// var = mutable (bisa diubah)
var score: Int = 0
var namaUser: String = "Budi"

// Type inference — Kotlin bisa menebak tipe otomatis
val pi = 3.14          // Double
val huruf = 'A'        // Char
val benar = true       // Boolean
var counter = 0        // Int
```

### Tipe Data Primitif Kotlin
```kotlin
// Bilangan Bulat
val a: Byte    = 127              // 8-bit  (-128 to 127)
val b: Short   = 32767            // 16-bit (-32,768 to 32,767)
val c: Int     = 2_147_483_647    // 32-bit (bisa pakai _ sebagai pemisah ribuan)
val d: Long    = 9_223_372_036L   // 64-bit (tambah L di akhir)

// Bilangan Desimal
val e: Float   = 3.14f            // 32-bit (tambah f)
val f: Double  = 3.141592653589   // 64-bit (default)

// Lainnya
val g: Boolean = true             // true / false
val h: Char    = 'K'              // Satu karakter
val i: String  = "Hello Kotlin"   // Teks

// Konversi tipe
val x: Int = 42
val y: Long  = x.toLong()
val z: Double = x.toDouble()
val s: String = x.toString()
```

### Nullable vs Non-Null
Ini adalah fitur unggulan Kotlin untuk mencegah **NullPointerException**!
```kotlin
// Non-nullable — TIDAK BISA berisi null
var nama: String = "Budi"
// nama = null  ← ERROR! Tidak diperbolehkan

// Nullable — bisa berisi null (tambah tanda ?)
var email: String? = null
email = "budi@email.com"  // OK
email = null               // OK

// Safe call operator ?.
val panjang: Int? = email?.length  // null jika email null

// Elvis operator ?: — nilai default jika null
val panjangAman: Int = email?.length ?: 0

// Non-null assertion !! — paksa (berbahaya, gunakan hati-hati)
val panjangPaksa: Int = email!!.length  // Crash jika email null!

// let — eksekusi blok hanya jika tidak null
email?.let { e ->
    println("Email valid: $e")
}
```

### String & String Template
```kotlin
val nama  = "Budi"
val umur  = 25
val kota  = "Jakarta"

// String biasa
val sapa1 = "Halo, " + nama + "!"

// String template (lebih bersih, pakai $)
val sapa2 = "Halo, $nama! Umur kamu $umur tahun."

// Ekspresi dalam template (pakai ${})
val sapa3 = "Tahun depan kamu berumur ${umur + 1}."
val info  = "Nama: ${nama.uppercase()}, Kota: $kota"

// Multiline string (triple quote)
val pesan = """
    Halo $nama,
    Selamat datang di aplikasi kami.
    Kamu tinggal di $kota.
""".trimIndent()

// String methods
println(nama.length)          // 4
println(nama.uppercase())     // BUDI
println(nama.lowercase())     // budi
println(nama.reversed())      // iduB
println(nama.contains("ud")) // true
println("  spasi  ".trim())  // "spasi"
println("a,b,c".split(","))  // [a, b, c]
```

### Collections
```kotlin
// ── LIST ──────────────────────────────────────────
// Immutable List (read-only)
val buah: List<String> = listOf("Apel", "Mangga", "Jeruk")

// Mutable List (bisa diubah)
val sayur: MutableList<String> = mutableListOf("Bayam", "Kangkung")
sayur.add("Wortel")
sayur.remove("Bayam")
sayur[0] = "Brokoli"  // Update index ke-0

println(buah[0])        // Apel
println(buah.size)      // 3
println(buah.isEmpty()) // false

// ── SET ───────────────────────────────────────────
// Set = tidak ada duplikat
val angka: Set<Int> = setOf(1, 2, 3, 2, 1)  // {1, 2, 3}
val angkaMutable: MutableSet<Int> = mutableSetOf(10, 20, 30)
angkaMutable.add(40)

// ── MAP ───────────────────────────────────────────
// Map = pasangan key-value
val nilaiMahasiswa: Map<String, Int> = mapOf(
    "Budi"  to 85,
    "Ani"   to 92,
    "Citra" to 78
)

val nilaiMutable: MutableMap<String, Int> = mutableMapOf(
    "Budi" to 85
)
nilaiMutable["Ani"] = 92          // tambah/update
nilaiMutable.remove("Budi")       // hapus

println(nilaiMahasiswa["Ani"])    // 92
println(nilaiMahasiswa.keys)      // [Budi, Ani, Citra]
println(nilaiMahasiswa.values)    // [85, 92, 78]
```

### Array
```kotlin
// Array statis
val angka = arrayOf(1, 2, 3, 4, 5)
val nama  = arrayOf("Ani", "Budi", "Citra")

// Array dengan tipe spesifik (lebih efisien)
val intArray  = intArrayOf(1, 2, 3)
val boolArray = booleanArrayOf(true, false, true)

// Akses
println(angka[0])     // 1
angka[0] = 10         // ubah nilai
println(angka.size)   // 5

// Konversi ke List
val daftarAngka = angka.toList()
```

---

## 5. Manipulasi Data

### Operasi pada List
```kotlin
val angka = mutableListOf(5, 2, 8, 1, 9, 3)

// Sorting
angka.sort()                     // ascending: [1, 2, 3, 5, 8, 9]
angka.sortDescending()           // descending: [9, 8, 5, 3, 2, 1]

// Filter — ambil yang memenuhi kondisi
val genap = angka.filter { it % 2 == 0 }  // [8, 2]

// Map — transformasi setiap elemen
val dikali2 = angka.map { it * 2 }        // [10, 4, 16, 2, 18, 6]

// Find — cari elemen pertama
val lebihDari5 = angka.find { it > 5 }    // 8

// Any / All / None
println(angka.any { it > 7 })   // true (ada yang > 7)
println(angka.all { it > 0 })   // true (semua > 0)
println(angka.none { it > 10 }) // true (tidak ada yang > 10)

// Reduce — hitung akumulasi
val total = angka.reduce { acc, i -> acc + i }   // 28
val jumlah = angka.sum()                          // 28 (khusus angka)
val rerata = angka.average()                      // 4.666...

// GroupBy — kelompokkan berdasarkan kondisi
val mahasiswa = listOf("Ani", "Budi", "Anto", "Bety")
val perHuruf = mahasiswa.groupBy { it[0] }
// {A=[Ani, Anto], B=[Budi, Bety]}

// FlatMap — ratakan list bersarang
val nested = listOf(listOf(1, 2), listOf(3, 4))
val flat = nested.flatMap { it }  // [1, 2, 3, 4]

// Take / Drop
println(angka.take(3))    // 3 elemen pertama
println(angka.drop(3))    // hapus 3 elemen pertama
println(angka.takeLast(2)) // 2 elemen terakhir
```

### Destructuring
```kotlin
// Dengan Pair dan Triple
val pasangan = Pair("Jakarta", 10_000_000)
val (kota, populasi) = pasangan
println("$kota: $populasi")

val titik = Triple(10.5, -6.2, 0.0)
val (lat, lon, alt) = titik

// Dengan data class
data class Mahasiswa(val nama: String, val nilai: Int, val prodi: String)

val mhs = Mahasiswa("Budi", 90, "Informatika")
val (n, v, p) = mhs
println("$n - $v - $p")

// Dalam loop Map
val map = mapOf("a" to 1, "b" to 2)
for ((key, value) in map) {
    println("$key = $value")
}
```

---

## 6. Function (Fungsi)

### Sintaks Dasar
```kotlin
// Fungsi tanpa return value (Unit = void)
fun sapa(nama: String) {
    println("Halo, $nama!")
}

// Fungsi dengan return value
fun tambah(a: Int, b: Int): Int {
    return a + b
}

// Single expression function (tanpa kurung kurawal)
fun kali(a: Int, b: Int): Int = a * b
fun kuadrat(x: Int) = x * x  // tipe otomatis terdeteksi

// Memanggil fungsi
sapa("Budi")
val hasil = tambah(5, 3)
println(hasil)  // 8
```

### Parameter Default & Named Parameter
```kotlin
// Parameter dengan nilai default
fun buatProfil(
    nama: String,
    umur: Int = 0,
    kota: String = "Jakarta",
    aktif: Boolean = true
) {
    println("Nama: $nama, Umur: $umur, Kota: $kota, Aktif: $aktif")
}

// Panggil dengan berbagai cara
buatProfil("Budi")                         // pakai semua default
buatProfil("Ani", 25)                      // override umur saja
buatProfil("Citra", kota = "Bandung")      // named parameter — skip umur
buatProfil("Deni", 30, "Surabaya", false)  // semua diisi
```

### Vararg — Jumlah Parameter Tak Terbatas
```kotlin
fun jumlahkan(vararg angka: Int): Int {
    return angka.sum()
}

println(jumlahkan(1, 2, 3))           // 6
println(jumlahkan(10, 20, 30, 40))    // 100

// Spread operator — kirim array ke vararg
val arr = intArrayOf(1, 2, 3)
println(jumlahkan(*arr))              // 6
```

### Higher-Order Function (Fungsi sebagai Parameter)
```kotlin
// Fungsi yang menerima fungsi lain sebagai parameter
fun prosesAngka(angka: Int, operasi: (Int) -> Int): Int {
    return operasi(angka)
}

// Panggil dengan lambda
val dua = prosesAngka(5) { it * 2 }     // 10
val tiga = prosesAngka(5) { it * it }   // 25

// Fungsi yang return fungsi
fun buatPenambah(jumlah: Int): (Int) -> Int {
    return { angka -> angka + jumlah }
}

val tambah5 = buatPenambah(5)
println(tambah5(10))   // 15
println(tambah5(20))   // 25
```

### Lambda Expression
```kotlin
// Lambda = fungsi anonim
val salam: (String) -> String = { nama -> "Halo, $nama!" }
println(salam("Budi"))  // Halo, Budi!

// it — nama implisit jika hanya 1 parameter
val kaliDua: (Int) -> Int = { it * 2 }
println(kaliDua(7))  // 14

// Lambda multiline
val validasi: (String) -> Boolean = { teks ->
    val cukupPanjang = teks.length >= 8
    val adaHurufBesar = teks.any { it.isUpperCase() }
    cukupPanjang && adaHurufBesar
}
println(validasi("Password1"))  // true
println(validasi("pass"))       // false
```

### Extension Function
```kotlin
// Tambahkan fungsi ke class yang sudah ada tanpa mewarisinya
fun String.balik(): String = this.reversed()
fun String.kapital(): String = this.replaceFirstChar { it.uppercaseChar() }
fun Int.rupiah(): String = "Rp ${String.format("%,d", this)}"

println("kotlin".balik())   // niltok
println("budi".kapital())   // Budi
println(50000.rupiah())     // Rp 50,000

// Extension function di Android (sangat berguna)
fun View.tampilkan() { this.visibility = View.VISIBLE }
fun View.sembunyikan() { this.visibility = View.GONE }
fun Context.toast(pesan: String) {
    Toast.makeText(this, pesan, Toast.LENGTH_SHORT).show()
}
```

### Scope Functions: `let`, `run`, `with`, `apply`, `also`
```kotlin
data class User(var nama: String = "", var email: String = "", var umur: Int = 0)

// apply — konfigurasi objek, return objek itu sendiri
val user = User().apply {
    nama  = "Budi Santoso"
    email = "budi@example.com"
    umur  = 25
}

// with — jalankan blok pada objek, return hasil blok
val info = with(user) {
    "Nama: $nama, Email: $email"
}

// let — jalankan blok, return hasil blok (sering untuk null-check)
val panjangNama = user.nama?.let { nama ->
    println("Nama: $nama")
    nama.length
} ?: 0

// also — seperti apply tapi pakai 'it', return objek
val userBaru = User().also {
    it.nama = "Ani"
    println("User dibuat: ${it.nama}")
}

// run — kombinasi with + let
val hasilRun = user.run {
    copy(nama = nama.uppercase())
}
```

---

## 7. Kotlin Flow Control

### If Expression
```kotlin
val nilai = 75

// If biasa
if (nilai >= 80) {
    println("A")
} else if (nilai >= 70) {
    println("B")
} else {
    println("C")
}

// If sebagai expression (return nilai)
val grade = if (nilai >= 80) "A" else if (nilai >= 70) "B" else "C"

// Digunakan langsung dalam string template
println("Grade kamu: ${if (nilai >= 80) "A" else "B"}")
```

### When Expression (pengganti switch-case)
```kotlin
val hari = 3

// When dengan nilai
val namaHari = when (hari) {
    1    -> "Senin"
    2    -> "Selasa"
    3    -> "Rabu"
    4, 5 -> "Kamis atau Jumat"   // beberapa nilai
    else -> "Weekend"
}
println(namaHari)  // Rabu

// When tanpa argumen (seperti if-else)
val suhu = 35
when {
    suhu < 0   -> println("Beku")
    suhu < 20  -> println("Dingin")
    suhu < 30  -> println("Sejuk")
    suhu < 35  -> println("Panas")
    else       -> println("Sangat Panas")
}

// When dengan type check
fun cekTipe(obj: Any) {
    when (obj) {
        is Int    -> println("Angka: $obj")
        is String -> println("Teks: ${obj.uppercase()}")
        is Boolean -> println("Boolean: $obj")
        else      -> println("Tipe lain")
    }
}
```

### Loop
```kotlin
// For loop
for (i in 1..10) { print("$i ") }       // 1 2 3 4 ... 10
for (i in 1 until 10) { print("$i ") }  // 1 2 3 4 ... 9 (tidak termasuk 10)
for (i in 10 downTo 1) { print("$i ") } // 10 9 8 ... 1
for (i in 0..20 step 5) { print("$i ") } // 0 5 10 15 20

// Loop list dengan index
val buah = listOf("Apel", "Mangga", "Jeruk")
for (item in buah) {
    println(item)
}
for ((index, item) in buah.withIndex()) {
    println("$index: $item")
}

// While loop
var counter = 0
while (counter < 5) {
    print("$counter ")
    counter++
}

// do-while
do {
    println("Dijalankan minimal sekali")
    counter++
} while (counter < 3)

// break & continue
for (i in 1..10) {
    if (i == 5) continue  // skip 5
    if (i == 8) break     // berhenti di 8
    print("$i ")          // 1 2 3 4 6 7
}
```

---

## 8. Class & Object

### Class Biasa
```kotlin
class Mahasiswa {
    // Properties
    var nama: String = ""
    var nim: String = ""
    var ipk: Double = 0.0

    // Constructor sekunder
    constructor(nama: String, nim: String, ipk: Double) {
        this.nama = nama
        this.nim = nim
        this.ipk = ipk
    }

    // Method
    fun tampilkan() {
        println("[$nim] $nama — IPK: $ipk")
    }

    fun isLulus(): Boolean = ipk >= 2.0
}

val mhs = Mahasiswa("Budi", "123456", 3.5)
mhs.tampilkan()
println(mhs.isLulus())
```

### Primary Constructor & init
```kotlin
// Constructor langsung di header class (lebih idiomatik)
class Produk(
    val id: Int,
    var nama: String,
    var harga: Double,
    var stok: Int = 0   // nilai default
) {
    val kodeUnik: String

    // init block — dijalankan saat objek dibuat
    init {
        kodeUnik = "PRD-${id.toString().padStart(4, '0')}"
        println("Produk '$nama' berhasil dibuat dengan kode $kodeUnik")
    }

    fun tampilkan() = println("$kodeUnik | $nama | Rp$harga | Stok: $stok")
    fun kurangiStok(jumlah: Int) {
        require(jumlah <= stok) { "Stok tidak cukup!" }
        stok -= jumlah
    }
}

val p = Produk(1, "Laptop", 15_000_000.0, 10)
p.tampilkan()   // PRD-0001 | Laptop | Rp15000000.0 | Stok: 10
p.kurangiStok(3)
p.tampilkan()   // PRD-0001 | Laptop | Rp15000000.0 | Stok: 7
```

### Data Class — untuk Model Data
```kotlin
// data class otomatis generate: equals(), hashCode(), toString(), copy()
data class User(
    val id: Int,
    val nama: String,
    val email: String,
    val umur: Int = 0
)

val user1 = User(1, "Budi", "budi@gmail.com", 25)
val user2 = user1.copy(nama = "Ani", email = "ani@gmail.com")  // salin dengan perubahan

println(user1)          // User(id=1, nama=Budi, email=budi@gmail.com, umur=25)
println(user1 == user2) // false (berbeda isi)
println(user1.nama)     // Budi

// Data class biasa dipakai untuk response API, database entity, dll.
data class ApiResponse<T>(
    val success: Boolean,
    val message: String,
    val data: T? = null
)
```

### Inheritance (Pewarisan)
```kotlin
// open = bisa diwarisi
open class Hewan(val nama: String) {
    open fun suara(): String = "..."  // open = bisa di-override

    fun tidur() = println("$nama sedang tidur")
}

class Kucing(nama: String) : Hewan(nama) {
    override fun suara() = "Meow"
    fun mencakar() = println("$nama mencakar!")
}

class Anjing(nama: String, val ras: String) : Hewan(nama) {
    override fun suara() = "Woof"
}

val kucing = Kucing("Kitty")
println(kucing.suara())  // Meow
kucing.tidur()           // Kitty sedang tidur
kucing.mencakar()        // Kitty mencakar!
```

### Interface
```kotlin
interface Bisa Terbang {
    fun terbang()
    fun mendarat() = println("Mendarat...")  // bisa punya implementasi default
}

interface BisaBerenang {
    fun berenang()
}

class Bebek(val nama: String) : BisaTerbang, BisaBerenang {
    override fun terbang() = println("$nama terbang!")
    override fun berenang() = println("$nama berenang!")
}
```

### Object & Companion Object — Singleton
```kotlin
// object = singleton (hanya ada satu instance)
object DatabaseConfig {
    const val HOST = "localhost"
    const val PORT = 5432
    const val DB_NAME = "myapp"

    fun getUrl() = "jdbc:postgresql://$HOST:$PORT/$DB_NAME"
}

println(DatabaseConfig.getUrl())

// companion object = static member di Kotlin
class MathHelper {
    companion object {
        const val PI = 3.14159265358979

        fun luasLingkaran(r: Double) = PI * r * r
        fun kelilingLingkaran(r: Double) = 2 * PI * r
    }
}

println(MathHelper.PI)
println(MathHelper.luasLingkaran(7.0))
```

### Enum Class
```kotlin
enum class StatusPesanan {
    MENUNGGU, DIPROSES, DIKIRIM, SELESAI, DIBATALKAN;

    fun tampilkan(): String = when (this) {
        MENUNGGU    -> "⏳ Menunggu"
        DIPROSES    -> "🔄 Diproses"
        DIKIRIM     -> "🚚 Dikirim"
        SELESAI     -> "✅ Selesai"
        DIBATALKAN  -> "❌ Dibatalkan"
    }
}

var status = StatusPesanan.MENUNGGU
println(status.tampilkan())  // ⏳ Menunggu
status = StatusPesanan.DIKIRIM
println(status.tampilkan())  // 🚚 Dikirim
```

### Sealed Class — Tipe Data Terbatas
```kotlin
// Sealed class = semua subclass harus di file yang sama
// Sangat berguna untuk merepresentasikan state/result
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val pesan: String, val kode: Int = 0) : Result<Nothing>()
    object Loading : Result<Nothing>()
}

fun prosesResult(result: Result<String>) {
    when (result) {
        is Result.Success -> println("Data: ${result.data}")
        is Result.Error   -> println("Error ${result.kode}: ${result.pesan}")
        is Result.Loading -> println("Loading...")
    }
}

prosesResult(Result.Loading)
prosesResult(Result.Success("Data berhasil dimuat"))
prosesResult(Result.Error("Tidak ada koneksi", 503))
```

---

## 9. Activity

### Apa itu Activity?
Activity adalah satu **layar** di aplikasi Android. Setiap halaman yang user lihat biasanya adalah satu Activity.

### Lifecycle Activity (Siklus Hidup)
```
          ┌─────────────────────────────────────┐
          │                                     │
    ──► onCreate()      ◄── Activity dibuat    │
          │                                     │
    ──► onStart()       ◄── Activity terlihat  │
          │                                     │
    ──► onResume()      ◄── User bisa interact  │
          │                                     │
          │     ◄── USER MENGGUNAKAN APP ──►    │
          │                                     │
    ──► onPause()       ◄── Sebagian tertutup  │
          │                                     │
    ──► onStop()        ◄── Tidak terlihat     │
          │                                     │
    ──► onDestroy()     ◄── Activity dihancurkan│
          │                                     │
          └─────────────────────────────────────┘
           (onRestart() jika kembali dari onStop)
```

### Implementasi Lengkap MainActivity.kt
```kotlin
package com.example.myfirstapp

import android.os.Bundle
import android.util.Log
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity

class MainActivity : AppCompatActivity() {

    // Tag untuk logging
    private val TAG = "MainActivity"

    // Deklarasi view (akan diinisialisasi di onCreate)
    private lateinit var tvHasil: TextView
    private lateinit var etInput: EditText
    private lateinit var btnSubmit: Button

    // Variabel state
    private var counter = 0

    // ── LIFECYCLE ────────────────────────────────────────────────────────────

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Pasang layout XML ke activity ini
        setContentView(R.layout.activity_main)

        Log.d(TAG, "onCreate dipanggil")

        // Inisialisasi view (hubungkan dengan elemen XML)
        tvHasil  = findViewById(R.id.tvHasil)
        etInput  = findViewById(R.id.etInput)
        btnSubmit = findViewById(R.id.btnSubmit)

        // Pulihkan state jika activity dibuat ulang (rotasi layar, dll)
        savedInstanceState?.let {
            counter = it.getInt("counter", 0)
            tvHasil.text = it.getString("teksHasil", "")
        }

        // Set listener
        setupListeners()
    }

    override fun onStart() {
        super.onStart()
        Log.d(TAG, "onStart dipanggil — Activity terlihat")
    }

    override fun onResume() {
        super.onResume()
        Log.d(TAG, "onResume dipanggil — User bisa interact")
    }

    override fun onPause() {
        super.onPause()
        Log.d(TAG, "onPause — Activity kehilangan fokus")
    }

    override fun onStop() {
        super.onStop()
        Log.d(TAG, "onStop — Activity tidak terlihat")
    }

    override fun onDestroy() {
        super.onDestroy()
        Log.d(TAG, "onDestroy — Activity dihancurkan")
    }

    // Simpan state sebelum activity dihancurkan sementara (rotasi, dll)
    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.putInt("counter", counter)
        outState.putString("teksHasil", tvHasil.text.toString())
        Log.d(TAG, "State disimpan")
    }

    // ── SETUP ────────────────────────────────────────────────────────────────

    private fun setupListeners() {
        btnSubmit.setOnClickListener {
            val input = etInput.text.toString().trim()

            if (input.isEmpty()) {
                // Tampilkan pesan error
                etInput.error = "Input tidak boleh kosong!"
                return@setOnClickListener
            }

            counter++
            tampilkanHasil(input)
        }
    }

    // ── LOGIC ────────────────────────────────────────────────────────────────

    private fun tampilkanHasil(teks: String) {
        val hasil = "[$counter] Kamu memasukkan: $teks"
        tvHasil.text = hasil
        Toast.makeText(this, "Berhasil!", Toast.LENGTH_SHORT).show()
        etInput.text.clear()
    }
}
```

### View Binding — Cara Modern (Tanpa findViewById)
```kotlin
// 1. Aktifkan di build.gradle:
//    buildFeatures { viewBinding = true }

// 2. Android Studio otomatis buat class binding dari XML
//    activity_main.xml → ActivityMainBinding

class MainActivityWithBinding : AppCompatActivity() {

    // Deklarasi binding
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Inisialisasi binding
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)  // pasang root view

        // Akses view langsung dari binding (tanpa findViewById!)
        binding.tvHasil.text = "Selamat datang!"
        binding.etInput.hint = "Ketik sesuatu..."

        binding.btnSubmit.setOnClickListener {
            val teks = binding.etInput.text.toString()
            binding.tvHasil.text = "Input: $teks"
        }
    }
}
```

---

## 10. Layout XML & View Components

### ConstraintLayout — Layout Utama
```xml
<!-- res/layout/activity_main.xml -->
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp"
    tools:context=".MainActivity">

    <!-- TextView -->
    <TextView
        android:id="@+id/tvJudul"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="Aplikasi Pertamaku"
        android:textSize="24sp"
        android:textStyle="bold"
        android:textColor="@color/black"
        android:gravity="center"
        android:padding="8dp"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <!-- EditText -->
    <com.google.android.material.textfield.TextInputLayout
        android:id="@+id/tilInput"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        app:layout_constraintTop_toBottomOf="@id/tvJudul"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:hint="Masukkan nama Anda">

        <com.google.android.material.textfield.TextInputEditText
            android:id="@+id/etInput"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="textPersonName"
            android:maxLength="50" />
    </com.google.android.material.textfield.TextInputLayout>

    <!-- Button -->
    <com.google.android.material.button.MaterialButton
        android:id="@+id/btnSubmit"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:text="Kirim"
        android:textSize="16sp"
        app:cornerRadius="8dp"
        app:layout_constraintTop_toBottomOf="@id/tilInput"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <!-- TextView Hasil -->
    <TextView
        android:id="@+id/tvHasil"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        android:textSize="16sp"
        android:lineSpacingExtra="4dp"
        tools:text="Hasil akan muncul di sini"
        app:layout_constraintTop_toBottomOf="@id/btnSubmit"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

### LinearLayout — Susunan Vertikal/Horizontal
```xml
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"      <!-- atau horizontal -->
    android:gravity="center"
    android:padding="16dp">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Item 1"
        android:layout_margin="8dp" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Item 2"
        android:layout_weight="1"       <!-- bagi sisa ruang -->
        android:layout_margin="8dp" />

</LinearLayout>
```

### ScrollView
```xml
<ScrollView
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <!-- Konten panjang di sini -->

    </LinearLayout>
</ScrollView>
```

### Komponen View Umum
```xml
<!-- CheckBox -->
<CheckBox
    android:id="@+id/cbSetuju"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Saya setuju dengan syarat dan ketentuan"
    android:checked="false" />

<!-- RadioButton (dalam RadioGroup) -->
<RadioGroup
    android:id="@+id/rgJenisKelamin"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="horizontal">

    <RadioButton
        android:id="@+id/rbLaki"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Laki-laki" />

    <RadioButton
        android:id="@+id/rbPerempuan"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Perempuan" />
</RadioGroup>

<!-- Switch -->
<com.google.android.material.switchmaterial.SwitchMaterial
    android:id="@+id/switchNotif"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Aktifkan Notifikasi"
    android:checked="true" />

<!-- Spinner (Dropdown) -->
<Spinner
    android:id="@+id/spProvinsi"
    android:layout_width="match_parent"
    android:layout_height="wrap_content" />

<!-- ImageView -->
<ImageView
    android:id="@+id/ivFoto"
    android:layout_width="120dp"
    android:layout_height="120dp"
    android:scaleType="centerCrop"
    android:src="@drawable/ic_launcher_foreground"
    android:contentDescription="Foto profil" />

<!-- ProgressBar -->
<ProgressBar
    android:id="@+id/progressBar"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:visibility="gone" />

<!-- SeekBar (Slider) -->
<SeekBar
    android:id="@+id/seekBar"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:max="100"
    android:progress="50" />
```

### Handling Semua Komponen di Kotlin
```kotlin
// CheckBox
val cbSetuju = findViewById<CheckBox>(R.id.cbSetuju)
cbSetuju.setOnCheckedChangeListener { _, isChecked ->
    binding.btnSubmit.isEnabled = isChecked
}

// RadioGroup
val rgJenis = findViewById<RadioGroup>(R.id.rgJenisKelamin)
val rbLaki  = findViewById<RadioButton>(R.id.rbLaki)
rgJenis.setOnCheckedChangeListener { _, checkedId ->
    val jenisKelamin = when (checkedId) {
        R.id.rbLaki     -> "Laki-laki"
        R.id.rbPerempuan -> "Perempuan"
        else             -> "Tidak diketahui"
    }
    Toast.makeText(this, jenisKelamin, Toast.LENGTH_SHORT).show()
}
val terpilih = if (rbLaki.isChecked) "Laki-laki" else "Perempuan"

// Spinner
val spProvinsi = findViewById<Spinner>(R.id.spProvinsi)
val daftarProvinsi = arrayOf("DKI Jakarta", "Jawa Barat", "Jawa Tengah", "Jawa Timur")
val adapter = ArrayAdapter(this, android.R.layout.simple_spinner_item, daftarProvinsi)
adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item)
spProvinsi.adapter = adapter
spProvinsi.onItemSelectedListener = object : AdapterView.OnItemSelectedListener {
    override fun onItemSelected(parent: AdapterView<*>, view: View?, pos: Int, id: Long) {
        val dipilih = daftarProvinsi[pos]
        Toast.makeText(this@MainActivity, "Dipilih: $dipilih", Toast.LENGTH_SHORT).show()
    }
    override fun onNothingSelected(parent: AdapterView<*>) {}
}

// SeekBar
val seekBar = findViewById<SeekBar>(R.id.seekBar)
seekBar.setOnSeekBarChangeListener(object : SeekBar.OnSeekBarChangeListener {
    override fun onProgressChanged(sb: SeekBar, progress: Int, fromUser: Boolean) {
        binding.tvHasil.text = "Nilai: $progress"
    }
    override fun onStartTrackingTouch(sb: SeekBar) {}
    override fun onStopTrackingTouch(sb: SeekBar) {}
})
```

---

## 11. Intent & Navigasi Antar Activity

### Apa itu Intent?
Intent adalah mekanisme untuk komunikasi antar komponen Android (pindah Activity, buka kamera, kirim data, dll).

### Explicit Intent — Pindah ke Activity Tertentu
```kotlin
// Di MainActivity.kt — Kirim data ke SecondActivity
binding.btnLanjut.setOnClickListener {
    val nama = binding.etNama.text.toString()
    val umur = binding.etUmur.text.toString().toIntOrNull() ?: 0

    val intent = Intent(this, SecondActivity::class.java).apply {
        putExtra("NAMA", nama)
        putExtra("UMUR", umur)
        putExtra("IS_ADMIN", false)
        putStringArrayListExtra("HOBI", arrayListOf("Coding", "Gaming"))
    }
    startActivity(intent)
}
```

```kotlin
// Di SecondActivity.kt — Terima data dari MainActivity
class SecondActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_second)

        // Ambil data dari intent
        val nama    = intent.getStringExtra("NAMA") ?: "Tidak ada nama"
        val umur    = intent.getIntExtra("UMUR", 0)
        val isAdmin = intent.getBooleanExtra("IS_ADMIN", false)
        val hobi    = intent.getStringArrayListExtra("HOBI")

        binding.tvInfo.text = "Halo, $nama! Umur: $umur"
        binding.tvHobi.text = "Hobi: ${hobi?.joinToString(", ")}"

        // Tombol kembali
        binding.btnKembali.setOnClickListener {
            finish()  // tutup activity ini, kembali ke sebelumnya
        }
    }
}
```

### Kirim Data Kembali (startActivityForResult versi modern)
```kotlin
// MainActivity.kt
// 1. Daftarkan launcher untuk menerima hasil
private val launcherTambahData = registerForActivityResult(
    ActivityResultContracts.StartActivityForResult()
) { result ->
    if (result.resultCode == RESULT_OK) {
        val dataBaru = result.data?.getStringExtra("DATA_BARU")
        Toast.makeText(this, "Terima data: $dataBaru", Toast.LENGTH_SHORT).show()
    }
}

// 2. Panggil activity untuk isi data
binding.btnTambah.setOnClickListener {
    val intent = Intent(this, TambahDataActivity::class.java)
    launcherTambahData.launch(intent)
}
```

```kotlin
// TambahDataActivity.kt
binding.btnSimpan.setOnClickListener {
    val dataBaru = binding.etData.text.toString()

    val resultIntent = Intent().apply {
        putExtra("DATA_BARU", dataBaru)
    }
    setResult(RESULT_OK, resultIntent)  // set hasil
    finish()                             // tutup activity
}
```

### Implicit Intent — Buka App Lain
```kotlin
// Buka browser
val urlIntent = Intent(Intent.ACTION_VIEW, Uri.parse("https://google.com"))
startActivity(urlIntent)

// Kirim email
val emailIntent = Intent(Intent.ACTION_SENDTO).apply {
    data = Uri.parse("mailto:tujuan@email.com")
    putExtra(Intent.EXTRA_SUBJECT, "Subjek Email")
    putExtra(Intent.EXTRA_TEXT, "Isi email...")
}
startActivity(Intent.createChooser(emailIntent, "Kirim email dengan:"))

// Buat panggilan telepon
val telIntent = Intent(Intent.ACTION_DIAL, Uri.parse("tel:+6281234567890"))
startActivity(telIntent)

// Bagikan teks
val shareIntent = Intent(Intent.ACTION_SEND).apply {
    type = "text/plain"
    putExtra(Intent.EXTRA_TEXT, "Halo dari aplikasiku!")
}
startActivity(Intent.createChooser(shareIntent, "Bagikan melalui:"))
```

---

## 12. RecyclerView & Adapter

RecyclerView adalah komponen untuk menampilkan daftar data secara efisien.

### Data Model
```kotlin
// data/model/Todo.kt
data class Todo(
    val id: Int,
    var judul: String,
    var selesai: Boolean = false,
    val tanggal: String = ""
)
```

### Item Layout XML
```xml
<!-- res/layout/item_todo.xml -->
<?xml version="1.0" encoding="utf-8"?>
<com.google.android.material.card.MaterialCardView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="4dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="12dp"
        android:gravity="center_vertical">

        <CheckBox
            android:id="@+id/cbSelesai"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content" />

        <LinearLayout
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:orientation="vertical"
            android:layout_marginStart="8dp">

            <TextView
                android:id="@+id/tvJudul"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:textSize="16sp"
                android:textStyle="bold"
                android:textColor="@color/black" />

            <TextView
                android:id="@+id/tvTanggal"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:textSize="12sp"
                android:textColor="#888888" />
        </LinearLayout>

        <ImageButton
            android:id="@+id/btnHapus"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@android:drawable/ic_delete"
            android:background="?attr/selectableItemBackgroundBorderless"
            android:contentDescription="Hapus" />

    </LinearLayout>
</com.google.android.material.card.MaterialCardView>
```

### Adapter
```kotlin
// adapter/TodoAdapter.kt
class TodoAdapter(
    private val daftarTodo: MutableList<Todo>,
    private val onChecked: (Todo, Boolean) -> Unit,
    private val onHapus: (Todo, Int) -> Unit
) : RecyclerView.Adapter<TodoAdapter.TodoViewHolder>() {

    // ViewHolder — representasi satu item di list
    inner class TodoViewHolder(
        private val binding: ItemTodoBinding
    ) : RecyclerView.ViewHolder(binding.root) {

        fun bind(todo: Todo, position: Int) {
            binding.tvJudul.text   = todo.judul
            binding.tvTanggal.text = todo.tanggal
            binding.cbSelesai.isChecked = todo.selesai

            // Coret teks jika selesai
            binding.tvJudul.paintFlags = if (todo.selesai) {
                binding.tvJudul.paintFlags or Paint.STRIKE_THRU_TEXT_FLAG
            } else {
                binding.tvJudul.paintFlags and Paint.STRIKE_THRU_TEXT_FLAG.inv()
            }

            // Event listener
            binding.cbSelesai.setOnCheckedChangeListener { _, isChecked ->
                onChecked(todo, isChecked)
            }
            binding.btnHapus.setOnClickListener {
                onHapus(todo, position)
            }
        }
    }

    // Wajib di-override: inflate layout item
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): TodoViewHolder {
        val binding = ItemTodoBinding.inflate(
            LayoutInflater.from(parent.context), parent, false
        )
        return TodoViewHolder(binding)
    }

    // Wajib di-override: bind data ke ViewHolder
    override fun onBindViewHolder(holder: TodoViewHolder, position: Int) {
        holder.bind(daftarTodo[position], position)
    }

    // Wajib di-override: jumlah item
    override fun getItemCount() = daftarTodo.size

    // Helper functions
    fun tambahItem(todo: Todo) {
        daftarTodo.add(todo)
        notifyItemInserted(daftarTodo.size - 1)
    }

    fun hapusItem(position: Int) {
        daftarTodo.removeAt(position)
        notifyItemRemoved(position)
        notifyItemRangeChanged(position, daftarTodo.size)
    }
}
```

### Setup RecyclerView di Activity
```kotlin
// Di activity / fragment:
private lateinit var adapter: TodoAdapter
private val daftarTodo = mutableListOf<Todo>()

private fun setupRecyclerView() {
    adapter = TodoAdapter(
        daftarTodo = daftarTodo,
        onChecked = { todo, isChecked ->
            todo.selesai = isChecked
            // Simpan perubahan ke database/preferences
        },
        onHapus = { todo, position ->
            // Tampilkan konfirmasi sebelum hapus
            MaterialAlertDialogBuilder(this)
                .setTitle("Hapus Todo")
                .setMessage("Yakin ingin menghapus \"${todo.judul}\"?")
                .setPositiveButton("Hapus") { _, _ ->
                    adapter.hapusItem(position)
                    Toast.makeText(this, "Todo dihapus", Toast.LENGTH_SHORT).show()
                }
                .setNegativeButton("Batal", null)
                .show()
        }
    )

    binding.rvTodo.apply {
        layoutManager = LinearLayoutManager(this@MainActivity)
        adapter = this@MainActivity.adapter
        // Tambahkan divider (garis pemisah)
        addItemDecoration(
            DividerItemDecoration(this@MainActivity, DividerItemDecoration.VERTICAL)
        )
    }
}
```

---

## 13. SharedPreferences & Penyimpanan Data

### SharedPreferences — Simpan Data Sederhana
```kotlin
// Simpan data
fun simpanPreferensi(context: Context) {
    val prefs = context.getSharedPreferences("MyAppPrefs", Context.MODE_PRIVATE)
    prefs.edit().apply {
        putString("nama_user", "Budi Santoso")
        putInt("umur", 25)
        putBoolean("sudah_login", true)
        putFloat("nilai", 95.5f)
        apply()  // async — gunakan commit() untuk sync
    }
}

// Ambil data
fun ambilPreferensi(context: Context) {
    val prefs = context.getSharedPreferences("MyAppPrefs", Context.MODE_PRIVATE)
    val nama       = prefs.getString("nama_user", "Tamu")   // default: "Tamu"
    val umur       = prefs.getInt("umur", 0)
    val sudahLogin = prefs.getBoolean("sudah_login", false)
    val nilai      = prefs.getFloat("nilai", 0f)

    println("Nama: $nama, Umur: $umur, Login: $sudahLogin")
}

// Hapus data
fun hapusPreferensi(context: Context) {
    val prefs = context.getSharedPreferences("MyAppPrefs", Context.MODE_PRIVATE)
    prefs.edit().apply {
        remove("nama_user")    // hapus satu key
        // atau
        clear()                // hapus semua
        apply()
    }
}
```

### Buat Helper Class untuk SharedPreferences
```kotlin
// utils/PrefHelper.kt
class PrefHelper(context: Context) {

    private val prefs = context.getSharedPreferences("MyAppPrefs", Context.MODE_PRIVATE)

    // Keys
    companion object {
        const val KEY_NAMA    = "nama"
        const val KEY_TOKEN   = "token"
        const val KEY_LOGIN   = "is_login"
        const val KEY_TEMA    = "tema_gelap"
    }

    // Properti dengan getter & setter
    var nama: String
        get() = prefs.getString(KEY_NAMA, "") ?: ""
        set(value) = prefs.edit().putString(KEY_NAMA, value).apply()

    var token: String
        get() = prefs.getString(KEY_TOKEN, "") ?: ""
        set(value) = prefs.edit().putString(KEY_TOKEN, value).apply()

    var isLogin: Boolean
        get() = prefs.getBoolean(KEY_LOGIN, false)
        set(value) = prefs.edit().putBoolean(KEY_LOGIN, value).apply()

    var temaGelap: Boolean
        get() = prefs.getBoolean(KEY_TEMA, false)
        set(value) = prefs.edit().putBoolean(KEY_TEMA, value).apply()

    fun logout() {
        prefs.edit().clear().apply()
    }
}

// Cara pakai:
val pref = PrefHelper(this)
pref.nama = "Budi Santoso"
pref.isLogin = true
println(pref.nama)     // Budi Santoso
println(pref.isLogin)  // true
pref.logout()          // hapus semua
```

---

## 14. Proyek Lengkap: Aplikasi To-Do List

Mari kita rangkum semua materi dalam satu proyek yang bisa langsung dijalankan!

### Struktur File
```
app/src/main/
├── java/com/example/todoapp/
│   ├── MainActivity.kt
│   ├── TambahTodoActivity.kt
│   ├── model/
│   │   └── Todo.kt
│   ├── adapter/
│   │   └── TodoAdapter.kt
│   └── utils/
│       └── PrefHelper.kt
└── res/
    ├── layout/
    │   ├── activity_main.xml
    │   ├── activity_tambah_todo.xml
    │   └── item_todo.xml
    └── values/
        ├── strings.xml
        └── colors.xml
```

### Model: `Todo.kt`
```kotlin
package com.example.todoapp.model

data class Todo(
    val id: Int,
    var judul: String,
    var deskripsi: String = "",
    var selesai: Boolean = false,
    val tanggal: String = ""
)
```

### Layout Utama: `activity_main.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <com.google.android.material.appbar.AppBarLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <com.google.android.material.appbar.MaterialToolbar
            android:id="@+id/toolbar"
            android:layout_width="match_parent"
            android:layout_height="?attr/actionBarSize"
            app:title="📝 To-Do List"
            app:titleTextColor="@android:color/white" />
    </com.google.android.material.appbar.AppBarLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        app:layout_behavior="@string/appbar_scrolling_view_behavior">

        <!-- Info jumlah todo -->
        <TextView
            android:id="@+id/tvInfo"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:padding="16dp"
            android:textSize="14sp"
            android:textColor="#666666"
            tools:text="3 tugas, 1 selesai" />

        <!-- RecyclerView -->
        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/rvTodo"
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1"
            android:clipToPadding="false"
            android:paddingBottom="80dp"
            tools:listitem="@layout/item_todo"
            tools:itemCount="5" />
    </LinearLayout>

    <!-- FAB untuk tambah todo baru -->
    <com.google.android.material.floatingactionbutton.FloatingActionButton
        android:id="@+id/fabTambah"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="bottom|end"
        android:layout_margin="16dp"
        android:src="@android:drawable/ic_input_add"
        android:contentDescription="Tambah Todo"
        app:tint="@android:color/white" />

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

### Layout Tambah Todo: `activity_tambah_todo.xml`
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp"
    tools:context=".TambahTodoActivity">

    <com.google.android.material.textfield.TextInputLayout
        android:id="@+id/tilJudul"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        app:hint="Judul tugas *"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent">

        <com.google.android.material.textfield.TextInputEditText
            android:id="@+id/etJudul"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="textCapSentences"
            android:maxLength="100"
            android:imeOptions="actionNext" />
    </com.google.android.material.textfield.TextInputLayout>

    <com.google.android.material.textfield.TextInputLayout
        android:id="@+id/tilDeskripsi"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        app:hint="Deskripsi (opsional)"
        app:layout_constraintTop_toBottomOf="@id/tilJudul"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent">

        <com.google.android.material.textfield.TextInputEditText
            android:id="@+id/etDeskripsi"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:inputType="textMultiLine"
            android:minLines="3"
            android:gravity="top"
            android:maxLength="500" />
    </com.google.android.material.textfield.TextInputLayout>

    <com.google.android.material.button.MaterialButton
        android:id="@+id/btnSimpan"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginTop="24dp"
        android:text="💾 Simpan Tugas"
        android:textSize="16sp"
        android:padding="12dp"
        app:cornerRadius="8dp"
        app:layout_constraintTop_toBottomOf="@id/tilDeskripsi"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

### MainActivity.kt (Lengkap)
```kotlin
package com.example.todoapp

import android.content.Intent
import android.os.Bundle
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.recyclerview.widget.LinearLayoutManager
import com.example.todoapp.adapter.TodoAdapter
import com.example.todoapp.databinding.ActivityMainBinding
import com.example.todoapp.model.Todo
import com.google.android.material.dialog.MaterialAlertDialogBuilder
import java.text.SimpleDateFormat
import java.util.*

class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding
    private lateinit var adapter: TodoAdapter
    private val daftarTodo = mutableListOf<Todo>()
    private var nextId = 1

    private val launcherTambah = registerForActivityResult(
        androidx.activity.result.contract.ActivityResultContracts.StartActivityForResult()
    ) { result ->
        if (result.resultCode == RESULT_OK) {
            val judul  = result.data?.getStringExtra("JUDUL") ?: return@registerForActivityResult
            val deskr  = result.data?.getStringExtra("DESKRIPSI") ?: ""
            tambahTodo(judul, deskr)
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        setSupportActionBar(binding.toolbar)
        setupRecyclerView()
        setupFab()
        muatDataContoh()
    }

    private fun setupRecyclerView() {
        adapter = TodoAdapter(
            daftarTodo = daftarTodo,
            onChecked = { todo, isChecked ->
                todo.selesai = isChecked
                perbaruiInfo()
                val pesan = if (isChecked) "✅ Tugas selesai!" else "↩️ Tandai belum selesai"
                Toast.makeText(this, pesan, Toast.LENGTH_SHORT).show()
            },
            onHapus = { _, position ->
                konfirmasiHapus(position)
            }
        )

        binding.rvTodo.apply {
            layoutManager = LinearLayoutManager(this@MainActivity)
            adapter = this@MainActivity.adapter
        }
    }

    private fun setupFab() {
        binding.fabTambah.setOnClickListener {
            val intent = Intent(this, TambahTodoActivity::class.java)
            launcherTambah.launch(intent)
        }
    }

    private fun tambahTodo(judul: String, deskripsi: String) {
        val tanggal = SimpleDateFormat("dd MMM yyyy, HH:mm", Locale("id")).format(Date())
        val todo = Todo(
            id = nextId++,
            judul = judul,
            deskripsi = deskripsi,
            tanggal = tanggal
        )
        adapter.tambahItem(todo)
        perbaruiInfo()
        binding.rvTodo.smoothScrollToPosition(daftarTodo.size - 1)
    }

    private fun konfirmasiHapus(position: Int) {
        MaterialAlertDialogBuilder(this)
            .setTitle("🗑️ Hapus Tugas")
            .setMessage("Yakin ingin menghapus tugas ini?")
            .setPositiveButton("Hapus") { _, _ ->
                adapter.hapusItem(position)
                perbaruiInfo()
                Toast.makeText(this, "Tugas dihapus", Toast.LENGTH_SHORT).show()
            }
            .setNegativeButton("Batal", null)
            .show()
    }

    private fun perbaruiInfo() {
        val total   = daftarTodo.size
        val selesai = daftarTodo.count { it.selesai }
        binding.tvInfo.text = "$total tugas, $selesai selesai"
    }

    private fun muatDataContoh() {
        listOf(
            "Belajar Kotlin" to "Pelajari syntax dasar dan fitur-fitur Kotlin",
            "Buat project Android" to "Mulai dari membuat project baru di Android Studio",
            "Latihan RecyclerView" to "Implementasikan RecyclerView dengan adapter"
        ).forEach { (judul, deskr) ->
            tambahTodo(judul, deskr)
        }
    }
}
```

### TambahTodoActivity.kt
```kotlin
package com.example.todoapp

import android.content.Intent
import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import com.example.todoapp.databinding.ActivityTambahTodoBinding

class TambahTodoActivity : AppCompatActivity() {

    private lateinit var binding: ActivityTambahTodoBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityTambahTodoBinding.inflate(layoutInflater)
        setContentView(binding.root)

        supportActionBar?.apply {
            title = "Tambah Tugas Baru"
            setDisplayHomeAsUpEnabled(true)   // tombol kembali
        }

        binding.btnSimpan.setOnClickListener {
            simpanTodo()
        }
    }

    private fun simpanTodo() {
        val judul    = binding.etJudul.text.toString().trim()
        val deskripsi = binding.etDeskripsi.text.toString().trim()

        // Validasi
        if (judul.isEmpty()) {
            binding.tilJudul.error = "Judul tidak boleh kosong!"
            return
        }
        binding.tilJudul.error = null

        // Kirim data kembali ke MainActivity
        val resultIntent = Intent().apply {
            putExtra("JUDUL", judul)
            putExtra("DESKRIPSI", deskripsi)
        }
        setResult(RESULT_OK, resultIntent)
        finish()
    }

    // Tangani tombol kembali di toolbar
    override fun onSupportNavigateUp(): Boolean {
        onBackPressedDispatcher.onBackPressed()
        return true
    }
}
```

---

## 📌 Cheat Sheet Cepat

### Ukuran Unit di Android XML
| Unit | Kegunaan | Keterangan |
|------|----------|------------|
| `dp` | Margin, padding, ukuran view | Density-independent pixels |
| `sp` | Ukuran teks | Scale-independent pixels |
| `px` | Hindari! | Berbeda di tiap device |

### Nilai `layout_width` / `layout_height`
| Nilai | Artinya |
|-------|---------|
| `match_parent` | Selebar/setinggi parent |
| `wrap_content` | Sesuai isi konten |
| `0dp` | Di ConstraintLayout: isi sisa ruang (chain) |
| `120dp` | Ukuran eksplisit |

### Gravity vs Layout_gravity
```xml
android:gravity="center"         <!-- Posisi KONTEN DI DALAM view -->
android:layout_gravity="center"  <!-- Posisi VIEW DI DALAM parent-nya -->
```

### Visibility
```xml
android:visibility="visible"    <!-- Terlihat (default) -->
android:visibility="invisible"  <!-- Tidak terlihat, tapi TETAP ada (space reserved) -->
android:visibility="gone"       <!-- Tidak terlihat, TIDAK ada space -->
```

Di Kotlin:
```kotlin
view.visibility = View.VISIBLE
view.visibility = View.INVISIBLE
view.visibility = View.GONE

// Cara lebih bersih dengan extension function
view.isVisible   = true
view.isInvisible = true
view.isGone      = true
```

---

## 🔗 Referensi & Resources

| Resource | Link |
|----------|------|
| Dokumentasi Android | [developer.android.com](https://developer.android.com/docs) |
| Kotlin Docs | [kotlinlang.org](https://kotlinlang.org/docs) |
| Material Design | [m3.material.io](https://m3.material.io) |
| Android Codelabs | [codelabs.developers.google.com](https://codelabs.developers.google.com/?cat=Android) |
| Jetpack Components | [developer.android.com/jetpack](https://developer.android.com/jetpack) |

---

> 💡 **Tips Belajar:** Praktikkan setiap konsep langsung di Android Studio. Buat project kecil untuk setiap topik — jangan hanya membaca!

**Selamat belajar! 🚀**
