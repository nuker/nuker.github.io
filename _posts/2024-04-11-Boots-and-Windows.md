---
layout: post
title: "Boots and Windows"
---

# Disecting the Windows MBR

> Reading on [ired team](https://www.ired.team/miscellaneous-reversing-forensics/windows-kernel-internals/writing-a-custom-bootloader) they talk about writing your own boot secotor which is interesting
they also show that the first 512 bytes of your bootable hard drive (sector 0) is the bootloader

# PLACE IMAGE

> We can copy those bytes from the hex dump and paste them into a text file like this

```
// bytes from sector0
EB52904E5446532020202020000208000000000000F80000
003F00FF0000A8030000000000800080008CCA5C74000000
000000000C00000000000002000000000000F60000000100
0000962C5EFC645EFC6800000000FA33C08ED0BC007CFB68
C0071F1E686600CB88160E0066813E03004E5446537515B4
41BBAA55CD13720C81FB55AA7506F7C101007503E9DD001E
83EC18681A00B4488A160E008BF4161FCD139F83C4189E58
1F72E13B060B0075DBA30F00C12E0F00041E5A33DBB90020
2BC866FF06110003160F008EC2FF061600E84B002BC877EF
B800BBCD1A6623C0752D6681FB54435041752481F9020172
1E166807BB166852111668090066536653665516161668B8
0166610E07CD1A33C0BF0A13B9F60CFCF3AAE9FE01909066
601E0666A111006603061C001E6668000000665006536801
00681000B4428A160E00161F8BF4CD1366595B5A66596659
66591F0F82160066FF06110003160F008EC2FF0E160075BC
071F6661C3A1F601E80900A1FA01E80300F4EBFD8BF0AC3C
007409B40EBB0700CD10EBF2C30D0A41206469736B206561
642
```

> Now we can convert it to a bin file and try to do some gdb analysis

```bash
cat sector0 | xxd -r -p - > sector0.bin
```

> "-r" =  reverse operation: convert (or patch) hexdump into binary.
> "-p" = output in postscript plain hexdump style.
> "-" = take from stdin "cat sector0 |"
