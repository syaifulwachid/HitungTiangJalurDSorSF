
---

# Dokumentasi Rutin AutoLISP: HitungTiangJalurDSorSF

## Deskripsi

Rutin AutoLISP `HitungTiangJalurDSorSF` digunakan untuk menghitung jumlah POLE/TIANG pada jalur distribusi, SF, atau MF dengan memperhitungkan vertex pada objek polyline di AutoCAD. Fungsi ini menghitung vertex dan mengabaikan vertex yang berdekatan (berjarak kurang dari jarak minimum yang ditentukan).

## Cara Penggunaan

1. **Ketik Perintah:**
   Ketik `HitungTiangJalurDSorSF` di command line AutoCAD dan tekan Enter.

2. **Pilih Objek:**
   Pilih satu atau lebih objek polyline yang mewakili jalur kabel.

3. **Tekan Enter:**
   Setelah memilih objek, tekan Enter untuk memproses.

4. **Hasil:**
   Program akan menghitung jumlah POLE/TIANG berdasarkan vertex pada objek polyline yang dipilih. Vertex yang berdekatan (berjarak kurang dari jarak minimum) akan dihitung sebagai satu vertex.

5. **Dialog Informasi:**
   Sebuah pesan dialog akan muncul menampilkan jumlah total POLE/TIANG yang valid dan jumlah vertex yang tidak dihitung karena jaraknya terlalu dekat.

## Detail Program

### Fungsi `show-usage-instructions`

Fungsi ini menampilkan instruksi penggunaan rutin `HitungTiangJalurDSorSF` dalam bentuk dialog di AutoCAD.

```lisp
(defun show-usage-instructions ()
  (alert (strcat
    "Cara menggunakan fungsi untuk MENGHITUNG TIANG\n"
    "SESUAI JALUR DISTRIBUSI/SF/MF\n"
    "(SINGLE LINE/BANYAK LINE SEKALIGUS)\n\n"
    "1. Ketik 'HitungTiangJalurDSorSF' di command line AutoCAD.\n"
    "2. Pilih satu atau lebih objek polyline yang mewakili jalur kabel.\n"
    "3. Setelah memilih objek, tekan Enter.\n"
    "4. Program akan menghitung jumlah POLE/TIANG berdasarkan vertex pada objek polyline yang dipilih.\n"
    "5. Vertex yang berdekatan (berjarak kurang dari jarak minimum) akan dihitung sebagai satu vertex.\n"
    "6. Sebuah pesan dialog akan muncul menampilkan jumlah total POLE/TIANG yang valid dan jumlah vertex yang tidak dihitung karena jaraknya terlalu dekat.\n\n"
    "Dibuat oleh: Syaiful Wachid\n"
    "Senior Project Designer: Fiberhome Indonesia\n"
    "Linkedin Profile: https://www.linkedin.com/in/syaiful-wachid-5373n/"
  ))
)
```

### Fungsi `distance-between-points`

Fungsi ini menghitung jarak antara dua titik 2D atau 3D.

```lisp
(defun distance-between-points (p1 p2)
  (if (and p1 p2)  ;; Pastikan kedua titik valid
    (distance p1 p2)
    nil ;; Jika ada titik yang nil, kembalikan nil
  )
)
```

### Fungsi `C:HitungTiangJalurDSorSF`

Fungsi utama untuk menghitung POLE/TIANG dari objek polyline yang dipilih. Vertex yang berdekatan dihitung sebagai satu vertex.

