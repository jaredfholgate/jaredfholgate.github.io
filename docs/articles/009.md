---
layout: page
title: Back to Basics - Take a Nibble or a Byte?
description: A back to basics article on the binary data types and ways to represent them.
date: 2021-01-01T18:50:00
---

This article covers some of the basics of data types. It is mainly a method for me to refresh my knowledge, but hopefully someone finds it a useful reference.

## Bases

A base is a method of encoding numbers for use in maths or coding. The numbers we use every day to count are base-10. Other base types are generally used to make binary (base-2) into a compressed, readable string.

Binary (base-2) is the language used by computers, which can only support 1 and 0 (on or off) at the lowest level.

- Base-2
  - Binary (0 and 1)
  - e.g. 00010111
- Base-8
  - Octal (0 - 7)
  - e.g. 27 05 20
- Base-10
  - Decimal (0 - 9)
  - e.g. 1 2 3 4 5 6 7 8 9 10 11 12
- Base-16
  - Hexadecimal (0 - 9 and A - F)
  - e.g. 00FF A45D

Each base type can be converted from one to the other.

E.g.

- Base-2: 00010111
- Base-8: 27
- Base-10: 23
- Base-16: 17

Base is often noted in text using a subscript next to the number.

E.g.

- Base-2: 00010111<sub>2</sub>
- Base-8: 27<sub>8</sub>
- Base-10: 23<sub>10</sub>
- Base-16: 17<sub>16</sub>

In C# and other C family languages Hexadecimal is denoted by prefixing with '0x'. 

E.g.

- 0xA46FD97C

### Binary (Base-2)

Binary is a series of bits that a read right to left. Each bit represents a number double the previous size. For example a simple 8 bit sequence (known as an unsigned byte) can represent a number between 0 and 255 by toggling the bits on and off;

```plaintext
0   0   0   0   0   0   0   0   = 0
1   1   1   1   1   1   1   1   = 255
0   0   0   1   0   1   1   1   = 23
_____________________________
128 64  32  16  8   4   2   1
```

### Unsigned

The means it does not support negative numbers. The above exmple being a 8 bit unsigned byte.

### Signed

This means it does support negative numbers, so the 8 bit signed byte supports number between -128 and +127. Signed bits use a system known as 2's completment.

In twos complement, the positive number are represented the same as in an unsigned byte, except the furthest left bit now represents the sign (0 = positive, 1 = negative), for example;

```plaintext
0   0   0   0   0   0   0   0   = 0
0   1   1   1   1   1   1   1   = 127
0   0   0   1   0   1   1   1   = 23
_____________________________
Sig 64  32  16  8   4   2   1
```

Then for negative numbers, they again count up, but start from a lower base examples include;

```plaintext
1   1   1   1   1   1   1   1   = -1
1   0   0   0   0   0   0   0   = -128
1   1   1   0   1   0   0   1   = -23
_____________________________
Sig 64  32  16  8   4   2   1
```

### Binary Lengths

Common named binary lenghts include;

- 1 = Bit
- 4 = Nibble
- 8 = Byte
- 16 = Short Word
- 32 = Word
- 64 = Long Word

### Bytes

Examples of bytes in everyday use are;

IPv4 Address. Each of the 4 segments (known as an Octet) is an 8 bit unsigned byte with values between 0 and 255. These are written in base-10 format, separated wit a dot.

E.g.

```plaintext
255.255.255.255
192.168.0.1
10.0.1.5
8.8.8.8
```

### Short Word

Examples of short words in everyday use are;

IPv6 Address. Each of the 8 segments is a 16 bit unsigned word, encoded as hexadecimal and separated with a colon.

E.g.

```plaintext
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```

### Word

Examples of words in everyday use are;

The default integer type (int or Int32) in C# is a 32 bit signed word. This allows values between -2147483648 and 2147483647.

### Long Word

Examples of words in everyday use are;

The default long integer type (long or Int64) in C# is a 64 bit signed word. This allows values between -9223372036854775808 and 9223372036854775807.

### Octal (Base-8)

Octal is used less than Hexadecinal in computing. Octal is a method to encode bits as a string. Octal breaks up the bits in segments of 3 bits. If the final segment is not a full 3 bits, it is just padded with 0's. If the first 3 bit segment is all zeros, it can be ignored.

E.g.

```plaintext
11 101 111 becomes 011 101 111
00 010 111 becomes 010 111
```

To convert binary to Octal, encode each 3 bit segment to a number between 0 and 7. 

```plaintext
000 = 0
001 = 1
010 = 2
011 = 3
100 = 4
101 = 5
110 = 6
111 = 7
```

E.g.

```plaintext
011 101 111 = 357

0   1   1 | 1   0   1 | 1   1   1  =  3 | 5 | 7
_________________________________
4   2   1 | 4   2   1 | 4   2   1

010 111 = 27
```

### Hexadecimal (Base-16)

Hexadecimal is another method to encode bits as a readable string. Hexadeimal uses a Nibble (4 bits) as the segment size. The first nibble is padded with 0's if it is not a full 4 bits. It can be ignored if it is all 0's.

E.g.

```plaintext
1 1111 1111 becomes 0001 1111 1111
00 1110 1101 becomes 1110 1101
```

Hexadecimal uses the numbers 0-1 and the letters A-F to represent 16 digits. So for a Nibble, each converts as follows;

```plaintext
0000 = 0
0001 = 1
0010 = 2
0011 = 3
0100 = 4
0101 = 5
0110 = 6
0111 = 7
1000 = 8
1001 = 9
1010 = A
1011 = B
1100 = C
1101 = D
1110 = E
1111 = F
```

Some examples of Hexadecimal include;

```plaintext
1111 1110 0110 0101 = FE65

1   1   1   1 | 1   1   1   0 | 0   1   1   0 | 0   1   0   1  =  F | E | 6 | 5
_____________________________________________________________
8   4   2   1 | 8   4   2   1 | 8   4   2   1 | 8   4   2   1 


1111 1111 1111 1111 = FFFF
```

Examples of common data types that use hexadecimal include;

- IPv6 Address: 2001:0db8:85a3:0000:0000:8a2e:0370:7334
- GUID: 5a56511c-89f8-4509-9eb0-da3b70c61823
- Colours: #345FFF

## Summary

I hope you found something useful in there. I will add to the article as I remember new bits...
