## Cyber Jawara 2015 Writeup: Beats


### Pendahuluan

Pada tutorial kali ini, akan dibahas mengenai solusi dari soal _beats_ pada kompetisi Cyber Jawara 2015. Soal ini merupakan sebuah aplikasi ELF 32bit.


### Langkah-langkah

* Pertama kita melihat dulu tipe dari aplikasi _beats_ tersebut (langkah ini sifatnya opsional):

```
% file beats
beats: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, \
for GNU/Linux 2.6.32, BuildID[sha1]=63776c74e5a42191ebd0010b6f91dd0da1f4f0a2, not stripped
```

* Selanjutnya, kita akan menggunakan radare2 untuk melakukan analisa terhadap aplikasi tersebut. Gunakan instruksi __aaa__ pada radare2 seperti ini:

```
% r2 beats
[0x080483b0]> aaa
```

* Kemudian, kita akan melakukan _static analysis_ terhadap fungsi __main__ pada aplikasi tersebut. Berikut ini adalah analisa yang telah diberi keterangan untuk masing-masing instruksinya:

```
[0x080483b0]> pd @sym.main
/ (fcn) sym.main 197
|           ; var int local_0_1    @ ebp-0x1                                    ;
|           ; var int local_1      @ ebp-0x4                                    ;
|           ; var int local_3      @ ebp-0xc                                    ; variabel untuk counter
|           ; var int local_23     @ ebp-0x5c                                   ; variabel untuk input flag dari user
|           ; DATA XREF from 0x080483c7 (sym.main)
|           ;-- main:
|           0x080484ab    8d4c2404       lea ecx, [esp + 4]                     ; 0x4
|           0x080484af    83e4f0         and esp, 0xfffffff0                    ;
|           0x080484b2    ff71fc         push dword [ecx - 4]                   ;
|           0x080484b5    55             push ebp                               ;
|           0x080484b6    89e5           mov ebp, esp                           ; inisialisasi stack
|           0x080484b8    51             push ecx                               ;
|           0x080484b9    83ec64         sub esp, 0x64                          ; inisialisasi variabel local sebanyak 0x64 byte
|           0x080484bc    a180980408     mov eax, dword [obj.stdout__GLIBC_2.0] ; EAX = stdout
|           0x080484c1    50             push eax                               ; parameter4: EAX (stdout)
|           0x080484c2    6a10           push 0x10                              ; parameter3: 16 (jumlah karakter yang akan ditulis)
|           0x080484c4    6a01           push 1                                 ; parameter2: 1 (ukuran karakter yang akan ditulis 1 byte)
|           0x080484c6    6819860408     push str.Guess_the_flag:               ; parameter1: "Guess the flag: " @ 0x8048619
|           0x080484cb    e8b0feffff     call sym.imp.fwrite                    ; fwrite("Guess the flag: ", 1, 16, stdout)
|           0x080484d0    83c410         add esp, 0x10                          ; normalisasi stack
|           0x080484d3    a180980408     mov eax, dword [obj.stdout__GLIBC_2.0] ; EAX = stdout
|           0x080484d8    83ec0c         sub esp, 0xc                           ; ESP -= 0x0c
|           0x080484db    50             push eax                               ; parameter1: EAX (stdout)
|           0x080484dc    e87ffeffff     call sym.imp.fflush                    ; fflush(stdout);
|           0x080484e1    83c410         add esp, 0x10                          ; normalisasi stack
|           0x080484e4    a160980408     mov eax, dword [obj.stdin__GLIBC_2.0]  ; EAX = stdin
|           0x080484e9    83ec04         sub esp, 4                             ;
|           0x080484ec    50             push eax                               ; parameter3: EAX (stdin)
|           0x080484ed    6a50           push 0x50                              ; parameter2: 80 byte
|           0x080484ef    8d45a4         lea eax, [ebp-local_23]                ; EAX = buffer
|           0x080484f2    50             push eax                               ; parameter1: EAX (buffer)
|           0x080484f3    e878feffff     call sym.imp.fgets                     ; fgets(local_23, 80, stdin);
|           0x080484f8    83c410         add esp, 0x10                          ; normalisasi stack
|           0x080484fb    c745f4000000.  mov dword [ebp-local_3], 0             ; local_3 = 0 (counter)
|       ,=< 0x08048502    eb40           jmp 0x8048544                          ; lompat ke alamat 0x8048544
|       |   ; JMP XREF from 0x0804854a (sym.main)
|      .--> 0x08048504    8d55a4         lea edx, [ebp-local_23]                ; EDX = local_23 (buffer)
|      ||   0x08048507    8b45f4         mov eax, dword [ebp-local_3]           ; EAX = counter
|      ||   0x0804850a    01d0           add eax, edx                           ; EAX = ambil karakter sesuai dengan counter (buffer)
|      ||   0x0804850c    0fb600         movzx eax, byte [eax]                  ; EAX = buffer[i]
|      ||   0x0804850f    f7d0           not eax                                ; EAX ~= EAX
|      ||   0x08048511    89c2           mov edx, eax                           ; EDX = hasilnya disimpan di register EDX
|      ||   0x08048513    8b45f4         mov eax, dword [ebp-local_3]           ; EAX = counter
|      ||   0x08048516    0500860408     add eax, obj.cipher                    ; EAX = ambil karakter sesuai dengan counter (cipher)
|      ||   0x0804851b    0fb600         movzx eax, byte [eax]                  ; EAX = cipher[i]
|      ||   0x0804851e    38c2           cmp dl, al                             ; bandingkan nilai pada register DL dan AL
|     ,===< 0x08048520    741e           je 0x8048540                           ; jika sama, maka lompat ke alamat 0x8048540
|     |||   0x08048522    a180980408     mov eax, dword [obj.stdout__GLIBC_2.0] ; EAX = stdout
|     |||   0x08048527    50             push eax                               ; parameter4: EAX (stdout)
|     |||   0x08048528    6a0a           push 0xa                               ; parameter3: 10 (jumlah karakter yang akan ditulis)
|     |||   0x0804852a    6a01           push 1                                 ; parameter2: 1 (ukuran karakter yang akan ditulis 1 byte)
|     |||   0x0804852c    682a860408     push str.Nice_try._n                   ; parameter1: "Nice try.." @ 0x804862a
|     |||   0x08048531    e84afeffff     call sym.imp.fwrite                    ; fwrite("Nice try..", 1, 10, stdout);
|     |||   0x08048536    83c410         add esp, 0x10                          ; normalisasi stack
|     |||   0x08048539    b8ffffffff     mov eax, 0xffffffff                    ; eax = return value (-1)
|    ,====< 0x0804853e    eb28           jmp 0x8048568                          ; lompat ke instruksi untuk keluar dari aplikasi
|    ||||   ; JMP XREF from 0x08048520 (sym.main)                               ;
|    |`---> 0x08048540    8345f401       add dword [ebp-local_3], 1             ; tambahkan 1 ke variabel counter
|    | ||   ; JMP XREF from 0x08048502 (sym.main)                               ;
|    | |`-> 0x08048544    8b45f4         mov eax, dword [ebp-local_3]           ; EAX = local_3 (counter)
|    | |    0x08048547    83f818         cmp eax, 0x18                          ; apakah jumlahnya sudah 24 karakter?
|    | `==< 0x0804854a    76b8           jbe 0x8048504                          ; jika belum, maka ulangi ke alamat 0x8048504
|    |      0x0804854c    a180980408     mov eax, dword [obj.stdout__GLIBC_2.0] ; EAX = stdout
|    |      0x08048551    50             push eax                               ; parameter4: EAX (stdout)
|    |      0x08048552    6a11           push 0x11                              ; parameter3: 17 (jumlah karakter yang akan ditulis)
|    |      0x08048554    6a01           push 1                                 ; parameter2: 1 (ukuran karakter yang akan ditulis 1 byte)
|    |      0x08048556    6835860408     push str.Congraturations__n            ; parameter1: "Congraturations!." @ 0x8048635
|    |      0x0804855b    e820feffff     call sym.imp.fwrite                    ; fwrite("Congraturations!.", 1, 17, stdout);
|    |      0x08048560    83c410         add esp, 0x10                          ; normalisasi stack
|    |      0x08048563    b800000000     mov eax, 0                             ; EAX = 0 (return value)
|    |      ; JMP XREF from 0x0804853e (sym.main)
|    `----> 0x08048568    8b4dfc         mov ecx, dword [ebp-local_1]           ;
|           0x0804856b    c9             leave                                  ; normalisasi stack
|           0x0804856c    8d61fc         lea esp, [ecx - 4]                     ;
```

