# HitungTiangJalurDSorSF

## Deskripsi

`HitungTiangJalurDSorSF` adalah sebuah program AutoLISP yang digunakan untuk menghitung jumlah tiang (pole) pada jalur distribusi atau jalur SF/MF di AutoCAD. Program ini menghitung jumlah vertex dari objek polyline yang dipilih, di mana setiap vertex dianggap sebagai tiang.

## Instruksi Penggunaan

1. **Ketik `HitungTiangJalurDSorSF` di command line AutoCAD.**
2. **Pilih satu atau lebih objek polyline yang ingin dihitung jumlah tiang-nya.**
3. **Tekan Enter setelah memilih objek.**
4. **Program akan menghitung jumlah tiang dari objek polyline yang dipilih.**
5. **Sebuah pesan dialog akan muncul menampilkan total jumlah tiang.**

## Kode Program

```lisp
(defun show-usage-instructions ()
  (alert (strcat
    "Cara menggunakan fungsi untuk MENGHITUNG TIANG\n"
    "SESUAI JALUR DISTRIBUSI/SF/MF\n"
    "(SINGLE LINE/BANYAK LINE SEKALIGUS)\n\n"
    "1. Ketik 'HitungTiangJalurDSorSF' di command line AutoCAD.\n"
    "2. Pilih satu atau lebih objek jalur kabel yang ingin dihitung jumlah POLE/TIANG-nya.\n"
    "3. Setelah memilih, tekan Enter.\n"
    "4. Program akan menghitung jumlah POLE/TIANG dari objek jalur kabel yang dipilih.\n"
    "5. Sebuah pesan dialog akan muncul menampilkan total jumlah POLE/TIANG.\n\n"
    "Dibuat oleh: Syaiful Wachid\n"
    "Senior Project Designer: Fiberhome Indonesia\n"
    "Linkedin Profile: https://www.linkedin.com/in/syaiful-wachid-5373n/"
  ))
)

;; Tampilkan instruksi penggunaan saat program dimuat
(show-usage-instructions)

(defun C:HitungTiangJalurDSorSF ( / ss i obj vertexCount)
  ;; Meminta pengguna untuk memilih objek polyline
  (setq ss (ssget '((0 . "POLYLINE,LWPOLYLINE")))) ;; Hanya objek polyline
  (if (null ss)
    (progn
      (alert "Tidak ada objek polyline yang dipilih.")
      (exit)
    )
  )

  ;; Inisialisasi jumlah vertex
  (setq vertexCount 0)

  ;; Loop untuk menghitung jumlah vertex dari setiap polyline yang dipilih
  (setq i 0)
  (while (< i (sslength ss))
    (setq obj (ssname ss i))
    (setq vertexCount (+ vertexCount (cdr (assoc 90 (entget obj))))) ;; Ambil jumlah vertex
    (setq i (1+ i))
  )

  ;; Tampilkan jumlah total vertex
  (alert (strcat 
    "TOTAL TIANG PADA JALUR YANG DIPILIH\n"
    "ADALAH => " (itoa vertexCount)))

  (princ)
)
