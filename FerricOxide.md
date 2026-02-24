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


->The is the core logic for the challenge.

    sub_10E63(v137);
    v128 = v137[0]; 
    v116 = v137[1]; 
    v138[0] = v137[2];
    for ( j = 0; j != 16; ++j )
      *((_BYTE *)v137 + j) = *((_BYTE *)v138 + j) ^ *((_BYTE *)&v116 + j) ^ byte_3110[j] ^ *((_BYTE *)&v128 + j);


Trap the "Access Denied" print
      
    catch syscall write

    find /g $rsp, +0x4000, 0x1111111111111111


Dump the real key!
Take the stack address GDB just gave you, add 0x60 to it, and read 16 bytes (two 64-bit words). Replace <address> with your found address:

    0x55555556580d  cmp    dl, BYTE PTR [rsp+rcx*1+0xe0]


<img width="1069" height="425" alt="image" src="https://github.com/user-attachments/assets/7c07865c-f42d-49b9-abdf-d37e2ec68b7a" />


->The real key is founded 

```
Target Key	Mask	XOR Result (Your Input)
0x9b	0x3b	a0
0x71	0x4a	3b
0xda	0x72	a8
0x28	0xfa	d2
0xb6	0xda	6c
0x08	0xb8	b0
0xb5	0x63	d6
0xf9	0x39	c0
0x6f	0x0a	65
0x84	0xfa	7e
0xd4	0xb0	64
0x94	0x46	d2
0x88	0x9c	14
0x39	0x14	2d
0xd5	0x73	a6
0x3c	0xdd	e1

```


The Flag:-



<img width="750" height="171" alt="image" src="https://github.com/user-attachments/assets/13db72cf-55a9-448c-b37d-8f59369c489d" />



  *Flag:```tomctf{th3_ru5t_4lw4y35_c0mpl1c4t3d_but_345y_t0_f1nd}```*
