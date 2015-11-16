## Cyber Jawara 2015 Writeup: Ellegarden


### Pendahuluan

Tutorial ini akan membahas soal _ellegarden_ dari kompetisi Cyber Jawara 2015. Soal ini berupa file executable ELF 32bit.


### Langkah-langkah

* Pertama periksa jenis executable _ellegarden_ tersebut (langkah ini sifatnya opsional):

```
% file ellegarden
ellegarden: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, \
interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=d123e05b945e7da70948c7ffd6b60366a53da3cc, stripped
```

* Selanjutnya, periksa _printable string_ pada executable tersebut:

```
% strings ellegarden
/lib/ld-linux.so.2
libc.so.6
_IO_stdin_used
puts
strcmp
__libc_start_main
__gmon_start__
GLIBC_2.0
PTRh
QVh+
[^_]
Good morning kids!
But, do I know you?
MORNING!
this is your flag CJ2015{Dr_dR3_4r3_w47cH1n6_y0u}
;*2$"(
GCC: (Debian 4.9.2-10) 4.9.2
GCC: (Debian 4.8.4-1) 4.8.4
.shstrtab
.interp
.note.ABI-tag
.note.gnu.build-id
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rel.dyn
.rel.plt
.init
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.jcr
.dynamic
.got
.got.plt
.data
.bss
.comment
```

* Bisa terlihat sebuah string yang cukup menarik yaitu __this is your flag CJ2015{Dr_dR3_4r3_w47cH1n6_y0u}__. String tersebut akan ditampilkan jika parameter yang dimasukkan pada commandline benar. Berikut ini adalah penjelasan dari _static analysis_ aplikasi tersebut menggunakan radare2:

```
[0x08048430]>
/ (fcn) main 113
|           ; var int local_1      @ ebp-0x4
|           ; DATA XREF from 0x08048347 (main)
|           0x0804842b    8d4c2404       lea ecx, [esp + 4]             ; 0x4
|           0x0804842f    83e4f0         and esp, 0xfffffff0            ;
|           0x08048432    ff71fc         push dword [ecx - 4]           ;
|           0x08048435    55             push ebp                       ;
|           0x08048436    89e5           mov ebp, esp                   ; inisialisasi stack
|           0x08048438    51             push ecx                       ;
|           0x08048439    83ec04         sub esp, 4                     ;
|           0x0804843c    89c8           mov eax, ecx                   ; EAX = ARGC
|           0x0804843e    833801         cmp dword [eax], 1             ; apakah user memasukkan parameter?
|       ,=< 0x08048441    7f12           jg 0x8048455                   ; jika iya, maka lompat ke alamat 0x8048455
|       |   0x08048443    83ec0c         sub esp, 0xc                   ; jika tidak, maka tampilkan pesan berikut
|       |   0x08048446    6830850408     push str.Good_morning_kids_    ; parameter1: "Good morning kids!"
|       |   0x0804844b    e8b0feffff     call sym.imp.puts              ; puts("Good morning kids!");
|       |   0x08048450    83c410         add esp, 0x10                  ; normalisasi stack
|      ,==< 0x08048453    eb3f           jmp 0x8048494                  ; lompat ke alamat 0x8048494
|      ||   ; JMP XREF from 0x08048441 (main)
|      |`-> 0x08048455    8b4004         mov eax, dword [eax + 4]       ; [0x4:4]=0x10101
|      |    0x08048458    83c004         add eax, 4                     ;
|      |    0x0804845b    8b00           mov eax, dword [eax]           ; EAX = parameter pada command line (ARGV[1])
|      |    0x0804845d    83ec08         sub esp, 8                     ;
|      |    0x08048460    6843850408     push 0x8048543                 ; parameter2: string pada alamat 0x8048543 "sun"
|      |    0x08048465    50             push eax                       ; parameter1: ARGV[1]
|      |    0x08048466    e885feffff     call sym.imp.strcmp            ; strcmp("sun", ARGV[1]);
|      |    0x0804846b    83c410         add esp, 0x10                  ; normalisasi stack
|      |    0x0804846e    85c0           test eax, eax                  ; apakah hasil dari perbandingan di atas sama?
|      |,=< 0x08048470    7412           je 0x8048484                   ; jika sama, maka lompat ke alamat 0x8048484
|      ||   0x08048472    83ec0c         sub esp, 0xc                   ; jika tidak, maka tampilkan pesan berikut
|      ||   0x08048475    6847850408     push str.Hi__                  ; parameter1: "Hi!.But, do I know you?"
|      ||   0x0804847a    e881feffff     call sym.imp.puts              ; puts("Hi!.But, do I know you?");
|      ||   0x0804847f    83c410         add esp, 0x10                  ; normalisasi stack
|     ,===< 0x08048482    eb10           jmp 0x8048494                  ; lompat ke alamat 0x8048494
|     |||   ; JMP XREF from 0x08048470 (main)
|     ||`-> 0x08048484    83ec0c         sub esp, 0xc                   ;
|     ||    0x08048487    6860850408     push str.MORNING__             ; parameter1: "MORNING!.this is your flag CJ2015{Dr_dR3_4r3_w47cH1n6_y0u}"
|     ||    0x0804848c    e86ffeffff     call sym.imp.puts              ; puts("MORNING!.this is your flag CJ2015{Dr_dR3_4r3_w47cH1n6_y0u}")
|     ||    0x08048491    83c410         add esp, 0x10                  ; normalisasi stack
|     ||    ; JMP XREF from 0x08048482 (main)
|     ||    ; JMP XREF from 0x08048453 (main)
|     ``--> 0x08048494    8b4dfc         mov ecx, dword [ebp-local_1]   ;
|           0x08048497    c9             leave                          ;
|           0x08048498    8d61fc         lea esp, [ecx - 4]             ;
\           0x0804849b    c3             ret                            ;
...
```

* Dan berikut ini adalah string yang terdapat pada alamat __0x8048543__:

```
[0x08048530]> x 16@0x8048543
- offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x08048543  7375 6e00 4869 210a 4275 742c 2064 6f20  sun.Hi!.But, do
```

* Bisa terlihat bahwa string __sun__ diakhiri dengan _null terminator_ akan dibandingkan dengan parameter pertama yang dimasukkan pada _command line_ menggunakan fungsi __strcmp__. Berikut ini adalah hasil dari eksekusi aplikasi tersebut dengan tiga cara:

```
% # Tanpa menggunakan parameter
% ./ellegarden
Good morning kids!
%
% # Menggunakan parameter acak
% ./ellegarden rammstein
Hi!
But, do I know you?
%
% # Menggunakan parameter "sun"
% ./ellegarden sun
MORNING!
this is your flag CJ2015{Dr_dR3_4r3_w47cH1n6_y0u}
```

* Seperti yang bisa dilihat, cara ketiga menampilkan flag string yang berisi flag.


### Penutup

Sekian tutorial kali ini, semoga bermanfaat. Terima kasih kepada Tuhan Yang Maha Esa, Maxindo, N3 dan Anda yang telah membaca tutorial ini.


### Referensi

* [Radare portable reversing framework](http://radare.org/r/)
