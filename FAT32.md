
  <img width="469" height="700" alt="image" src="https://github.com/user-attachments/assets/f4340d79-5be9-46eb-9f8a-04888adc121b" />




1.Examine the FAT32 Structure

 ->FAT32 uses two copies of the File Allocation Table (FAT1 and FAT2) for redundancy. Each
entry is 4 bytes (32-bit), representing the next cluster in a chain.
```
hexdump -s 0x0E -n 2 -C blackhole.img
```

  
<img width="622" height="71" alt="FAT32-1" src="https://github.com/user-attachments/assets/e044f0ea-5b64-4d4d-879c-57bf7fc2d985" />


  
   ->00 10 in little-endian is 4096. You’ve successfully broken the fundamental "map" of the
disk.
```
hexdump -s 0x28 -n 2 -C blackhole.img 
```
   
   
   <img width="729" height="80" alt="FAT32-2" src="https://github.com/user-attachments/assets/96002cba-d687-4ef2-8e47-34a43dd2e1ef" />


   
   
   ->The high bit (0x80) tells the filesystem driver: "Do not mirror FAT tables; use the one
specified in the lower bits (FAT2)."

     ->The Problem: The "Bytes Per Sector" at offset 0x0B is set to 4096.

 ->The Fix: They must change 00 10 back to 00 02 (512 bytes). This is the only way to align
the file system so it can be mounted.
  
  2.Protocol Deep Dive (The Red Herring)
  ->SYSTEM CORRUPTED. CLUSTER PARITY MISMATCH AT FAT2.....
  
  Go to Offset 0x28: This is the ExtFlags field. You see 80 00.
  
  Consult the Specs: The 0x80 bit means "FAT mirroring is disabled" and "FAT2 is the
active table."
  
  Conclusion: Most people look at FAT1 (offset 0x4000). You must look at FAT2 (offset0x8000).
  
  3.Pattern Recognition (The Stego)
 
 Navigate to the start of FAT2 at 0x8000. You will see a list of 4-byte entries (pointers).

   ```
  Look at the Pointers:
  
  Entry 1: 0x00000005 (Binary ends in 1)
  
  Entry 2: 0x00000006 (Skip)
  
  Entry 3: 0x00000007 (Skip)
  
  Entry 4: 0x00000004 (Binary ends in 0)

  ```


  The Pattern: The Least Significant Bit (LSB) of the pointers is fluctuating, but only every
3rd entry. This is a Bit-Stream.

  4.Understand the Data Hiding Scheme

 ->FAT32 uses two copies of the File Allocation Table (FAT1 and FAT2) for redundancy. Each
entry is 4 bytes (32-bit), representing the next cluster in a chain.

 ->The challenge hides data by encoding bits in the LSB (Least Significant Bit) of FAT
entries — a classic bit-flip steganography technique.
 
 ~FAT1 (offset 0x4000): Contains a decoy flag encoded in LSBs, one bit per entry,
sequentially.

 ~FAT2 (offset 0x8000): Contains the real flag encoded in LSBs, but with Step-3 skipping —
after reading each entry, 2 entries (8 bytes) are skipped.
 
 ->The first 32 bytes (8 entries) of each FAT region are skipped, as they contain reserved
FAT metadata.
   
   Solution Script :-
   
```
def solve():
filename = "blackhole.img"
fat2_offset = 0x8000 
 with open(filename, "rb") as f:
# Start of FAT2 data entries
f.seek(fat2_offset + 32) bits = ""
# Increase this to 512 to ensure we get the full tail end
for _ in range(512):
chunk = f.read(4)
if not chunk: break
 pointer = int.from_bytes(chunk, 'little')
bits += str(pointer & 1)
 # The "Step-3" logic: skip 2 entries (8 bytes)
f.seek(8, 1)
  flag = ""
for i in range(0, len(bits), 8):
byte = bits[i:i+8]
if len(byte) == 8:
char_code = int(byte, 2)
# Stop if we hit a null byte or non-printable garbage
if char_code == 0: break 
flag += chr(char_code)
 print(f"\n[+] Full Flag: {flag}")
 solve()
``` 
 
 
 <img width="587" height="55" alt="FAT-Flag" src="https://github.com/user-attachments/assets/290c229f-30c0-4b67-a0f7-84275695a3cf" />


 
  The Flag:- tomctf{y0ur_ar3_b1t_fl1p_f0r3n51c5_m45T3r}
