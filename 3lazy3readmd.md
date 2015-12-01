## Motivation

Yesterday, I downloaded &pi;. Not all of it, just the first 100 billion digits. I downloaded digits of &pi; from [piworld], where the digits are available in zipped text form like this:

    1415926535 8979323846 2643383279 5028841971 6939937510 5820974944 5923078164 0628620899 8628034825 3421170679 : 1
    8214808651 3282306647 0938446095 5058223172 5359408128 4811174502 8410270193 8521105559 6446229489 5493038196 : 2
    4428810975 6659334461 2847564823 3786783165 2712019091 4564856692 3460348610 4543266482 1339360726 0249141273 : 3
    7245870066 0631558817 4881520920 9628292540 9171536436 7892590360 0113305305 4882046652 1384146951 9415116094 : 4
    3305727036 5759591953 0921861173 8193261179 3105118548 0744623799 6274956735 1885752724 8912279381 8301194912 : 5
    9833673362 4406566430 8602139494 6395224737 1907021798 6094370277 0539217176 2931767523 8467481846 7669405132 : 6
    0005681271 4526356082 7785771342 7577896091 7363717872 1468440901 2249534301 4654958537 1050792279 6892589235 : 7
    4201995611 2129021960 8640344181 5981362977 4771309960 5187072113 4999999837 2978049951 0597317328 1609631859 : 8
    5024459455 3469083026 4252230825 3344685035 2619311881 7101000313 7838752886 5875332083 8142061717 7669147303 : 9
    5982534904 2875546873 1159562863 8823537875 9375195778 1857780532 1712268066 1300192787 6611195909 2164201989 : 10

This format is horribly inefficient, so the first thing I did was converting the digits to packed [BCD], cutting down space usage from 54635&thinsp;MiB to just 47683&thinsp;MiB. That's closer to the theoretical optimum of 10<sup>11</sup> &middot; log<sub>256</sub>10 &approx; 41524101186&thinsp;B or about 39600 MiB but still pretty far away. Disk space is cheap, but wasting 8&thinsp;GiB to inefficient encoding is something I don't like.

A more efficient encoding scheme for decimal digits called *[densely packed decimal]* exists. It is based on the observation that you can safe digits from 0 to 7 directly in three bits and digits 8 and 9 in one bit plus a flag bit to denote the high digit. This allows you to encode three decimal digits into 10&nbsp;bits.

With DPD, we can encode 12&nbsp;digits in 5&thinsp;B as opposed to the 10&nbsp;digits we can encode with packed BCD. This reduces the number of bytes we need to encode 100 billion digits of &pi; from the aforementioned 47683&thinsp;MiB to just 41666666670&thinsp;B &approx; 39736&thinsp;MiB which is much closer to the theoretical optimum.

## The DPD encoding scheme

Three decimal digits &alpha;, &beta;, &gamma; with Boolean representations &alpha; = abcd, &beta; = efgh, &gamma; = iklm are encoded into 10 bits as follows:

    -- output --   α   β   γ
    bcd fgh 0klm  0-7 0-7 0-7
    bcd fgh 100m  0-7 0-7 8-9
    bcd klh 101m  0-7 8-9 0-7
    kld fgh 110m  8-9 0-7 0-7
    kld 00h 111m  8-9 8-9 0-7
    fgd 01h 111m  8-9 0-7 8-9
    bcd 10h 111m  0-7 8-9 8-9
    00d 11h 111m  8-9 8-9 8-9

four groups A, B, C, D of three encoded digits each shall be packed into five bytes like this:

    AAAAAAAA AABBBBBB BBBBCCCC CCCCCCDD DDDDDDDD

If the input doesn't contain a multiple of 12 decimal digits, the remaining digits shall be assumed to be 0. For example, the digits 248163264128 are split into the triplets 248 163 264 128 which are encoded as:

    248 => 010 100 1000
    163 => 001 110 0011
    264 => 010 110 0100
    128 => 001 010 1000

