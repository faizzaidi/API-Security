# API1:2023 Otorisasi Level Objek yang Rusak 

| Agen ancaman/Vektor serangan | Kelemahan Keamanan | Dampak |
| - | - | - |
| Khusus API: **Mudah** dieksploitasi | **Luas** Prevalensi: **Mudah** Terdeteksi | **Sedang** Teknis: Spesifik Bisnis |
| Penyerang dapat memanfaatkan endpoint API yang rentan terhadap otorisasi level objek yang rusak dengan memanipulasi ID objek yang dikirim dalam permintaan. ID objek bisa apa saja dari integer sekuensial, UUID, atau string generik. Terlepas dari tipe datanya, mereka mudah diidentifikasi dalam target permintaan (parameter path atau query string), header permintaan, atau bahkan sebagai bagian dari payload permintaan. | Masalah ini sangat umum dalam aplikasi berbasis API karena komponen server biasanya tidak sepenuhnya melacak keadaan klien, dan sebaliknya, lebih mengandalkan parameter seperti ID objek, yang dikirim dari klien untuk memutuskan objek mana yang akan diakses. Respons server biasanya cukup untuk memahami apakah permintaan berhasil. | Akses tanpa otorisasi ke objek pengguna lain dapat mengakibatkan pengungkapan data ke pihak yang tidak berwenang, kehilangan data, atau manipulasi data. Dalam keadaan tertentu, akses tanpa otorisasi ke objek juga dapat mengarah pada pengambilalihan akun total. |

## Apakah API Rentan?

Otorisasi level objek adalah mekanisme kontrol akses yang biasanya diimplementasikan di level kode untuk memvalidasi bahwa pengguna hanya dapat mengakses objek yang seharusnya mereka miliki izin untuk mengaksesnya.

Setiap endpoint API yang menerima ID objek, dan melakukan tindakan apa pun pada objek, harus menerapkan pemeriksaan otorisasi level objek. Pemeriksaan tersebut harus memvalidasi bahwa pengguna yang login memiliki izin untuk melakukan tindakan yang diminta pada objek yang diminta.

Kegagalan dalam mekanisme ini biasanya mengarah pada pengungkapan informasi yang tidak sah, modifikasi, atau penghancuran semua data.

Membandingkan ID pengguna sesi saat ini (misalnya dengan mengekstraknya dari token JWT) dengan parameter ID rentan bukanlah solusi yang memadai untuk menyelesaikan Otorisasi Level Objek yang Rusak (BOLA). Pendekatan ini hanya bisa mengatasi sebagian kecil kasus. 

Dalam kasus BOLA, memang dirancang bahwa pengguna akan memiliki akses ke endpoint/fungsi API yang rentan. Pelanggaran terjadi pada level objek, dengan memanipulasi ID. Jika penyerang berhasil mengakses endpoint/fungsi API yang seharusnya tidak mereka akses - ini adalah kasus [Otorisasi Tingkat Fungsi yang Rusak][5] (BFLA) daripada BOLA.

## Skenario Serangan Contoh 

### Skenario #1

Sebuah platform e-commerce untuk toko online (toko) menyediakan halaman daftar dengan grafik pendapatan untuk toko yang di-hosting. Dengan memeriksa permintaan browser, seorang penyerang dapat mengidentifikasi endpoint API yang digunakan sebagai sumber data untuk grafik tersebut dan polanya: `/shops/{shopName}/revenue_data.json`. Menggunakan endpoint API lainnya, penyerang dapat mendapatkan daftar semua nama toko yang di-hosting. Dengan skrip sederhana untuk memanipulasi nama dalam daftar, mengganti `{shopName}` dalam URL, penyerang mendapatkan akses ke data penjualan ribuan toko e-commerce.

### Skenario #2 

Sebuah produsen otomotif telah mengaktifkan kontrol jarak jauh kendaraannya melalui API seluler untuk berkomunikasi dengan ponsel pengemudi. API memungkinkan pengemudi untuk mulai dan menghentikan mesin dari jarak jauh dan mengunci serta membuka kunci pintu. Sebagai bagian dari alur ini, pengguna mengirim Nomor Identifikasi Kendaraan (VIN) ke API.
API gagal memvalidasi bahwa VIN mewakili kendaraan yang dimiliki pengguna yang login, yang menyebabkan kerentanan BOLA. Seorang penyerang dapat mengakses kendaraan yang tidak dimilikinya.

### Skenario #3

Layanan penyimpanan dokumen online memungkinkan pengguna untuk melihat, mengedit, menyimpan, dan menghapus dokumen mereka. Ketika dokumen pengguna dihapus, mutasi GraphQL dengan ID dokumen dikirim ke API.

```
POST /graphql
{
  "operationName":"deleteReports",
  "variables":{
    "reportKeys":["<DOCUMENT_ID>"]
  },
  "query":"mutation deleteReports($siteId: ID!, $reportKeys: [String]!) {
    {
      deleteReports(reportKeys: $reportKeys)
    }
  }"
}
```

Karena dokumen dengan ID yang diberikan dihapus tanpa pemeriksaan izin lebih lanjut, pengguna mungkin dapat menghapus dokumen pengguna lain.

## Cara Mencegah

* Terapkan mekanisme otorisasi yang tepat yang mengandalkan kebijakan dan hierarki pengguna.
* Gunakan mekanisme otorisasi untuk memeriksa apakah pengguna yang login memiliki akses untuk melakukan tindakan yang diminta pada catatan di setiap fungsi yang menggunakan input dari klien untuk mengakses catatan di basis data.  
* Lebih suka menggunakan nilai acak dan tidak terduga sebagai GUID untuk ID catatan. 
* Tulis tes untuk mengevaluasi kerentanan mekanisme otorisasi. Jangan terapkan perubahan yang membuat tes gagal.

## Referensi

### OWASP

* [Cheat Sheet Otorisasi][1]  
* [Cheat Sheet Otomatisasi Pengujian Otorisasi][2]

### Eksternal  

* [CWE-285: Otorisasi Tidak Tepat][3]
* [CWE-639: Otorisasi yang Dilewati Melalui Kunci yang Dikendalikan Pengguna][4]

[1]: https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html
[2]: https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Testing_Automation_Cheat_Sheet.html
[3]: https://cwe.mitre.org/data/definitions/285.html
[4]: https://cwe.mitre.org/data/definitions/639.html
[5]: ./0xa5-broken-function-level-authorization-id.md