## Cyber Jawara 2015 Writeup: Scarlet


### Pendahuluan

Tutorial singkat kali ini akan membahas solusi soal _scarlet_ pada kompetisi Cyber Jawara 2015. Soal tersebut berisi 2 buah berkas, yaitu sebuah berkas gambar dengan nama __Capture.JPG__ dan sebuah berkas teks dengan nama __kabayan.txt__.


### Langkah-langkah

* Berikut ini adalah informasi dari kedua berkas tersebut:

```
% file *
Capture.JPG: JPEG image data, JFIF standard 1.01, resolution (DPI), density 96x96, segment length 16, \
             Exif Standard: [TIFF image data, big-endian, direntries=4], baseline, precision 8, 205x33, frames 3
kabayan.txt: ASCII text, with very long lines, with CRLF line terminators
```

* Sekarang, perhatikan gambar __Capture.JPG__. Gambar tersebut menggunakan karakter [dancingmen](http://www.fonts2u.com/gl-dancingmen.font), yang jika diterjemahkan akan menghasilkan kata __CYRJWRA__.

* Selanjutnya, buka berkas __kabayan.txt__ menggunakan teks editor, bisa terlihat bahwa teks pada berkas tersebut menggunakan _encoding_ base64 sehingga terlebih dahulu harus di-decode dengan cara seperti ini:

```
% base64 -d < kabayan.txt > kabayan.bin
base64: invalid input
```

* Pesan _invalid input_ pada langkah di atas terjadi kerena pada bagian akhir berkas __kabayan.txt__ terdapat karakter baris baru yang biasa dikenal dengan istilah _newline_ (karakter "\r\n"). Hasil dari langkah di atas adalah berkas bernama __kabayan.bin__. Selanjutnya kita perlu mencari tahu jenis berkas tersebut:

```
% file kabayan.bin
kabayan.bin: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, Exif Standard: \
             [TIFF image data, little-endian, direntries=2, software=Google], baseline, precision 8, 153x210, frames 3
```

* Bisa terlihat bahwa berkas tersebut merupakan gambar dalam format JPEG. Selanjutnya periksa _string_ yang terdapat pada berkas tersebut:

```
% strings kabayan.bin
JFIF
~Exif
Google
...
flag.txt
flag.txt
```

* Ternyata gambar tersebut berisi __flag.txt__. Selanjutnya, gunakan binwalk untuk melihat informasi __flag.txt__ pada gambar tersebut:

```
% binwalk kabayan.bin

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01
30            0x1E            TIFF image data, little-endian offset of first image directory: 8
8574          0x217E          Zip archive data, encrypted at least v2.0 to extract, compressed size: 29, uncompressed size: 15, name: flag.txt
8747          0x222B          End of Zip archive
```

* Ekstrak arsip zip tersebut dari gambar menggunakan __dd__ seperti ini:

```
% dd if=kabayan.bin of=flag.zip skip=8574 bs=1
195+0 records in
195+0 records out
195 bytes (195 B) copied, 0.00127534 s, 153 kB/s
```

* Hasilnya adalah arsip zip yang berisi berkas __flag.txt__:

```
% file flag.zip
flag.zip: Zip archive data, at least v2.0 to extract
...
% 7z l flag.zip

7-Zip [64] 9.20  Copyright (c) 1999-2010 Igor Pavlov  2010-11-18
p7zip Version 9.20 (locale=en_US.utf8,Utf16=on,HugeFiles=on,4 CPUs)

Listing archive: flag.zip

--
Path = flag.zip
Type = zip
Physical Size = 195

   Date      Time    Attr         Size   Compressed  Name
------------------- ----- ------------ ------------  ------------------------
2015-11-06 13:00:11 ....A           15           29  flag.txt
------------------- ----- ------------ ------------  ------------------------
                                    15           29  1 files, 0 folders
```

* Gunakan __CYRJWRA__ sebagai kata kunci untuk membuka arsip tersebut:

```
% 7z x -pCYRJWRA flag.zip

7-Zip [64] 9.20  Copyright (c) 1999-2010 Igor Pavlov  2010-11-18
p7zip Version 9.20 (locale=en_US.utf8,Utf16=on,HugeFiles=on,4 CPUs)

Processing archive: flag.zip

Extracting  flag.txt

Everything is Ok

Size:       15
Compressed: 195
```

* Langkah terakhir adalah melihat isi dari berkas __flag.txt__:

```
% cat flag.txt
w1lu73n9_T3P4ng
```


### Penutup

Sekian tutorial singkat kali ini, semoga bermanfaat. Terima kasih kepada Tuhan Yang Maha Esa, Maxindo, N3 dan Anda yang telah membaca tutorial ini.


### Referensi

* [GL-DancingMen Font](http://www.fonts2u.com/gl-dancingmen.font)
