title: "BitHacks"
date: 2015-03-07 21:22:30
categories: Algorithm
tags: BitHacks
---

I've recently come across a very interesting article on [BitHacks](https://graphics.stanford.edu/~seander/bithacks.html) -- the low level magics for bit level operations. Some of the tricks introduced here are really excellently clever, some of them may even make you exclaim for their genius!

I had a lot of fun reading through some of the BitHacks. It's also worth noticing these BitHacks are not only for intellectual pleasures, they provide actual boosts to algorithm performance as well. When an operations is used often enough, the overall performance benefits to the whole program might be significant.

I couldn't help but keep wondering how on earth did these clever CS guys ever come up with such algorithms. I tried very hard to find some answers and the following are some patterns I noticed in this attempt. Still, honestly, I highly doubt if I can come up with same solutions myself if I ever run into these problems again. Some of them are just to clever.


<!--more-->

## Some examples

* [Compute the sign of an integer](https://graphics.stanford.edu/~seander/bithacks.html#CopyIntegerSign)

The beauty part of bit hackings is that most of the times they avoid branching in CPU, which could be expensive for modern pipelining CPUs, as a misprediction in branching means flushing operations, causing waste in time and power.

For example, to compute the sign of an integer, instead of using branching, we can use the following code:

```c
int v;       // the integer
int sign;    // the sign of the integer

// CHAR_BIT is the number of bits per byte (normally 8). 
// But for compatibility issues, here uses CHAR_BIT instead.

sign = - (v < 0);   // sign = -1 when v is negative

// return -1 when v is negative, 0 when 0, and 1 when positive:
// cast v to be unsigned, right shift the sign bit to the LSB, cast it back
// to integer, and assign the sign.
sign = (v != 0) | -(int)((unsigned)((int v) >> (sizeof(int) * CHAR_BIT - 1));

// Or for better speed
sign = (v > 0) - (v < 0);
```

* [Determine if integer is power of 2](https://graphics.stanford.edu/~seander/bithacks.html#DetermineIfPowerOf2)

```c
// One interesting feature of an integer to the power of 2:
// v & (v - 1) == 0
// 0 is also incorrectly considered to be a power of 2 with
// the above equation, but the fix is simple

f = v && !(v & (v - 1));
```


* [Find the min or max of two integers](https://graphics.stanford.edu/~seander/bithacks.html#IntegerMinOrMax).

The effect shall be the same as

```c
max = x > y ? x : y;
min = x < y ? x : y;
```

Somehow the above approach also use branch to determine value. A BitHack way is to:

```c
result = x ^ ((x ^ y) & -(x < y));
```

Amazing! Isn't it? Here when x < y, -(x < y) evaluates to -1, which is all 1s in binary representation. The result will then evaluates to:

```c
x ^ (x ^ y)
```

Which is y. While when x > y, -(x < y) evaluates to 0, and the result will be assigned x.

This is a very interesting feature of the XOR operation. Remember how XOR could be used to exchange the value of two numbers without extra memory:

```c
a = a ^ b;
b = a ^ b;  // (a ^ b) ^ b
a = a ^ b;  // (a ^ b) ^ a

// Here a and b are exchanged
```

XOR has many interesting features, and is a very important operations in BitHacks. Here's another example.

* [Absolute value of an integer](https://graphics.stanford.edu/~seander/bithacks.html#IntegerAbs)

```c
mask = v >> (sizeof(int) * CHAR_BIT - 1);

result = (v + mask) ^ mask;
```

When v is positive, mask is 0, result will be assigned v. And when v is negative, mask will evaluate to -1 (all 1s in binary), and the result will be (v - 1) ^ (-1). As XORing all 1s gives the NOT of an integer (v ^ -1 == ~v), the result becomes ~(v - 1). Not surprisingly, this is the negative value of v.


An alternative but similar approach is:

```c
result = (v ^ mask) - mask;
```


## Some Observations

I believe some general guidelines could be useful for inventing our own BitHacks could be useful, but the post did not mention any special techniques, and some of the hacks seems really ad-hoc (sure, that's why they're called "hacks", right?). Nevertheless, the following is some observations I had when reading. Keeping these in mind might help the next time when inventing BitHacks.


### Use AND, OR, shift and masking

Use AND to clear the bit field, OR to set the bit field, and use shift to move important bit to the right position. Use a carefully designed mask to clear or set the fields.

An excellent example could be found in [here: counting bit set, in parallel](https://graphics.stanford.edu/~seander/bithacks.html#CountBitsSetParallel).

In the example, instead of iterating all the bit fields in the given integer, we could use a mask to mask the fields, and merge the all the counts, a bit like reduce in the map-reduce. In this way, no loops are required to count the bit sets.

The operations could be explained as follows:

```c
// first, I randomly pick a 32-bit integer to count bits set
// value: 0110 1000 1011 0100  0101 1001 1011 1110

// MASK1: 0101 0101 0101 0101  0101 0101 0101 0101
//         1 0  0 0  0 1  1 0   1 1  0 1  0 1  1 0
// +       0 1  1 0  1 1  0 0   0 0  1 0  1 1  1 1
// c   :  0101 0100 0110 0100  0101 0101 0110 1001
// The idea is to find the bit counts for every two bits.
// To achieve so, using mask and shift, we can shift the value
// 1 to the right and add the masked original value together,
// as following:

c = ((v >> 1) & MASK0) + (v & MASK0);

// Although here the post uses a faster way;
// c = v - ((v >> 1) & MASK0);

// Then add the counts for each 4 bits. The mask becomes:
// MASK1: 0011 0011 0011 0011  0011 0011 0011 0011

c = ((c >> 2) & MASK1) + (c & MASK1);

// After this operation, then find the count in every 8 bits,
// and every 16 bits, and every 32 bits. The final count would
// be what we want.

// MASK2: 0000 1111 0000 1111  0000 1111 0000 1111
// MASK3: 0000 0000 1111 1111  0000 0000 1111 1111
// MASK4: 0000 0000 0000 0000  1111 1111 1111 1111

c = ((c >> 4) & MASK2) + (c & MASK2);
c = ((c >> 8) & MASK3) + (c & MASK3);
c = ((c >> 16) & MASK4) + (c & MASK4);

```

### XOR operation

The XOR operation has many interesting features which could be used cleverly in BitHacks.

```c
int value, other;

value ^ 0 == value;
value ^ 0xFFFFFFFF == ~value;
value ^ (-1) == ~value;           // same as above
value ^ value == 0
value ^ (value ^ other) == other;
```

The previous, and some following examples all uses these features to efficiently "hack the bits". For example: exchanging values without extra memory, finding absolute value, finding the min or max of two values.


### Two's complements

In the previous example: Absolute value of an integer, a mask was used to conditionally find the original and the negative value, by setting the mask to either 0 or all 1s, which is -1 in integer representation. The negative value of v could be found by very simple operations:

```c
v = ~v + 1

// here to find the value of ~v, one could use v ^ -1.
// And since (v ^ 0 == v), we can easily come up with
// the solution to find the abs of v.

v = (v ^ mask) - mask

// or its alternative, as described previously
v = (v + mask) ^ mask
```

Taking advantage of this property, we can also come up with a way to [conditionally negate a value](https://graphics.stanford.edu/~seander/bithacks.html#ConditionalNegate):

```c
result = (v ^ -flag) + flag;
```


### Lookup table

A lot of complicated operations could be accomplished by using lookup tables, one example could be the counting bits set as above. Some other examples includes:

* [Compute parity by lookup table](https://graphics.stanford.edu/~seander/bithacks.html#ParityLookupTable)
* [Reverse bits lookup table](https://graphics.stanford.edu/~seander/bithacks.html#BitReverseTable)
* [Find the log base 2 of an integer with a lookup table](https://graphics.stanford.edu/~seander/bithacks.html#IntegerLogLookup)


Although it might take up more space for memory, it's often worthwhile to trade some amount of memory for speed. Memory operations may be more expensive, but considering prefetching is now prevalent in modern CPUs, fetching data from the cache is way faster than a misprediction in branch operations. 


## Afterthoughts

There are many other interesting and mind-opening techniques, tricks and hacks to speed up your program in [this post](https://graphics.stanford.edu/~seander/bithacks.html#CopyIntegerSign). I have a hunch that I might actually use some of them in the future, or come up with my own BitHacks with similar mindset. Before this, I didn't even realize I could play with bit operations in C/C++ this way. Also, [this post](https://graphics.stanford.edu/~seander/bithacks.html#CopyIntegerSign) could serve as a great reference when a performance critical part needs optimization with BitHacks.
