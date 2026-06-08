# Materi RecyclerView Kotlin — Android Studio
## Studi Kasus: Aplikasi Todo List

---

## Daftar Isi
1. [Apa itu RecyclerView?](#1-apa-itu-recyclerview)
2. [Komponen Utama RecyclerView](#2-komponen-utama-recyclerview)
3. [Persiapan Project](#3-persiapan-project)
4. [Struktur Folder Project](#4-struktur-folder-project)
5. [Step 1 — Tambah Dependency](#step-1--tambah-dependency)
6. [Step 2 — Buat Data Class](#step-2--buat-data-class-model)
7. [Step 3 — Buat Layout Item](#step-3--buat-layout-item-recyclerview)
8. [Step 4 — Buat Layout Activity](#step-4--buat-layout-activity-main)
9. [Step 5 — Buat Adapter](#step-5--buat-adapter)
10. [Step 6 — Setup di Activity](#step-6--setup-di-mainactivity)
11. [Step 7 — Tambah Fitur Hapus Item](#step-7--tambah-fitur-hapus-item-bonus)
12. [Hasil Akhir & Penjelasan Alur](#hasil-akhir--penjelasan-alur)

---

## 1. Apa itu RecyclerView?

`RecyclerView` adalah komponen UI di Android yang digunakan untuk menampilkan **daftar data dalam jumlah besar secara efisien**. 

Berbeda dengan `ListView` (pendahulunya), RecyclerView hanya membuat View untuk item yang **terlihat di layar**, lalu **mendaur ulang (recycle)** View tersebut saat discroll — sehingga penggunaan memori jauh lebih hemat.

---

## 2. Komponen Utama RecyclerView

RecyclerView tidak bisa bekerja sendiri. Ia membutuhkan **5 komponen** yang bekerja sama. Pahami dulu peran masing-masing sebelum mulai coding.

---

### 2.1 RecyclerView (Widget)

`RecyclerView` adalah **komponen UI** yang ditempatkan di dalam file layout XML. Ia bertindak sebagai **wadah/container** yang menampilkan item-item secara berulang.

RecyclerView sendiri **tidak tahu** data apa yang harus ditampilkan — ia bergantung sepenuhnya pada Adapter dan LayoutManager.

```xml
<!-- Ditempatkan di activity_main.xml -->
<androidx.recyclerview.widget.RecyclerView
    android:id="@+id/recyclerView"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

---

### 2.2 Data Class / Model

Data Class adalah **struktur data** yang merepresentasikan satu item dalam list. Misalnya, satu item todo, satu kontak, atau satu produk.

Data class adalah **sumber kebenaran (source of truth)** — semua yang ditampilkan di layar berasal dari sini.

```kotlin
// Satu "TodoItem" merepresentasikan satu baris di list
data class TodoItem(
    val id: Int,
    val title: String,
    var isDone: Boolean = false
)
```

Biasanya data disimpan dalam bentuk **List** atau **MutableList**:
```kotlin
val todoList = mutableListOf<TodoItem>()  // Kumpulan data yang akan ditampilkan
```

---

### 2.3 ViewHolder

`ViewHolder` adalah class yang bertugas **menyimpan referensi** ke setiap widget (TextView, ImageView, dll.) dalam satu item layout.

**Mengapa perlu ViewHolder?**

Tanpa ViewHolder, setiap kali item ditampilkan, Android harus memanggil `findViewById()` untuk mencari widget di dalam layout — ini lambat. Dengan ViewHolder, pencarian hanya dilakukan **sekali saja** saat View pertama kali dibuat, lalu referensinya disimpan dan digunakan ulang.

```
Tanpa ViewHolder:  scroll 100 item = 100x findViewById() per widget ← LAMBAT
Dengan ViewHolder: scroll 100 item = hanya ~5x findViewById() total  ← CEPAT
```

```kotlin
// ViewHolder menyimpan referensi semua widget dari item_todo.xml
class TodoViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
    val tvTitle: TextView    = itemView.findViewById(R.id.tvTitle)
    val checkboxDone: CheckBox = itemView.findViewById(R.id.checkboxDone)
    val btnDelete: ImageButton = itemView.findViewById(R.id.btnDelete)
    // Setelah ini, cukup akses holder.tvTitle, holder.checkboxDone, dst.
}
```

---

### 2.4 Adapter

`Adapter` adalah **jembatan** antara data (List) dan tampilan (RecyclerView). Ia tahu data apa yang ada, dan tahu bagaimana cara menampilkannya di layar.

Adapter wajib mengimplementasikan **3 fungsi utama**:

| Fungsi | Kapan dipanggil | Tugas |
|---|---|---|
| `onCreateViewHolder()` | Saat View baru perlu dibuat | Inflate (buat) View dari XML, bungkus ke ViewHolder |
| `onBindViewHolder()` | Saat item perlu ditampilkan/diperbarui | Isi View dengan data dari List[position] |
| `getItemCount()` | Setiap saat | Beritahu RecyclerView jumlah total item |

**Alur kerja Adapter secara sederhana:**

```
RecyclerView: "Saya perlu tampilkan item ke-3!"
      ↓
Adapter onBindViewHolder(holder, position=2):
  - Ambil data: todoList[2]
  - Isi view: holder.tvTitle.text = todoList[2].title
  - Isi view: holder.checkboxDone.isChecked = todoList[2].isDone
      ↓
Item ke-3 tampil di layar ✓
```

---

### 2.5 LayoutManager

`LayoutManager` bertugas mengatur **bagaimana item-item disusun** di dalam RecyclerView — apakah vertikal, horizontal, atau grid.

Android menyediakan 3 jenis LayoutManager bawaan:

| LayoutManager | Tampilan | Contoh Penggunaan |
|---|---|---|
| `LinearLayoutManager` (vertikal) | Daftar dari atas ke bawah | Chat, todo list, daftar kontak |
| `LinearLayoutManager` (horizontal) | Daftar dari kiri ke kanan | Carousel produk, thumbnail |
| `GridLayoutManager` | Grid dengan kolom tertentu | Galeri foto, daftar produk |
| `StaggeredGridLayoutManager` | Grid tidak beraturan (Pinterest-style) | Pinterest, feed berita |

```kotlin
// Contoh penggunaan masing-masing:

// Vertikal (paling umum)
recyclerView.layoutManager = LinearLayoutManager(this)

// Horizontal
recyclerView.layoutManager = LinearLayoutManager(this, LinearLayoutManager.HORIZONTAL, false)

// Grid 2 kolom
recyclerView.layoutManager = GridLayoutManager(this, 2)
```

---

### 2.6 Bagaimana Semua Komponen Terhubung

```
┌─────────────────────────────────────────────────┐
│                  MainActivity                   │
│                                                 │
│  todoList ──────────────────────────┐           │
│  (MutableList<TodoItem>)            │           │
│                                     ▼           │
│                              ┌─────────────┐   │
│                              │  TodoAdapter │   │
│                              │             │   │
│                              │ onCreateVH()│   │
│                              │ onBindVH()  │   │
│                              │ getCount()  │   │
│                              └──────┬──────┘   │
│                                     │           │
│  recyclerView.adapter = todoAdapter─┘           │
│  recyclerView.layoutManager = LinearLayout...   │
│                                     │           │
│                              ┌──────▼──────┐   │
│                              │ RecyclerView │   │
│                              │             │   │
│                              │ [item 1]    │   │
│                              │ [item 2]    │   │
│                              │ [item 3]    │   │
│                              │   ...       │   │
│                              └─────────────┘   │
└─────────────────────────────────────────────────┘
```

**Urutan yang terjadi saat aplikasi dibuka:**

```
1. MainActivity.onCreate() dipanggil
2. setContentView() → layout activity_main.xml dimuat
3. RecyclerView ditemukan via findViewById()
4. TodoAdapter dibuat dengan todoList sebagai data
5. LayoutManager dipasang ke RecyclerView
6. Adapter dipasang ke RecyclerView
7. RecyclerView memanggil adapter.getItemCount() → tahu ada berapa item
8. Untuk setiap item yang terlihat di layar:
   a. onCreateViewHolder() → buat View baru dari item_todo.xml
   b. onBindViewHolder()   → isi View dengan data todoList[position]
9. Item tampil di layar ✓
```

---

## 3. Persiapan Project

- Buat project baru di Android Studio
- Pilih template: **Empty Views Activity**
- Language: **Kotlin**
- Minimum SDK: **API 21** (atau sesuai kebutuhan)

---

## 4. Struktur Folder Project

```
app/
├── manifests/
│   └── AndroidManifest.xml
├── java/com.example.todoapp/
│   ├── MainActivity.kt          ← Activity utama
│   ├── TodoItem.kt              ← Data class (model)
│   └── TodoAdapter.kt           ← Adapter RecyclerView
└── res/
    └── layout/
        ├── activity_main.xml    ← Layout utama
        └── item_todo.xml        ← Layout tiap item di list
```

---

## Step 1 — Tambah Dependency

Buka file `build.gradle (Module: app)`, lalu pastikan dependency berikut sudah ada di blok `dependencies`:

```groovy
dependencies {
    // RecyclerView — komponen utama yang akan digunakan
    implementation 'androidx.recyclerview:recyclerview:1.3.2'

    // Dependency lain bawaan project (biarkan saja)
    implementation 'androidx.core:core-ktx:1.12.0'
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.google.android.material:material:1.11.0'
}
```

> **Catatan:** Jika menggunakan Android Studio terbaru, RecyclerView biasanya sudah otomatis tersedia melalui `material` dependency. Namun lebih aman jika ditambahkan secara eksplisit.

Setelah menambahkan, klik **"Sync Now"** di bagian atas editor.

---

## Step 2 — Buat Data Class (Model)

Buat file Kotlin baru: **`TodoItem.kt`**

```kotlin
// TodoItem.kt

data class TodoItem(            // "data class" adalah class khusus di Kotlin untuk menyimpan data
    val id: Int,                // ID unik untuk setiap item todo
    val title: String,          // Judul/nama tugas
    var isDone: Boolean = false // Status apakah tugas sudah selesai, default-nya false
)
```

**Penjelasan baris per baris:**

| Baris | Penjelasan |
|---|---|
| `data class TodoItem(...)` | `data class` secara otomatis menghasilkan `equals()`, `hashCode()`, `toString()`, dan `copy()` — cocok untuk model data |
| `val id: Int` | `val` berarti nilai tidak bisa diubah (immutable). ID bersifat tetap |
| `val title: String` | Judul todo juga tidak berubah setelah dibuat |
| `var isDone: Boolean = false` | `var` berarti nilai bisa diubah (mutable). Status bisa berubah saat user mencentang |
| `= false` | Nilai default — saat item baru dibuat, statusnya belum selesai |

---

## Step 3 — Buat Layout Item RecyclerView

Buat file XML baru di folder `res/layout/`: **`item_todo.xml`**

Ini adalah tampilan untuk **satu baris** di dalam list.

```xml
<!-- item_todo.xml -->
<!-- Layout untuk satu item todo di dalam RecyclerView -->

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:padding="12dp"
    android:gravity="center_vertical">

    <!--
        CheckBox untuk menandai apakah todo sudah selesai.
        Tidak punya teks sendiri karena teks ada di TextView terpisah.
    -->
    <CheckBox
        android:id="@+id/checkboxDone"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />

    <!--
        TextView untuk menampilkan judul todo.
        layout_weight="1" membuat TextView ini mengambil sisa ruang yang tersedia.
    -->
    <TextView
        android:id="@+id/tvTitle"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:textSize="16sp"
        android:paddingStart="8dp"
        android:text="Sample Todo" />

    <!--
        Tombol hapus berbentuk ikon "X".
        Akan digunakan untuk menghapus item dari list.
    -->
    <ImageButton
        android:id="@+id/btnDelete"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@android:drawable/ic_menu_delete"
        android:background="?attr/selectableItemBackgroundBorderless"
        android:contentDescription="Hapus" />

</LinearLayout>
```

**Penjelasan atribut penting:**

| Atribut | Penjelasan |
|---|---|
| `android:id="@+id/..."` | Mendefinisikan ID agar bisa diakses dari kode Kotlin |
| `android:layout_weight="1"` | TextView mengambil semua sisa ruang horizontal |
| `android:layout_width="0dp"` | Harus `0dp` saat menggunakan `layout_weight` |
| `?attr/selectableItemBackgroundBorderless` | Menambahkan efek ripple saat tombol ditekan |

---

## Step 4 — Buat Layout Activity Main

Buka file `res/layout/activity_main.xml` dan ubah isinya:

```xml
<!-- activity_main.xml -->
<!-- Layout utama yang berisi input tambah todo + daftar RecyclerView -->

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <!-- Judul halaman -->
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="📝 Todo List"
        android:textSize="24sp"
        android:textStyle="bold"
        android:paddingBottom="12dp" />

    <!--
        Row berisi input teks dan tombol tambah.
        Menggunakan LinearLayout horizontal agar sejajar.
    -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <!-- Input untuk mengetik nama todo baru -->
        <EditText
            android:id="@+id/etTodo"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:hint="Masukkan tugas baru..."
            android:imeOptions="actionDone"
            android:inputType="text" />

        <!-- Tombol untuk menambahkan todo ke list -->
        <Button
            android:id="@+id/btnAdd"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginStart="8dp"
            android:text="Tambah" />

    </LinearLayout>

    <!--
        RecyclerView — tempat daftar todo akan ditampilkan.
        layout_height="0dp" + layout_weight="1" → mengisi sisa ruang layar.
    -->
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:layout_marginTop="12dp" />

</LinearLayout>
```

**Mengapa RecyclerView pakai `layout_height="0dp"` dan `layout_weight="1"`?**

Karena parent-nya adalah `LinearLayout` vertikal. Dengan `0dp` + `weight="1"`, RecyclerView akan **mengisi semua ruang tersisa** setelah komponen di atasnya ditempatkan. Ini mencegah RecyclerView terpotong atau overflow ke bawah layar.

---

## Step 5 — Buat Adapter

Ini adalah bagian **terpenting** dari RecyclerView. Buat file Kotlin baru: **`TodoAdapter.kt`**

```kotlin
// TodoAdapter.kt

import android.graphics.Paint          // Untuk efek strikethrough (coretan) pada teks
import android.view.LayoutInflater     // Untuk mengubah file XML layout menjadi View object
import android.view.View               // Kelas dasar semua komponen UI Android
import android.view.ViewGroup          // Container untuk View lainnya
import android.widget.CheckBox         // Widget checkbox
import android.widget.ImageButton      // Widget tombol dengan ikon
import android.widget.TextView         // Widget teks
import androidx.recyclerview.widget.RecyclerView  // Kelas dasar Adapter

// Deklarasi class Adapter
// Parameter pertama: todoList → data yang akan ditampilkan (MutableList agar bisa diubah)
// Parameter kedua: onDeleteClick → fungsi callback saat tombol hapus ditekan
class TodoAdapter(
    private val todoList: MutableList<TodoItem>,     // Daftar data todo
    private val onDeleteClick: (Int) -> Unit         // Callback hapus: menerima posisi item
) : RecyclerView.Adapter<TodoAdapter.TodoViewHolder>() {
    // "RecyclerView.Adapter<>" membutuhkan tipe ViewHolder yang akan kita buat di bawah


    // ─────────────────────────────────────────────────────────────────────────
    // INNER CLASS: ViewHolder
    // Tugasnya: menyimpan referensi ke semua View dalam satu item layout
    // Ini lebih efisien daripada memanggil findViewById() berulang kali
    // ─────────────────────────────────────────────────────────────────────────
    inner class TodoViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        // "itemView" adalah View hasil inflate dari item_todo.xml

        // Ambil referensi ke setiap widget menggunakan ID yang sudah didefinisikan di XML
        val tvTitle: TextView = itemView.findViewById(R.id.tvTitle)
        val checkboxDone: CheckBox = itemView.findViewById(R.id.checkboxDone)
        val btnDelete: ImageButton = itemView.findViewById(R.id.btnDelete)
    }
    // Sekarang setiap kali RecyclerView butuh menampilkan item, cukup gunakan holder.tvTitle, dll.


    // ─────────────────────────────────────────────────────────────────────────
    // FUNGSI 1: onCreateViewHolder()
    // Dipanggil PERTAMA KALI saat RecyclerView butuh membuat View baru.
    // Tugas: inflate (membuat) View dari file XML item_todo.xml
    // ─────────────────────────────────────────────────────────────────────────
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): TodoViewHolder {
        // LayoutInflater.from(parent.context) → mengambil inflater dari konteks RecyclerView
        // inflate(R.layout.item_todo, ...) → mengubah item_todo.xml menjadi View object
        // parent → diperlukan agar layout parameter (padding, margin) diterapkan dengan benar
        // false → jangan langsung attach ke parent, RecyclerView yang akan menanganinya
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_todo, parent, false)

        // Bungkus View ke dalam ViewHolder dan kembalikan
        return TodoViewHolder(view)
    }


    // ─────────────────────────────────────────────────────────────────────────
    // FUNGSI 2: onBindViewHolder()
    // Dipanggil setiap kali item perlu DITAMPILKAN atau diperbarui.
    // Tugas: mengisi View dengan data dari todoList[position]
    // ─────────────────────────────────────────────────────────────────────────
    override fun onBindViewHolder(holder: TodoViewHolder, position: Int) {
        // Ambil data todo berdasarkan posisi saat ini
        val todo = todoList[position]

        // Isi judul todo ke dalam TextView
        holder.tvTitle.text = todo.title

        // Sinkronisasi status checkbox dengan data
        // Matikan listener dulu sebelum set nilai, untuk mencegah infinite loop
        holder.checkboxDone.setOnCheckedChangeListener(null)
        holder.checkboxDone.isChecked = todo.isDone

        // Terapkan efek strikethrough jika todo sudah selesai
        if (todo.isDone) {
            // STRIKE_THRU_TEXT_FLAG → menambahkan garis coretan di tengah teks
            holder.tvTitle.paintFlags = holder.tvTitle.paintFlags or Paint.STRIKE_THRU_TEXT_FLAG
        } else {
            // Hapus flag strikethrough jika todo belum selesai
            // "and" digunakan bersama "inv()" untuk menghapus bit flag tertentu
            holder.tvTitle.paintFlags = holder.tvTitle.paintFlags and Paint.STRIKE_THRU_TEXT_FLAG.inv()
        }

        // Pasang listener pada checkbox
        // Saat user mencentang/uncheck, update data di todoList
        holder.checkboxDone.setOnCheckedChangeListener { _, isChecked ->
            // "_" → parameter pertama (CompoundButton) tidak dipakai, diberi nama "_" agar bersih
            todo.isDone = isChecked   // Update status pada data model

            // Beritahu adapter bahwa item pada posisi ini berubah → UI akan diperbarui
            notifyItemChanged(holder.adapterPosition)
        }

        // Pasang listener pada tombol hapus
        holder.btnDelete.setOnClickListener {
            // Panggil fungsi callback yang dikirim dari Activity
            // Kirimkan posisi item yang ingin dihapus
            onDeleteClick(holder.adapterPosition)
        }
    }


    // ─────────────────────────────────────────────────────────────────────────
    // FUNGSI 3: getItemCount()
    // Wajib di-override. Memberi tahu RecyclerView berapa jumlah total item.
    // ─────────────────────────────────────────────────────────────────────────
    override fun getItemCount(): Int = todoList.size
    // Atau bisa ditulis: return todoList.size


    // ─────────────────────────────────────────────────────────────────────────
    // FUNGSI TAMBAHAN: addItem()
    // Dipanggil dari Activity untuk menambahkan item baru ke list
    // ─────────────────────────────────────────────────────────────────────────
    fun addItem(item: TodoItem) {
        todoList.add(item)              // Tambahkan data ke list
        notifyItemInserted(todoList.size - 1)  // Beritahu adapter bahwa ada item baru di akhir list
        // notifyItemInserted() lebih efisien dari notifyDataSetChanged() karena hanya merender 1 item baru
    }


    // ─────────────────────────────────────────────────────────────────────────
    // FUNGSI TAMBAHAN: removeItem()
    // Dipanggil dari Activity (melalui callback) untuk menghapus item
    // ─────────────────────────────────────────────────────────────────────────
    fun removeItem(position: Int) {
        todoList.removeAt(position)     // Hapus data dari list berdasarkan posisi
        notifyItemRemoved(position)     // Beritahu adapter bahwa item di posisi ini telah dihapus
        // RecyclerView akan otomatis menjalankan animasi penghapusan
    }
}
```

---

## Step 6 — Setup di MainActivity

Buka file **`MainActivity.kt`** dan ubah isinya:

```kotlin
// MainActivity.kt

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle
import android.widget.Button
import android.widget.EditText
import android.widget.Toast
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView

class MainActivity : AppCompatActivity() {

    // Deklarasi variabel untuk komponen UI dan data
    // Menggunakan "lateinit var" karena nilainya belum bisa diinisialisasi sebelum onCreate()
    private lateinit var recyclerView: RecyclerView
    private lateinit var etTodo: EditText
    private lateinit var btnAdd: Button
    private lateinit var todoAdapter: TodoAdapter

    // MutableList untuk menyimpan semua item todo
    private val todoList = mutableListOf<TodoItem>()

    // Counter untuk ID unik setiap item
    private var idCounter = 1

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)  // Pasang layout activity_main.xml

        // ── Inisialisasi View ───────────────────────────────────────────────
        // findViewById mencari komponen berdasarkan ID yang ada di activity_main.xml
        recyclerView = findViewById(R.id.recyclerView)
        etTodo = findViewById(R.id.etTodo)
        btnAdd = findViewById(R.id.btnAdd)

        // ── Setup Adapter ───────────────────────────────────────────────────
        // Buat instance adapter dengan:
        // - todoList sebagai data
        // - lambda sebagai callback saat tombol hapus ditekan
        todoAdapter = TodoAdapter(todoList) { position ->
            // Kode ini akan dipanggil saat tombol hapus di item tertentu ditekan
            todoAdapter.removeItem(position)  // Hapus item dari adapter
        }

        // ── Setup RecyclerView ──────────────────────────────────────────────
        recyclerView.layoutManager = LinearLayoutManager(this)
        // LinearLayoutManager(this) → menampilkan item secara VERTIKAL (default)
        // Opsi lain:
        // LinearLayoutManager(this, LinearLayoutManager.HORIZONTAL, false) → horizontal
        // GridLayoutManager(this, 2) → grid 2 kolom

        recyclerView.adapter = todoAdapter
        // Hubungkan adapter ke RecyclerView — sekarang RecyclerView tahu harus menampilkan apa

        // ── Tambahkan data awal (opsional) ───────────────────────────────────
        todoList.add(TodoItem(idCounter++, "Belajar RecyclerView"))
        todoList.add(TodoItem(idCounter++, "Membuat Todo App"))
        todoList.add(TodoItem(idCounter++, "Push ke GitHub"))
        todoAdapter.notifyDataSetChanged()
        // notifyDataSetChanged() → memberi tahu adapter bahwa seluruh data telah berubah
        // Digunakan di sini karena kita menambahkan beberapa item sekaligus

        // ── Setup Tombol Tambah ─────────────────────────────────────────────
        btnAdd.setOnClickListener {
            val text = etTodo.text.toString().trim()
            // .toString() → mengubah Editable menjadi String
            // .trim() → menghapus spasi di awal dan akhir

            if (text.isEmpty()) {
                // Validasi: jangan tambahkan item kosong
                Toast.makeText(this, "Tugas tidak boleh kosong!", Toast.LENGTH_SHORT).show()
                return@setOnClickListener  // Keluar dari lambda, setara dengan "return" di fungsi biasa
            }

            // Buat item baru dan tambahkan ke adapter
            val newTodo = TodoItem(idCounter++, text)  // idCounter++ → gunakan nilai saat ini, lalu increment
            todoAdapter.addItem(newTodo)

            etTodo.text.clear()  // Kosongkan field input setelah berhasil ditambahkan

            // Scroll ke bawah agar item baru terlihat
            recyclerView.scrollToPosition(todoAdapter.itemCount - 1)
        }
    }
}
```

**Penjelasan bagian penting:**

| Kode | Penjelasan |
|---|---|
| `lateinit var` | Menunda inisialisasi variable sampai `onCreate()`. Cocok untuk View karena layout belum ter-inflate sebelum `setContentView()` |
| `LinearLayoutManager(this)` | Mengatur item ditampilkan dari atas ke bawah secara vertikal |
| `return@setOnClickListener` | Di dalam lambda, kita tidak bisa pakai `return` biasa. `return@label` digunakan untuk keluar dari lambda spesifik |
| `notifyDataSetChanged()` | Meminta adapter me-refresh seluruh tampilan. Dipakai saat data berubah drastis (bukan satu item) |

---

## Step 7 — Tambah Fitur Hapus Item (Bonus)

Fitur hapus sudah terintegrasi dalam kode di atas melalui **callback pattern**. Berikut penjelasan alurnya:

```
User tekan tombol "X" di item
        ↓
onDeleteClick(position) dipanggil di Adapter (baris holder.btnDelete.setOnClickListener)
        ↓
Callback diteruskan ke MainActivity (lambda { position -> ... })
        ↓
todoAdapter.removeItem(position) dipanggil
        ↓
todoList.removeAt(position) menghapus data
        ↓
notifyItemRemoved(position) memberitahu RecyclerView
        ↓
RecyclerView menampilkan animasi & item hilang dari layar
```

Kenapa menggunakan **callback** dan bukan langsung memanipulasi data dari Adapter?

> Prinsip **Separation of Concerns** — Adapter seharusnya hanya bertugas menampilkan data, bukan mengelola data. Logika bisnis (hapus, tambah, edit) sebaiknya ada di Activity atau ViewModel.

---

## Hasil Akhir & Penjelasan Alur

### Bagaimana RecyclerView bekerja secara keseluruhan:

```
[Data: todoList]
      ↓  (dibaca oleh)
[TodoAdapter]
      ↓  (membuat & mengisi)
[TodoViewHolder] ← [item_todo.xml (View)]
      ↓  (ditampilkan di dalam)
[RecyclerView] ← diatur oleh [LinearLayoutManager]
      ↓  (ada di dalam)
[activity_main.xml]
      ↓  (di-load oleh)
[MainActivity]
```

### Kapan setiap fungsi Adapter dipanggil:

| Situasi | Fungsi yang dipanggil |
|---|---|
| RecyclerView pertama kali ditampilkan | `onCreateViewHolder()` → `onBindViewHolder()` |
| User scroll ke bawah | `onCreateViewHolder()` (jika belum ada pool) atau langsung `onBindViewHolder()` |
| `addItem()` dipanggil | `notifyItemInserted()` → `onBindViewHolder()` untuk item baru |
| `removeItem()` dipanggil | `notifyItemRemoved()` → animasi hilang |
| Checkbox dicentang | `notifyItemChanged()` → `onBindViewHolder()` dijalankan ulang untuk item itu |

---

## Tips & Catatan Penting

1. **Selalu gunakan `notifyItemInserted/Removed/Changed`** daripada `notifyDataSetChanged()` untuk performa lebih baik dan animasi yang smooth.

2. **Hindari operasi berat di `onBindViewHolder()`** — fungsi ini dipanggil sangat sering saat scroll. Jangan load gambar dari network atau query database di sini.

3. **Jangan gunakan `holder.adapterPosition` yang deprecated** — gunakan `holder.bindingAdapterPosition` pada versi RecyclerView terbaru.

4. **Untuk proyek yang lebih besar**, pertimbangkan menggunakan `ListAdapter` dengan `DiffUtil` yang otomatis mendeteksi perubahan data dengan lebih efisien.

5. **Urutan setup di Activity yang benar:**
   ```
   setContentView() → findViewById() → buat Adapter → set LayoutManager → set Adapter
   ```
   Jika urutannya salah, bisa menyebabkan NullPointerException.

---

## Ringkasan File yang Dibuat

| File | Lokasi | Fungsi |
|---|---|---|
| `TodoItem.kt` | `java/.../` | Model/struktur data |
| `item_todo.xml` | `res/layout/` | Tampilan satu baris item |
| `activity_main.xml` | `res/layout/` | Tampilan halaman utama |
| `TodoAdapter.kt` | `java/.../` | Adapter RecyclerView |
| `MainActivity.kt` | `java/.../` | Controller utama |

---

*Selesai! Dengan memahami kelima komponen ini, kamu sudah bisa mengembangkan RecyclerView untuk kasus yang lebih kompleks seperti daftar kontak, feed berita, atau galeri foto.*
