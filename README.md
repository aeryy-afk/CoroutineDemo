# Kotlin Coroutines – Pemrograman Asinkron

## Deskripsi Project

Implementasi Kotlin Coroutines pada aplikasi Android untuk mempelajari proses asinkron, penanganan error, cancellation, dan pengelolaan thread.

## 1. Menambahkan getCourseCount()

**Implementasi:**
-  TaskRepository.kt : Menambahkan  getCourseCount()  dengan  delay(1500L) .
-  TasksUiState.kt : Menambahkan  courseCount: Int = 0 .
-  TasksViewModel.kt : Menggunakan  async  dan  await()  untuk mengambil jumlah course.
-  TasksScreen.kt : Menampilkan jumlah course jika nilainya lebih dari 0.

**Hasil dan Analisis:**

Waktu pemuatan tetap sekitar 2 detik karena ketiga proses berjalan bersamaan menggunakan  async .

| Proses | Durasi |
|---|---:|
|  getTasks()  | 2000 ms |
|  getAnnouncement()  | 1000 ms |
|  getCourseCount()  | 1500 ms |

Waktu total mengikuti proses terlama, yaitu 2000 ms.

## 2. Menambahkan Tombol Retry

**Implementasi:**
- Tombol Retry muncul ketika  errorMessage  tidak bernilai  null .
- Tombol memanggil  onLoad  untuk menjalankan kembali  viewModel.loadData() .

**Hasil:**

Data dapat dimuat ulang ketika terjadi error tanpa membuka ulang aplikasi.

## 3. Menangkap Exception

Mengubah  catch (e: IOException)  menjadi  catch (e: Exception) .

**Hasil Pengujian:**

Ketika Load ditekan lalu segera diikuti Cancel, pesan yang dapat muncul:

 Failed to load: StandaloneCoroutine was cancelled 

**Analisis:**

 CancellationException  merupakan turunan dari  Exception . Akibatnya, pembatalan coroutine ikut tertangkap dan dianggap sebagai error biasa.

## 4. Perbandingan Thread.sleep() dan withContext()

**Skenario A: Menggunakan  Dispatchers.IO **

 Thread.sleep(2000L)  dijalankan di dalam  withContext(Dispatchers.IO) .

Hasil: UI tetap responsif karena hanya thread latar belakang yang terblokir.

**Skenario B: Menghapus  withContext(Dispatchers.IO) **

 Thread.sleep(2000L)  dijalankan pada Main Thread.

Hasil: UI mengalami freeze karena Main Thread tidak dapat memproses interaksi pengguna.

**Kesimpulan:**

Operasi yang memblokir thread harus dijalankan pada dispatcher yang sesuai agar UI tetap responsif.

## 5. Pertanyaan Konsep

A. Perbedaan  launch  dan  async 

launch, Tidak menghasilkan nilai untuk diambil. Mengembalikan  Job. Tidak menggunakan  await().

async, Menghasilkan nilai. Mengembalikan  Deferred. Menggunakan  await().

**B. Mengapa  GlobalScope  Dihindari?**

Karena tidak terikat pada lifecycle Activity atau ViewModel. Coroutine dapat terus berjalan meskipun halaman ditutup dan menyebabkan pemborosan sumber daya. Gunakan  viewModelScope  sebagai alternatif.

**C. Apa yang Terjadi Jika  getTasks()  Gagal?**

 getAnnouncement()  juga dapat dibatalkan karena keduanya berada dalam  coroutineScope  yang sama. Hal ini merupakan penerapan Structured Concurrency.

## 6. Menambahkan Tombol Sort

**Implementasi:**
-  TasksViewModel.kt : Menambahkan fungsi  sortTasks() .
- Menggunakan  sortedBy { it.title }  untuk mengurutkan tugas.
- Menggunakan  withContext(Dispatchers.Default)  untuk proses pengurutan.
- Memperbarui UI State menggunakan  _uiState.update .
-  TasksScreen.kt : Menambahkan tombol Sort ketika daftar tugas tersedia.

**Contoh Kode:**

   kotlin
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
   

**Hasil:**

Daftar tugas dapat diurutkan berdasarkan judul dari A sampai Z.

## Teknologi

- Kotlin
- Android Studio
- Kotlin Coroutines
- ViewModel
- StateFlow
- Jetpack Compose

## Kesimpulan

Kotlin Coroutines memungkinkan proses asinkron berjalan secara bersamaan. Penggunaan  launch ,  async , dan  withContext  membantu mengatur pekerjaan sesuai kebutuhan.

Pemilihan dispatcher yang tepat juga penting untuk mencegah Main Thread terblokir dan menjaga aplikasi tetap responsif.
