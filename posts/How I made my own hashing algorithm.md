## How I made my own hashing algorithm
### and how you can too

#### Disclaimer: I'm not an expert and it's almost never a good idea to roll your own encryption.

## Introduction
The purpose of making this article is to demystify the basics of hasing algorithm so that you can write better code, by knowing what's going on under the hood of the code you use every day you can better understand where you can seek to improve.


Everything today from encryption to low level error detection uses some sort of hashing algorithm. The purpose of a hashing algorithm is to be a standard length and have a drastically different output for small changes in the input, and most importantly the hash algorithm needs to have the same output each time.

Examples include which you may have seen in day to day computer use such as:
Input: test
MD5: 098f6bcd4621d373cade4e832627b4f6
SHA1: a94a8fe5ccb19ba61c4c0873d391e987982fbbd3

These algorithms may not be very secure but are typically use to verify the integrity of a file. 


There are five components of a hash algorithm
1. Fixed length
2. Convolution
3. Padding
4. Displacement
5. And non-reversible

And some performace considerations are
1. Preallocation -- It's recommended that padding buffers should be preallocated on initialization
2. Precalculation -- You definitely want to reduce the amount of math you're doing on each iteration by pre calculating constants


## Preface

1. Choose a programming language with good bitwise operator support, in c/c++ these symbols look like this
```c
<< Left Shift
>> Right Shift
^ XOR
& AND
| OR
```
Some programming languages only offer functions for bitwise shifting which can be a little tedious.

2. It's recommened that you also have a basic understanding of binary and hexadecimal.


## Bitwise operators
When we're working with binary data we typically work in units of bits, nibbles, bytes, words, dwords, qwords
Here's what those units look like
```
Bit 0 or a 1
Nibble is four bits ex: 0101 or 4 bits
Bytes are two nibbles or ex: 0101 0101 or 8 bits
Words are two bytes or ex: 01010101 01010101 or 16 bits
Dwords are two Words or ex: 0101010101010101 0101010101010101 or 32 bits
Qwords are two Dwords or ex: 01010101010101010101010101010101 01010101010101010101010101010101 or 64 bits
... so on and so forth
```

When we perform a bitwise operation on a number it's typically done with a constant as a operand
like this, for this blog i'll be grouping numbers into nibbles just to make it easier to read.

#### Masking
There's certain situations where you would want to mask off a portion of the bits, below is how you would achieve this
```
0101 0101
&
0000 1111
_________
0000 0101
```
Typically these are expressed as hexidecimal numbers but for the purposes of demostration ,i'll be doing it this way, below i'll show you how this would be writtin in c
```c
uint8_t mask(uint8_t num){
    return num & 0xff; // 0xff = 0000 1111
}
```
This essentially turns any number into a 4 bit number

#### Shifting
Bit shifting is probably the most intuitive operation, as we are simply taking each bit and shifting it over x number of positions, whe we talk abouut convolutions this will be important
a single left shift will look something like this
```
0101 0101
<<
0000 0001
_________
1010 1010
```
In this case the operand is the number of times it's shifted and this is how you would do this in c
```c
uint8_t shiftLeftOne(uint8_t num){
    return num << 1
}
```

#### XOR
This is probably the most important operator when hashing
On a one bit level, the xor works by taking two inputs and cancelling them out if they are both equal to 1 or letting one through if only one input it 1
We can make a simple inversion by doing this
```
0101 0101
1111 1111
_________
1010 1010
```
Xor is a great operation of scrambling the input of each byte
but this as a hash algorithm wouldn't be great because it almost breaks all the rules.


#### Padding
Adding padding to your hash algorithm can be a great way to ensure that your input is over a certain length that makes the convoluions work better.
Constants are a great way to add what is a essentially predictable noise to your input, we're not going to go over calculations required to produce these constants, for our purposes we will just predefine constants of our own.
And our target buffer will be 16 bytes and our desired output will be 8 bits

Our constants will be:
```
1111 0000
1100 0011
1010 1010
0101 0101
```

When we want to pad a input we do so by first turning our input into bytes, we will convert the word 'test' to bytes so we can work with it
```
for the word test we get 0x74 0x65 0x73 0x74
```
Converting this to binary so we can see what's going on gets us:
```
t = 0111 0100
e = 0110 0101
s = 0111 0011
t = 0111 0100
```
If we simply take this set of bytes and add an extra 12 bytes(the difference between our buffer size and input bytes) of padding from our constants we get this
```
0111 0100 |
0110 0101 |
0111 0011 |
0111 0100 +-- Our original input

1111 0000
1100 0011
1010 1010
0101 0101

1111 0000
1100 0011
1010 1010
0101 0101

1111 0000
1100 0011
1010 1010
0101 0101
```

#### Convolutions
The simpliest convolution you can do on this would to simply add all the bytes together and let them over flow
```c
int main(){
  uint8_t hash = 0x00;
  uint8_t padding[4] = {
    0b11110000,
    0b11000011,
    0b10101010,
    0b01010101
  };
  const uint8_t strsize = 4; // for example sake only
  uint8_t buffer[16] = {
    0x74, // t
    0x65, // e
    0x73, // s
    0x74, // t
  };
  for(int i = strsize; i < 16; i++){
    buffer[i] = padding[i % 4];
  }

  for(int i = 0; i < 16; i++){
    printf("%b\n", buffer[i]); // Outputing the buffer prior to hashing
  }

  for(int i = 0; i < 16; i++){
    hash += buffer[i]; // naive hash attempt
  }

  printf("Hash: %b\n", hash);
  return 0;
}
```

The biggest problem with this approach is that small changes in the put do not make drastic changes to the output, the output of this hashing algorithm for the word test looks like this

```
1101 0101 for the word 'test'

if I change the first character to a 0x75 which is a single bit flip, the output is

1101 0111 for the word 'uest'
```

So this can't be a great hashing algorithm.


A better approach would be to add multiple operations to the hash and looking ahead in the buffer

In this example below we are looking ahead 3 bytes and adding the number at that location and if it goes over the buffer size it will rotate around the buffer

in the second convolution we are XORing 7 bytes ahead and rotating when it's higher than our buffer size. when the number of padding constants is even you want to choose odd offsets.

```c

for(int i = 0; i < 16; i++){
    hash += buffer[(i + 3 ) % 16]; // looking ahead 3 bytes and adding
    hash = hash ^ buffer[(i + 7) % 16]; // looking ahead 7 bytes and XORing
}
```
```
This outputs 1000 0010 for test
and a totally different output for uest(with just a single bit change) 1110110

```
You can immediately see a drastic change in the output. You may be starting to see how this can be expanded to something like sha256, similar concepts but their constants are generated externally to ensure the maximum amount of entropy. Here's a links to a great resource for example code for these types of modern hashing algorithms.

(crypto algorithms)[https://github.com/B-Con/crypto-algorithms/tree/master]


Thanks for reading my blog, if you have any questions, feel free to post an issue.




