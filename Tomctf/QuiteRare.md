# Quite-Rare

I had a flag on this handy QR code, but a fiend cut it up into pieces and threw one of them away! Is there still a way to retrieve the flag?

Advanced 
350 points 

We have three segments of a qr code with one segment missing

heres the basic structure of a qr code:


<img width="800" height="500" alt="image" src="https://github.com/user-attachments/assets/5fb5c37f-a882-4ef3-8609-d3747a7cf63c" />


I also found this website that tells you how to read QR codes manually:
https://qr.blinry.org/

this was my original plan but I couldn't figure out how to apply the mask pattern and sought other methods.

now for every qr code the alignment pattern(the big squares on three sides) and the timing pattern(alternating stripes of black and white) do not change.
so lets try to reassmble the qr code and fill those in.
heres the result:

<img width="454" height="438" alt="image" src="https://github.com/user-attachments/assets/e52cac6d-ab86-4490-8fa9-bb9a434c0e79" />



Now the version information can also be recovered because the bits repeat and after all that I get this:

<img width="342" height="336" alt="image" src="https://github.com/user-attachments/assets/2969fb3a-31e7-4bf3-8c9f-aa706a89058d" />
<img width="847" height="485" alt="image" src="https://github.com/user-attachments/assets/aac5b9fa-f7bd-4aa6-94f1-768ec8810b56" />

The tools online that could perform qr code analysis and recovery, which lead me to this site:
https://merri.cx/qrazybox/

lets open the qr we created,
select tools->Extract QR Information

`Decoded data : tomctf{QR_c0d3s_4r3_cr4zy} YOU WON!`

and we have our flag!

`flag: tomctf{QR_c0d3s_4r3_cr4zy}`
