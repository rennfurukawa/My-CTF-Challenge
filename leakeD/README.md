# leakeD

## Overview & Description

Challenge bisa diakses [disini](https://ctf.tcp1p.com/challenges#leak%20eD-50)

Category : Cryptography

Aku baru saja belajar tentang enkripsi bernama RSA kemarin, lalu temanku mencoba memberiku tantangan untuk menyelesaikannya. Tapi aku merasa aneh disini, kenapa eksponennya sangatlah kecil?....

Oh iya, kemarin temanku juga membisikkan kepadaku bahwa ada sesuatu yang bocor, tapi sampai sekarang aku tidak tahu maksudnya. Bisakah kamu membantuku?

Author : rennfurukawa

[rsa.py](/rsa.py)  [enc.txt](/enc.txt)

### TL;DR

- Diberikan beberapa komponen dalam enkripsi RSA : **N, E, C, D**. Namun disini **D** diberikan dengan sebuah rumus (**d % (p - 1)**)

- Namun disini **e / Public Exponent** ukurannya sangat kecil, yang dimana memungkinkan penyerang untuk melakukan penyerangan dengan metode **Coppersmith Method,  Hastad's Broadcast Attack,  Franklin-Reiter Related Message Attack.**

- Untuk recover **d**, kita bisa menggunakan **Chinese Remainder Theorem**. [Bukti ada di halaman 7](https://eprint.iacr.org/2004/147.pdf), [CRT-RSA](https://nitaj.users.lmno.cnrs.fr/rsa21.pdf)

- Dari page diatas, kita bisa mengimplementasikan bahwa **e*dp = 1 % (p - 1)**

- **e*dp - 1 = 0 % (p - 1)** yang dimana **e*dp - 1** membagi rata **p-1**

- Jadi kita tahu bahwa **e*dp - 1 = k*(p - 1)**, yang dimana **k < e** karena **k(p - 1)** adalah kelipatan dari **e*dp - 1**

- Untuk me-recover **p**, cukup menggunakan rumus **p = (e*dp -1 + k) / k**

- Kita bisa mencoba semua kemungkinan **k / key** dari  **0 - e = 5**, sehingga **N / p = q** merupakan bilangan bulat.

- Lalu kita mencari nilai q dengan rumus **q = N / p**

- Maka **d** adalah **d = 1 / e * (p - 1) * (q - 1)**

- Langkah terakhir yaitu men-dekripsi ciphertext dengan rumus **m = c^d mod n**


## Exploit

Diberikan 2 file bernama **rsa.py** & **enc.txt**, disini kita tahu bahwa metode enkripsi berada pada file **rsa.py** dan output-nya ada di **enc.txt**.

![](https://i.imgur.com/zVdiy4B.png)

Setelah meneliti source code enkripsi lebih lanjut, kita dapat mengetahui beberapa informasi, diantaranya: 

- Diketahui beberapa komponen RSA pada umumnya, yaitu **N, E , C**, namun disini yang menarik adalah kita diberikan sebuah **D / Private Exponent**, yang dimana hal ini seharusnya tidak boleh dibocorkan didalam enkripsi RSA.

- **D** yang bocor bukan benar2 murni, namun dibocorkan dengan sebuah rumus **(d % (p - 1))**

- Dan juga ada **e / Public Exponent**, namun ukurannya sangat kecil, yang dimana memungkinkan penyerang untuk melakukan penyerangan bisa saja dengan metode **Coppersmith Method,  Hastad's Broadcast Attack,  Franklin-Reiter Related Message Attack, dan lain-lain**

Setelah mengetahui hal-hal tersebut, untuk penyerangan, kita hanya perlu me-recover **P**m dari **d** yang dibocorkan tersebut. Idenya adalah dengan menggunakan **Chinese Remainder Theorem**

Proof of Work :

- Dari [sini](https://eprint.iacr.org/2004/147.pdf), kita bisa mengimplementasikan **e*dp = 1 % (p-1)**

- **e*dp - 1 = 0 % (p - 1)** yang dimana **e*dp - 1** membagi rata **p-1**

- Jadi kita tahu bahwa **e*dp - 1 = k*(p - 1)**, yang dimana **k < e** karena **k(p - 1)** adalah kelipatan dari **e*dp - 1**

- Untuk me-recover **p**, cukup menggunakan rumus **p = (e*dp -1 + k) / k**

- Kita bisa mencoba semua kemungkinan **k / key** dari  **0 - e = 5**, sehingga **N / p = q** merupakan bilangan bulat.

- Lalu kita mencari nilai q dengan rumus **q = N / p**

- Maka **d** adalah **d = 1 / e * (p - 1) * (q - 1)**

- Langkah terakhir yaitu men-dekripsi ciphertext dengan rumus **m = c^d mod n**

