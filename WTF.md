1.Intro

<img width="450" height="479" alt="image" src="https://github.com/user-attachments/assets/2f3d0844-9ffc-4a09-86bc-db6f18539ba9" />


This is the challenge entitled "Won the Fate" from TomCTF 2026.


<img width="627" height="210" alt="image" src="https://github.com/user-attachments/assets/22ba38fb-c125-4bd2-b82e-8b5847bb8708" />


2.Identification

A file they given named "wtf" we can identify that file by using strings :-

<img width="824" height="928" alt="image" src="https://github.com/user-attachments/assets/00b95e45-033c-4c7c-85b4-c5896a3727af" />

->By this we can conclude the give executable file is python binary file.

3.Conversition

we can convert the binary code into byte code by a github repo..

```https://github.com/extremecoders-re/pyinstxtractor```

<img width="781" height="256" alt="image" src="https://github.com/user-attachments/assets/a3782484-7ff9-478a-b0f6-b5dbf2944ed6" />

-> We can find the actual byte code of the given binary file

<img width="1718" height="412" alt="image" src="https://github.com/user-attachments/assets/81c69eb9-2025-433e-877b-36a76d20cc47" />
<img width="1701" height="907" alt="image" src="https://github.com/user-attachments/assets/7129dfa5-79ea-4aef-9571-618cbec5a2f5" />

4.Getting Source

-> We can easily get the actual source code by pylingual

```https://pylingual.io/```

<img width="1896" height="677" alt="image" src="https://github.com/user-attachments/assets/f6a6be2a-a8b6-4c6d-a1ed-f4abfb8e4f0f" />


->After decoding the parts of the flag we can get the flag

Flag:-```tomctf{h3y_1t_15_n0t_61ffcult_t0_f1n6_4n6_und3r5t4nd_re}```