* Pada alamat __0x08048516__ bisa terlihat instruksi yang mengakses variabel __cipher__, dan pada alamat __0x08048547__ bisa terlihat bahwa kalkulasi jumlah karakter pada flag (ukuran flag) adalah __0x18__ heksadesimal atau 24 desimal. Jadi kita cukup melihat 24 byte dari variabel cipher. Caranya adalah sebagai berikut:

```
[0x080487b0]> x 0x18@obj.cipher
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x08048600  bcb5 cdcf ceca 84bb 8da0 bb8d 9aa0 889e  ................
0x08048610  8ca0 979a 8d9a 82f5                      ........
```

* Untuk menemukan flag yang benar, maka kita akan melakukan instruksi _bitwise NOT_ untuk setiap karakter pada variabel cipher tersebut. Caranya adalah seperti ini:

```python
[0x080483b0]> !python
Python 2.7.10 (default, Sep  8 2015, 17:20:17)
[GCC 5.1.1 20150618 (Red Hat 5.1.1-4)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> print ''.join([chr(~ord(c) & 0xff) for c in "\xbc\xb5\xcd\xcf\xce\xca\x84\xbb\x8d\xa0\xbb\x8d\x9a\xa0\x88\x9e\x8c\xa0\x97\x9a\x8d\x9a\x82\xf5"])
CJ2015{Dr_Dre_was_here}
```

* Bisa terlihat bahwa flag yang benar adalah __CJ2015{Dr_Dre_was_here}__.


### Penutup

Sekian tutorial singkat ini, semoga bermanfaat. Terima kasih kepada Tuhan Yang Maha Esa, Maxindo, N3 dan Anda yang telah membaca tutorial ini.


### Referensi

* [Radare portable reversing framework](http://radare.org/r/)
