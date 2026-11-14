# Ferric-Oxide

1.Intro



<img width="464" height="477" alt="image" src="https://github.com/user-attachments/assets/7e4ffbd2-64d6-41a6-b3ca-98c103a41d5c" />


->The flag is Encrypted inside the Executable First we must find the Language by Using File commend:-


<img width="1703" height="58" alt="image" src="https://github.com/user-attachments/assets/e4c375b9-3375-46a3-aa5e-7b3fac895240" />


->We can identify the language of the given file by the Build number : ```BuildID[sha1]=a48d5ba07b0f9da6bc558133d8f2e049abd1f55a```

->Rust binaries can be incredibly dense to reverse engineer because the compiler heavily optimizes the code and bakes in a lot of standard library scaffolding.
->Rust standard library's runtime initialization block (likely std::rt::lang_start_internal or the C entry point that calls it). Before a Rust program ever runs code, it sets up its own environment.

chain of execution for a typical Rust binary on Linux:

    ->_start (The code you just pasted: sets up glibc)

    ->main (The code from your first message: sets up the Rust standard library/runtime)

    ->std::rt::lang_start_internal (Hidden inside the runtime setup)

    ->sub_10F45 (Your actual compiled Rust code and the CTF logic)


1. The Distraction

     At the very top, you see a while ( v0 != 1 ) loop that does some multiplication and division by 2. This is the Collatz conjecture. It does absolutely nothing useful for the program and is just a           simple obfuscation trick to confuse reverse engineers.

2. Input Validation & Hex Decoding

    Length Check: you can see ```if ( v42 - v38 != 32 )```. The program checks that you entered exactly *32 characters*. If you don't, it queues up the error message: *"Error: bad length"*.

    <img width="627" height="132" alt="image" src="https://github.com/user-attachments/assets/77c3d9ea-ea99-4535-92dc-e78ad0d70d2a" />


    Hex to Bytes: Starting around the code as  ```(for ( i = 0; i != 16; ++i ))```, the binary loops 16 times. It reads your 32-character string two characters at a time, validates that they are proper        hex ```(sub_1231A likely checks for 0-9, a-f)```, and converts them into 16 raw bytes. It stores your decoded 16-byte input into the variable v129.

3. The Target Key Generation

     Just before the check happens, the binary dynamically generates the correct key in memory to prevent you from simply running strings on the file:

->The is the core logic for the challenge.
```
    sub_10E63(v137);
    v128 = v137[0]; 
    v116 = v137[1]; 
    v138[0] = v137[2];
    for ( j = 0; j != 16; ++j )
      *((_BYTE *)v137 + j) = *((_BYTE *)v138 + j) ^ *((_BYTE *)&v116 + j) ^ byte_3110[j] ^ *((_BYTE *)&v128 + j);
```

->It loads three 16-byte chunks into memory, grabs a hardcoded array from the .data or .rodata section (byte_3110), and XORs them all together to produce the final 16-byte target key, storing it back into   v137.

4. The Verification Check

    This is the most critical part of the challenge (around line 465):
```
    while ( v68 != 16 )
{
  v23 = (*((_BYTE *)&v132 + v68) ^ *((_BYTE *)&v129 + v68)) == *((_BYTE *)v137 + v68);
  ++v68;
  v69 += v23;
}
if ( v69 != 16 )
{
  // ... Access denied.
```

->It iterates 16 times, comparing your input (v129) against the generated key (v137). If all 16 bytes match (v69 == 16), you pass!

->The decompilation of sub_10F45, we saw these two local variables on the stack:

    v129 (Your decoded input): [rbp-338h]

    v137 (The generated target key): [rbp-2D8h]
    
->The difference between them is exactly 0x60 (96 bytes). If we can find where our dummy input is stored on the stack, the real key will be exactly 0x60 bytes above it!

    
->Trap the "Access Denied" print
      
    catch syscall write
    continue
    11111111111111111111111111111111  //Dummy Data


<img width="595" height="134" alt="image" src="https://github.com/user-attachments/assets/f756285f-85a0-4554-b19a-c2eab1da2d3b" />


->Then find your input

    find /g $rsp, +0x4000, 0x1111111111111111


<img width="1111" height="172" alt="image" src="https://github.com/user-attachments/assets/c1d6cb3c-e6c3-4744-a96b-4152b4cbdbde" />


Dump the real key!
Take the stack address GDB just gave you, add 0x60 to it, and read 16 bytes (two 64-bit words). Replace <address> with your found address:

    0x55555556580d  cmp    dl, BYTE PTR [rsp+rcx*1+0xe0]


<img width="1069" height="425" alt="image" src="https://github.com/user-attachments/assets/7c07865c-f42d-49b9-abdf-d37e2ec68b7a" />


->The real key is founded 

```
Target Key	Mask	XOR Result (Your Input)
  0x9b	    0x3b	        a0
  0x71	    0x4a	        3b
  0xda	    0x72	        a8
  0x28	    0xfa	        d2
  0xb6	    0xda	        6c
  0x08	    0xb8	        b0
  0xb5	    0x63	        d6
  0xf9	    0x39	        c0
  0x6f	    0x0a	        65
  0x84	    0xfa	        7e
  0xd4	    0xb0	        64
  0x94	    0x46	        d2
  0x88	    0x9c	        14
  0x39	    0x14	        2d
  0xd5	    0x73	        a6
  0x3c	    0xdd	        e1

```


The Flag:-



<img width="753" height="153" alt="image" src="https://github.com/user-attachments/assets/bc9e3139-4cfe-498b-801d-71f7123b210e" />



  *Flag:```tomctf{th3_ru5t_4lw4y35_c0mpl1c4t3d_but_345y_t0_f1nd}```*
