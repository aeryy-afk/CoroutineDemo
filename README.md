# Kotlin Coroutines – Pemrograman Asinkron

## Deskripsi Project

Pada latihan ini, saya mempelajari penggunaan Kotlin Coroutines dalam pengembangan aplikasi Android. Latihan dilakukan dengan menambahkan beberapa fitur pada aplikasi Coroutine Demo, sekaligus mencoba bagaimana coroutine bekerja ketika menjalankan proses secara bersamaan, menangani error, dan melakukan pembatalan proses.

Selain itu, saya juga mempelajari pengaruh penggunaan thread terhadap responsivitas aplikasi agar proses yang membutuhkan waktu lama tidak membuat tampilan menjadi macet.

## Hasil Pengerjaan Latihan

### 1. Menambahkan Permintaan `getCourseCount()`

Pada latihan pertama, saya menambahkan fungsi baru untuk mengambil jumlah course yang tersedia.

**Implementasi:**
- Pada `TaskRepository.kt`, saya menambahkan fungsi `suspend fun getCourseCount(): Int` dengan `delay(1500L)` sebagai simulasi proses pengambilan data.
- Pada `TasksUiState.kt`, saya menambahkan properti `courseCount` dengan nilai awal 0.
- Pada `TasksViewModel.kt`, saya menggunakan `async` untuk menjalankan fungsi `getCourseCount()` dan mengambil hasilnya menggunakan `await()`.
- Pada `TasksScreen.kt`, saya menampilkan jumlah course jika nilainya lebih dari 0.

**Hasil dan Analisis:**

Setelah fungsi tersebut ditambahkan, waktu pemuatan aplikasi masih sekitar 2 detik.

Hal ini karena ketiga proses, yaitu `getTasks()`, `getAnnouncement()`, dan `getCourseCount()`, dijalankan secara bersamaan menggunakan `async` di dalam `coroutineScope`.

Berikut perbandingan waktu masing-masing proses:

| Proses | Waktu |
|---|---:|
| `getTasks()` | 2000 ms |
| `getAnnouncement()` | 1000 ms |
| `getCourseCount()` | 1500 ms |

Karena dijalankan secara bersamaan, waktu yang dibutuhkan mengikuti proses paling lama, yaitu `getTasks()` selama 2000 ms. Jadi, waktunya bukan hasil penjumlahan ketiga proses tersebut.

### 2. Menambahkan Tombol Retry

Pada latihan kedua, saya menambahkan tombol Retry agar pengguna bisa mencoba memuat data kembali ketika terjadi error.

**Implementasi:**
- Tombol Retry ditampilkan ketika `state.errorMessage` memiliki nilai.
- Pesan error dan tombol Retry diletakkan berdampingan menggunakan `Row`.
- Ketika tombol Retry ditekan, fungsi `onLoad` akan dipanggil untuk menjalankan kembali `viewModel.loadData()`.

**Hasil:**

Ketika proses pemuatan data mengalami kegagalan, pesan error akan ditampilkan bersama tombol Retry. Dengan adanya tombol tersebut, pengguna bisa mencoba memuat data kembali tanpa perlu menutup dan membuka aplikasi.

### 3. Menangkap Terlalu Banyak Exception

Pada latihan ketiga, saya mencoba mengubah penanganan error dari `catch (e: IOException)` menjadi `catch (e: Exception)`.

Setelah itu, saya menjalankan aplikasi dengan menekan tombol Load dan langsung menekan Cancel.

**Hasil Pengujian:**

Pesan yang dapat muncul adalah:

`Failed to load: StandaloneCoroutine was cancelled`

**Analisis:**

Hal ini terjadi karena pembatalan coroutine menggunakan `CancellationException`, yang merupakan turunan dari `Exception`.

Ketika menggunakan `catch (e: Exception)`, pembatalan coroutine ikut tertangkap oleh blok catch. Akibatnya, proses yang seharusnya berhenti secara normal justru dianggap sebagai error dan pesan kesalahan dapat muncul pada layar.

Dari percobaan ini, saya memahami bahwa penggunaan blok catch harus diperhatikan agar proses pembatalan coroutine tidak dianggap sebagai kesalahan biasa.

### 4. Perbandingan `Thread.sleep()` dan `withContext()`

Pada latihan keempat, saya mencoba mengganti `delay(2000L)` menjadi `Thread.sleep(2000L)` untuk melihat pengaruhnya terhadap aplikasi.

Pengujian dilakukan dengan dua skenario.

**Skenario A: Menggunakan `Thread.sleep()` di dalam `withContext(Dispatchers.IO)`**

Pada skenario ini, saya mengganti `delay(2000L)` dengan `Thread.sleep(2000L)`, tetapi tetap mempertahankan `withContext(Dispatchers.IO)`.

Hasilnya, aplikasi masih bisa merespons sentuhan dan interaksi pengguna.

Hal ini karena `Thread.sleep()` hanya memblokir thread yang sedang menjalankan proses tersebut, yaitu thread dari `Dispatchers.IO`. Sementara itu, Main Thread tetap bisa menjalankan tugas yang berhubungan dengan tampilan aplikasi.

**Skenario B: Menghapus `withContext(Dispatchers.IO)`**

Pada skenario kedua, saya menghapus `withContext(Dispatchers.IO)` sehingga `Thread.sleep(2000L)` dijalankan langsung di dalam fungsi `getTasks()`.

Hasil yang diharapkan adalah aplikasi mengalami freeze selama sekitar 2 detik. Selama proses tersebut berlangsung, aplikasi menjadi tidak responsif karena Main Thread ikut terblokir.