```lisp
(defun C:HitungTiangJalurDSorSF ( / ss i obj vertexList j k vertexCount ignoredCount minDistance currentVertex otherVertex usedVertices)
  ;; Meminta pengguna untuk memilih objek polyline
  (setq ss (ssget '((0 . "LWPOLYLINE,POLYLINE")))) ;; Hanya objek polyline
  (if (null ss)
    (progn
      (alert "Tidak ada objek polyline yang dipilih.")
      (exit)
    )
  )

  ;; Inisialisasi jumlah vertex, vertex yang tidak dihitung, dan jarak minimum
  (setq totalVertexCount 0)
  (setq ignoredCount 0)
  (setq minDistance 3.1) ;; Jarak minimum antar vertex untuk dihitung (ubah sesuai kebutuhan)
  (setq vertexList '())   ;; Daftar untuk menyimpan semua vertex
  (setq usedVertices '()) ;; Daftar untuk menyimpan vertex yang sudah dihitung

  ;; Loop untuk setiap polyline yang dipilih
  (setq i 0)
  (while (< i (sslength ss))
    (setq obj (vlax-ename->vla-object (ssname ss i))) ;; Dapatkan objek polyline
    (setq vertexCountInObj (1+ (fix (vlax-curve-getEndParam obj)))) ;; Memastikan menghitung semua vertex
    (setq j 0)

    ;; Loop untuk mengambil semua vertex dalam polyline dan menyimpannya ke vertexList
    (while (< j vertexCountInObj)
      (setq currentVertex (vlax-curve-getPointAtParam obj j)) ;; Mendapatkan koordinat vertex
      ;; Periksa apakah vertex valid sebelum menambahkannya ke vertexList
      (if currentVertex
        (setq vertexList (append vertexList (list currentVertex)))
      )
      (setq j (1+ j))
    )
    (setq i (1+ i))
  )

  ;; Hitung total jumlah vertex
  (setq totalVertexCount (length vertexList))

  ;; Loop untuk membandingkan setiap vertex dengan semua vertex lainnya
  (setq j 0)
  (while (< j totalVertexCount)
    (setq currentVertex (nth j vertexList))
    ;; Periksa apakah vertex ini sudah dihitung sebelumnya
    (if (not (member currentVertex usedVertices))
      (progn
        (setq isClose nil)
        ;; Bandingkan vertex saat ini dengan semua vertex lainnya
        (setq k 0)
        (while (< k totalVertexCount)
          (setq otherVertex (nth k vertexList))
          ;; Jangan bandingkan vertex dengan dirinya sendiri dan pastikan otherVertex valid
          (if (and (/= j k) otherVertex)
            (if (< (distance-between-points currentVertex otherVertex) minDistance)
              (progn
                ;; Jika vertex lain berdekatan, tandai sebagai berdekatan dan tambahkan ke daftar yang sudah digunakan
                (setq isClose T)
                (setq usedVertices (append usedVertices (list otherVertex)))
              )
            )
          )
          (setq k (1+ k))
        )
        ;; Jika ada vertex yang berdekatan, tambahkan ke ignoredCount
        (if isClose
          (setq ignoredCount (1+ ignoredCount))
        )
        ;; Tambahkan vertex saat ini ke daftar yang sudah digunakan
        (setq usedVertices (append usedVertices (list currentVertex)))
      )
    )
    (setq j (1+ j)) ;; Lanjutkan ke vertex berikutnya
  )

  ;; Hitung vertex yang valid dengan mengurangi total vertex dengan vertex yang tidak dihitung
  (setq validVertexCount (- totalVertexCount ignoredCount))

  ;; Tampilkan jumlah total vertex dan jumlah vertex yang tidak dihitung
  (alert (strcat 
    "TOTAL TIANG PADA JALUR YANG DIPILIH\n"
    "ADALAH => " (itoa validVertexCount) "\n"
    "Jumlah pole yang tidak dihitung karena jarak terlalu dekat: " (itoa ignoredCount)))

  (princ)
)
```

### Kompilasi

Untuk mengkompilasi file LISP menjadi format FAS, gunakan perintah berikut:

```lisp
(vlisp-compile 'st "C:/Path/To/Your/HitungTiangJalurDSorSF.lsp")
```

Gantilah `"C:/Path/To/Your/HitungTiangJalurDSorSF.lsp"` dengan path yang sesuai untuk file LISP Anda.

---


