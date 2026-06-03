# Firebase Realtime Database CRUD (Kotlin)

## Data Class

```kotlin
data class User(
    val nama: String = "",
    val umur: Int = 0
)
```

---

## Inisialisasi Firebase

```kotlin
private lateinit var database: DatabaseReference

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)

    database = FirebaseDatabase.getInstance().reference
}
```

---

# Create (Tambah Data)

Menyimpan data baru ke Firebase menggunakan `push()` agar setiap data memiliki ID unik.

```kotlin
val userId = database.child("users")
    .push()
    .key

val user = User(
    nama = "Harry",
    umur = 22
)

database.child("users")
    .child(userId!!)
    .setValue(user)
    .addOnSuccessListener {
        Log.d("FIREBASE", "Data berhasil ditambah")
    }
```

### Hasil di Firebase

```json
{
  "users": {
    "-ORabc123": {
      "nama": "Harry",
      "umur": 22
    }
  }
}
```

---

# Read (Ambil Satu Data)

Mengambil data berdasarkan ID.

```kotlin
val userId = "-ORabc123"

database.child("users")
    .child(userId)
    .get()
    .addOnSuccessListener { snapshot ->

        val user =
            snapshot.getValue(User::class.java)

        Log.d(
            "USER",
            "${user?.nama} ${user?.umur}"
        )
    }
```

### Output

```text
Harry 22
```

---

# Read (Ambil Semua Data)

Mengambil seluruh data pada node `users`.

```kotlin
database.child("users")
    .get()
    .addOnSuccessListener { snapshot ->

        for (data in snapshot.children) {

            val user =
                data.getValue(User::class.java)

            Log.d(
                "USER",
                "${user?.nama}"
            )
        }
    }
```

### Output

```text
Harry
Budi
```

---

# Update (Mengubah Seluruh Data)

Mengganti seluruh isi object.

```kotlin
val userBaru = User(
    nama = "Harry",
    umur = 23
)

database.child("users")
    .child("-ORabc123")
    .setValue(userBaru)
```

### Hasil

```json
{
  "nama": "Harry",
  "umur": 23
}
```

---

# Update (Mengubah Satu Field)

Mengubah hanya field tertentu tanpa mengganti seluruh object.

```kotlin
database.child("users")
    .child("-ORabc123")
    .child("umur")
    .setValue(23)
```

Atau menggunakan `updateChildren()`:

```kotlin
val updates = mapOf(
    "umur" to 23
)

database.child("users")
    .child("-ORabc123")
    .updateChildren(updates)
```

---

# Delete (Menghapus Data)

Menghapus data berdasarkan ID.

```kotlin
database.child("users")
    .child("-ORabc123")
    .removeValue()
```

### Sebelum

```json
{
  "users": {
    "-ORabc123": {
      "nama": "Harry",
      "umur": 22
    }
  }
}
```

### Sesudah

```json
{
  "users": {}
}
```

---

# Struktur Data Firebase

```json
{
  "users": {
    "-ORabc123": {
      "nama": "Harry",
      "umur": 22
    },
    "-ORabc456": {
      "nama": "Budi",
      "umur": 25
    }
  }
}
```

---

# Ringkasan CRUD

| Operasi  | Method Firebase                      |
| -------- | ------------------------------------ |
| Create   | `setValue()`                         |
| Read One | `get()`                              |
| Read All | `get()` + `for (snapshot.children)`  |
| Update   | `setValue()` atau `updateChildren()` |
| Delete   | `removeValue()`                      |

---

## Dependency

Tambahkan dependency Firebase Realtime Database pada `build.gradle.kts`.

```kotlin
implementation("com.google.firebase:firebase-database-ktx")
```
