1. Tambah Permintaan getCourseCount()
•
Implementasi Kode:
◦
 TaskRepository.kt: Menambahkan suspend fun getCourseCount(): Int dengan delay(1500L).
◦
 TasksUiState.kt: Menambahkan field courseCount: Int = 0.
◦
 TasksViewModel.kt: Memanggil async { repository.getCourseCount() } di dalam coroutineScope dan mengambil hasilnya dengan .await().
◦
 TasksScreen.kt: Menampilkan informasi Course Count di layar jika nilainya > 0.
•
Jawaban Pertanyaan:
Apakah total waktu pemuatan masih sekitar 2 detik? Jelaskan alasannya.
Ya, total waktu pemuatan masih sekitar 2 detik.
Alasan: Karena ketiga tugas (getTasks() berdurasi 2000ms, getAnnouncement() berdurasi 1000ms, dan getCourseCount() berdurasi 1500ms) dieksekusi secara sejajar/paralel menggunakan async di dalam coroutineScope. Total waktu pemuatan ditentukan oleh tugas dengan durasi paling lama (max(2000ms, 1000ms, 1500ms) = 2000ms), bukan penjumlahan kumulatif dari ketiganya.
2. Tombol Retry
•
Implementasi Kode:
◦
Pada  TasksScreen.kt, ketika state.errorMessage tidak null, pesan error ditampilkan dalam Row berdampingan dengan tombol Retry.
◦
Mengklik tombol Retry memanggil fungsi onLoad yang menjalankan kembali viewModel.loadData().
3. Menangkap Terlalu Banyak (catch (e: Exception))
•
Jawaban & Penjelasan:
Pesan apa yang sekarang muncul jika catch (e: IOException) diganti catch (e: Exception), lalu menekan Load lalu segera Cancel?
Pesan yang muncul: "Failed to load: StandaloneCoroutine was cancelled" (atau pesan dari CancellationException).
Kenapa berbeda dari sebelumnya? Pembatalan coroutine pada Kotlin diimplementasikan dengan melempar CancellationException. Jika kita menggunakan catch (e: Exception), maka CancellationException ikut tertangkap oleh blok catch alih-alih dibiarkan meneruskan proses pembatalan (cancellation propagation). Akibatnya, coroutine yang seharusnya dibatalkan secara bersih malah dianggap sebagai kegagalan/error biasa dan pesan error muncul di UI.
4. Di Mana Macetnya? (Thread.sleep vs withContext)
•
Jawaban & Penjelasan:
Skenario A: Mengganti delay(2000L) dengan Thread.sleep(2000L) di dalam withContext(Dispatchers.IO) pada getTasks() Layar TIDAK MACET. Thread.sleep memblokir thread latar belakang (IO thread pool), sehingga Main Thread (UI thread) tetap bebas merespons sentuhan dan menggambar ulang layar.
Skenario B: Menghapus withContext(Dispatchers.IO) lalu menjalankan Thread.sleep(2000L) Layar JADI MACET (freeze) selama 2 detik. Tanpa withContext(Dispatchers.IO), getTasks() dieksekusi langsung pada thread pemanggilnya (yaitu Main Thread jika dipanggil dari viewModelScope.launch secara default). Thread.sleep akan menghentikan seluruh aktivitas Main Thread.
5. Pertanyaan Konsep
•
(a) Beda launch dan async:
◦
launch: Digunakan untuk coroutine fire-and-forget yang tidak mengembalikan hasil/nilai (mengembalikan objek Job).
◦
async: Digunakan untuk coroutine yang mengembalikan hasil/nilai (mengembalikan objek Deferred<T>), di mana hasilnya dapat diperoleh dengan memanggil .await().
•
(b) Kenapa GlobalScope sebaiknya dihindari:
◦
GlobalScope terikat pada seluruh siklus hidup aplikasi (application lifecycle), bukan pada siklus hidup UI/ViewModel. Jika Activity atau Screen ditutup, coroutine di GlobalScope akan terus berjalan di latar belakang, yang mengakibatkan memory leak dan pemborosan sumber daya HP.
•
(c) Apa yang terjadi pada getAnnouncement() kalau getTasks() gagal:
◦
Sesuai prinsip Structured Concurrency, kegagalan/exception pada salah satu child coroutine (getTasks()) akan secara otomatis membatalkan sibling coroutine yang sedang berjalan (getAnnouncement()) dan membatalkan parent scope (coroutineScope).
6. Tantangan: Tombol Sort
•
Implementasi Kode:
◦
 TasksViewModel.kt: Menambahkan fungsi sortTasks() yang mengurutkan daftar tugas berdasarkan judul menggunakan withContext(Dispatchers.Default) agar komputasi berat tidak membebani Main Thread:
kotlin
fun sortTasks() {
    viewModelScope.launch {
        val currentTasks = _uiState.value.tasks
        if (currentTasks.isNotEmpty()) {
            val sorted = withContext(Dispatchers.Default) {
                currentTasks.sortedBy { it.title }
            }
            _uiState.update { it.copy(tasks = sorted) }
        }
    }
}
◦
 TasksScreen.kt: Menambahkan tombol Sort di baris aksi yang aktif jika daftar tugas telah dimuat.
