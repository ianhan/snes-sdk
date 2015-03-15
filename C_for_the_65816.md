# Introduction #

This page provides you with a couple of hints to help you get the most out of the 65816, which is notoriously hard to generate code for, while programming in C using tcc-65816.

# Data Types #

## Signedness ##

Handling signed types is a chore on the 65816. Use unsigned types wherever possible. If you are familiar with other processors, be informed that the 65816 is exceptionally inept, and handling signed types results in a larger performance impact than you might think.

## Pointers ##

The 65816 has a 24-bit address space divided into 256 banks with 64K of memory each. tcc-65816's pointers are 32-bit types of which the most significant byte is always zero. Pointer arithmetic is 16-bit. This has a couple of consequences:

  1. It is not possible to cross bank boundaries using pointer arithmetic (or array indices, for that matter). In general, data structures crossing bank boundaries are not supported.
  1. Conversion from pointers to scalars loses the bank information. (The only type, apart from pointers, that would be able to contain a complete 65816 long pointer is `(unsigned) long long`, but tcc-65816 is not currently able to convert between pointers and `long long`s. This is a bug, not a feature.)
  1. Conversion from scalars to pointers results in a bank 0 pointer.
  1. Writing a pointer to memory writes 32 bits. This can be a problem when writing to hardware registers or data structures that expect pointers to be 24 bits in size.

One solution to many of these problems is to use a union for pointer conversion:

```
union ptr {
  struct {
    unsigned short addr;
    unsigned char bank;
    unsigned char __pad;
  } c;
  void *p;
};
```

This union type allows easy conversion between pointers and scalars and access to the individual components of a 24-bit pointer.

# Function Parameters #

tcc-65816 passes function parameters on the stack, and - unlike other compilers - it does not align them. This means that you have to be extra careful to make sure that function prototypes are correct, because even minor deviations in type sizes wreak havoc, not to speak of prototypes missing altogether.