Hal ini terjadi karena `getTasks()` dipanggil dari `viewModelScope.launch` yang secara default berjalan pada Main Dispatcher. Akibatnya, `Thread.sleep()` menghentikan sementara pekerjaan Main Thread.

**Kesimpulan:**

Dari kedua percobaan tersebut, saya memahami bahwa penggunaan thread sangat berpengaruh terhadap responsivitas aplikasi. `Thread.sleep()` di dalam `Dispatchers.IO` tidak menghambat UI, sedangkan jika dijalankan pada Main Thread, aplikasi bisa mengalami freeze.

### 5. Pertanyaan Konsep Kotlin Coroutines

**A. Apa perbedaan `launch` dan `async`?**

`launch` digunakan ketika kita ingin menjalankan coroutine tanpa perlu mengambil nilai hasil dari proses tersebut. Fungsi ini mengembalikan objek `Job`.

Sementara itu, `async` digunakan ketika proses coroutine menghasilkan suatu nilai yang nantinya ingin digunakan. Hasilnya dapat diambil menggunakan `await()` melalui objek `Deferred`.

**B. Mengapa `GlobalScope` sebaiknya dihindari?**

Menurut saya, alasan utama `GlobalScope` sebaiknya dihindari adalah karena coroutine yang dijalankan tidak terikat pada lifecycle Activity atau ViewModel.

Artinya, ketika pengguna menutup halaman aplikasi, coroutine tersebut masih bisa terus berjalan. Hal ini dapat menyebabkan pekerjaan yang tidak diperlukan tetap berlangsung dan menghabiskan sumber daya perangkat.

Sebagai gantinya, kita bisa menggunakan `viewModelScope` agar coroutine mengikuti lifecycle ViewModel.

**C. Apa yang terjadi pada `getAnnouncement()` jika `getTasks()` gagal?**

Jika `getTasks()` mengalami kegagalan, maka coroutine lain yang berada dalam `coroutineScope` yang sama, termasuk `getAnnouncement()`, juga dapat dibatalkan.

Hal ini merupakan bagian dari konsep Structured Concurrency, yaitu setiap coroutine memiliki hubungan dengan parent dan sibling-nya. Ketika salah satu child mengalami kegagalan, parent dan coroutine lain dalam scope tersebut dapat ikut dibatalkan.

### 6. Tantangan: Menambahkan Tombol Sort

Pada tantangan terakhir, saya menambahkan tombol Sort untuk mengurutkan daftar tugas berdasarkan judul secara alfabetis.

**Implementasi:**
- Pada `TasksViewModel.kt`, saya menambahkan fungsi `sortTasks()`.
- Saya menggunakan `sortedBy { it.title }` untuk mengurutkan daftar tugas berdasarkan judul.
- Proses pengurutan dijalankan menggunakan `withContext(Dispatchers.Default)` agar tidak membebani Main Thread.
- Setelah proses pengurutan selesai, hasilnya digunakan untuk memperbarui UI State melalui `_uiState.update`.
- Pada `TasksScreen.kt`, saya menambahkan tombol Sort yang dapat digunakan ketika daftar tugas sudah dimuat.

**Contoh Kode:**

```kotlin
fun sortTasks() {
    viewModelScope.launch {
        val currentTasks = _uiState.value.tasks

        if (currentTasks.isNotEmpty()) {
            val sorted = withContext(Dispatchers.Default) {
                currentTasks.sortedBy { it.title }
            }

            _uiState.update {
                it.copy(tasks = sorted)
            }
        }
    }
}
```

**Hasil:**

Ketika tombol Sort ditekan, daftar tugas akan diurutkan berdasarkan judul dari A sampai Z. Proses pengurutan dilakukan pada `Dispatchers.Default`, sehingga pekerjaan komputasi tidak perlu dijalankan pada Main Thread.

## Teknologi yang Digunakan

Beberapa teknologi dan komponen yang digunakan dalam latihan ini antara lain:

- **Kotlin:** Bahasa pemrograman untuk mengembangkan aplikasi Android.
- **Android Studio:** IDE yang digunakan untuk membuat dan menjalankan aplikasi.
- **Kotlin Coroutines:** Digunakan untuk menjalankan proses secara asinkron.
- **ViewModel:** Mengelola data dan proses yang berhubungan dengan UI.
- **StateFlow:** Mengelola perubahan state agar tampilan dapat diperbarui ketika data berubah.
- **Jetpack Compose:** Digunakan untuk membangun tampilan aplikasi Android.

## Kesimpulan

Setelah menyelesaikan latihan ini, saya menjadi lebih memahami bagaimana Kotlin Coroutines digunakan dalam aplikasi Android. Saya belajar bahwa beberapa proses dapat dijalankan secara bersamaan menggunakan `async`, sedangkan `launch` dapat digunakan untuk menjalankan coroutine yang tidak memerlukan nilai hasil.

Saya juga memahami pentingnya penanganan error dan pembatalan coroutine, serta bagaimana penggunaan `Dispatchers.IO` dan `Dispatchers.Default` dapat membantu menjalankan pekerjaan di luar Main Thread.

Melalui percobaan `Thread.sleep()`, saya bisa melihat secara langsung bahwa pemilihan thread yang tidak tepat dapat menyebabkan aplikasi menjadi tidak responsif. Oleh karena itu, penggunaan coroutine dan dispatcher yang sesuai sangat penting agar aplikasi tetap berjalan dengan baik.