which is packed into bytes like this:

    01010010 00001110 00110101 10010000 10101000

yielding the bytes 0x52, 0x0E, 0x35, 0x90 0xA8.

## The challenge

Your goal is to write a program that receives decimal digits in packed BCD from standard input, converts these to DPD and writes the result to standard output. You may assume that the input is valid. The fastest program, measured as described in the section *measurement*, wins.

## Input files

I have a file `pi.bcd` containing the first 100 billion decimal digits of &pi; in packed BCD format. The file has the SHA 256 hash 

    1c20e1a8e20e8f73d7da413ef799d5d9c8e3d3b5281fe572a630898f63986940

and can be created from the digits of &pi; you can get from [piworld]. It begins like this:

    00000000  14 15 92 65 35 89 79 32  38 46 26 43 38 32 79 50  |...e5.y28F&C82yP|
    00000010  28 84 19 71 69 39 93 75  10 58 20 97 49 44 59 23  |(..qi9.u.X .IDY#|
    00000020  07 81 64 06 28 62 08 99  86 28 03 48 25 34 21 17  |..d.(b...(.H%4!.|
    00000030  06 79 82 14 80 86 51 32  82 30 66 47 09 38 44 60  |.y....Q2.0fG.8D`|
    00000040  95 50 58 22 31 72 53 59  40 81 28 48 11 17 45 02  |.PX"1rSY@.(H..E.|
    00000050  84 10 27 01 93 85 21 10  55 59 64 46 22 94 89 54  |..'...!.UYdF"..T|
    00000060  93 03 81 96 44 28 81 09  75 66 59 33 44 61 28 47  |....D(..ufY3Da(G|
    00000070  56 48 23 37 86 78 31 65  27 12 01 90 91 45 64 85  |VH#7.x1e'....Ed.|
    00000080  66 92 34 60 34 86 10 45  43 26 64 82 13 39 36 07  |f.4`4..EC&d..96.|
    00000090  26 02 49 14 12 73 72 45  87 00 66 06 31 55 88 17  |&.I..srE..f.1U..|

I do not make this file available due to its humongous size. A set of shorter files containing [1000], [100000], [10000000], and [1000000000] digits of Pi is available from my website for testing. Their SHA 256 checksums are:

    054944fdbab4a1d839b0afa808955c126c0ee35b794d4181b027b973f7610911  pi-1000000000.bcd
    8e919fc72b411fa8caf1770d89573852cf43e0951bfd72c6959943665fb60f49  pi-10000000.bcd
    a000f88f52aa39a2b8f5ad442dd8da42610f93a4a6465b39066a891a659cecae  pi-100000.bcd
    d49e1549fc49d106d5f33681f34c68b613d2928551572f7219b0e2c7e0441f9e  pi-1000.bcd

## Measurement

Your code is going to be measured on my personal computer on the 47 GiB `pi.bcd` file. It has 32 GiB of RAM, an Intel Core i7 4910MQ CPU and runs amd64 FreeBSD 10. You are not allowed to use more than one thread or process. Please provide a program that either runs on FreeBSD 10 or on i386 Linux using the [Linuxulator].

Data is going to be fed from an SSD and written to `/dev/null` (after doing one run to check if your program works correctly). I'm going to compensate for the time it takes to read the file from disk by only counting time spent by your program (i.e. `user` time).


[piworld]: http://piworld.calico.jp/epivalue1.html
[BCD]: https://en.wikipedia.org/wiki/Binary-coded_decimal
[densely packed decimal]: https://en.wikipedia.org/wiki/Densely_packed_decimal
[1000]: http://fuz.su/~fuz/pi/1000.bcd
[100000]: http://fuz.su/~fuz/pi/100000.bcd
[10000000]: http://fuz.su/~fuz/pi/10000000.bcd
[1000000000]: http://fuz.su/~fuz/pi/1000000000.bcd
[Linuxulator]: https://www.freebsd.org/doc/handbook/linuxemu.html
