

<img width="450" height="735" alt="image" src="https://github.com/user-attachments/assets/f3aa0270-7d71-40dc-b023-0baa6396e0bd" />




  1.Load the WAV File





```
rate, data = wav.read("audio.wav")
data = data.astype(np.float32) / 32767.0
```

->wav.read() returns two things — the sample rate and the raw audio data as integers (16-bit, range −32768 to +32767).

->Dividing by 32767.0 normalizes the data to the range −1.0 to +1.0 as floating point. This makes the math cleaner — the correlation values become consistent regardless of volume.

```for freq in [150000, -150000]:```

->The hidden signal was shifted by −150,000 Hz. But when you save a complex signal as a real-valued WAV, the negative frequency mirrors to positive — so the signal appears at both +150 kHz and −150 kHz in the spectrum.


```
t = np.arange(len(data)) / fs
baseband = data * np.exp(-1j * 2 * np.pi * freq * t)


**`t`** is a time array in seconds:

t = [0, 0.000001, 0.000002, 0.000003, ...]
      ↑           ↑
   sample 0    sample 1 (1/1000000 seconds later)


**`np.exp(-1j * 2 * np.pi * freq * t)`** generates a complex sinusoid at 150,000 Hz:

e^(-j2π·150000·t)
This is a rotating complex number — it spins around the unit circle 150,000 times per second.

**Multiplying** the audio by this sinusoid **shifts every frequency in the signal down by 150,000 Hz**:

Signal at 150,000 Hz  →  moves to 0 Hz (baseband)
Signal at 151,270 Hz  →  moves to 1,270 Hz  ← mark tone appears here
Signal at 151,070 Hz  →  moves to 1,070 Hz  ← space tone appears here
```

2.Alignment Scan

```
for offset in np.linspace(0, samples_per_bit, 20).astype(int):
```

We don't know exactly where the first bit starts in the audio file. Maybe the signal starts at sample 0, maybe at sample 500, maybe at sample 1000.

`np.linspace(0, 3333, 20)` generates 20 evenly spaced starting positions to try:
```
[0, 175, 350, 526, 701, 877, 1052, ..., 3333]

```

->For each starting position, we try decoding the whole signal. If the alignment is right, the flag will appear in the decoded text.


3.Slice Into Bit Chunks

```
for i in range(offset, len(baseband) - samples_per_bit, samples_per_bit):
    chunk = baseband[i : i + samples_per_bit]
    t_chunk = np.arange(len(chunk)) / fs
```

This slides a window through the signal, one bit at a time:
```
Bit 0:  samples[0    → 3333]
Bit 1:  samples[3333 → 6666]
Bit 2:  samples[6666 → 9999]
...and so on
```

`chunk` = 3333 samples representing exactly one bit period.

`t_chunk` = a fresh time array starting from 0 for each chunk:
```
[0, 0.000001, 0.000002, ..., 0.003333]

```
->This is important — the correlation math needs time relative to the start of each chunk, not absolute time in the file.


4.FSK Demodulation (The Core)

```
m = np.abs(np.sum(chunk * np.exp(-1j * 2 * np.pi * mark_f * t_chunk)))
s = np.abs(np.sum(chunk * np.exp(-1j * 2 * np.pi * space_f * t_chunk)))
bits += "1" if m > s else "0"

```
->```np.exp(-1j * 2 * np.pi * 1270 * t_chunk)```
->Sums all 3333 samples:

~If chunk was 1270 Hz → all samples shifted to DC, all positive, they add up to a big number

~If chunk was 1070 Hz → samples oscillate randomly after shift, they cancel out to near zero

```
bits += "1" if m > s else "0"
```
Whichever frequency had stronger correlation wins — that's the bit.

**Visually:**
```
Chunk contains 1270 Hz (mark):
    m = 850.3  (large — frequencies matched, signals added up)
    s =  12.1  (small — frequencies mismatched, signals cancelled)
    → "1"

Chunk contains 1070 Hz (space):
    m =   9.7  (small)
    s = 843.6  (large)
    → "0"
```
5.Convert Bits to Text

```
def bits_to_text(bits):
    chars = []
    for i in range(0, len(bits) - 8, 8):
        byte = bits[i:i+8]
        chars.append(chr(int(byte, 2)))
    return "".join(chars)
```

After decoding, `bits` is a long string like:
```
"0111010001101111011011010110001101110100011001100111..."
```

This function groups it into 8-bit chunks:
```
"01110100" → int("01110100", 2) → 116 → chr(116) → 't'
"01101111" → int("01101111", 2) → 111 → chr(111) → 'o'
"01101101" → int("01101101", 2) → 109 → chr(109) → 'm'
"01100011" → int("01100011", 2) → 99  → chr(99)  → 'c'
...

```

Final:-

```
decoded = bits_to_text(bits)
if "tomctf{" in decoded.lower():
    idx = decoded.lower().find("tomctf{")
    print(f"FLAG: {decoded[idx:].split('}')[0] + '}'}")
    exit()
```

After converting bits to text, search for the flag format `tomctf{`. Once found, slice from that position to the closing `}` and print it.

---

The Flow:-
```
audio.wav (raw 16-bit integers)
    │
    ▼ divide by 32767
Normalized float array [-1.0 to +1.0]
    │
    ▼ multiply by e^(-j2π·150000·t)
Baseband complex signal (FSK tones now at 1270/1070 Hz)
    │
    ▼ slice into 3333-sample chunks
One chunk per bit
    │
    ▼ correlate each chunk vs 1270 Hz and 1070 Hz
m and s values (which frequency is stronger?)
    │
    ▼ m > s → "1", else → "0"
Bit string: "010101110110..."
    │
    ▼ group into 8 bits, MSB-first → chr()
ASCII text: "tomctf{m0d3m_h1dd3n...}"
    │
    ▼
FLAG: tomctf{m0d3m_h1dd3n_1n_n01s3_b3ll_103}

```

Solution Script:-

```

import numpy as np
import scipy.io.wavfile as wav
 # --- SETTINGS FROM FOR SOLVING ---
fs = 1000000
baud = 300
mark_f = 1270
space_f = 1070
samples_per_bit = fs // baud
 def bits_to_text(bits):
"""Matches your format(ord(i), '08b') logic."""
chars = []
for i in range(0, len(bits) - 8, 8):
byte = bits[i:i+8]
chars.append(chr(int(byte, 2)))
return "".join(chars)
 # 1. Load the file
print("[*] Loading audio.wav...")
try:
rate, data = wav.read("audio.wav")
data = data.astype(np.float32) / 32767.0
except:
print("[-] Error: Make sure 'audio.wav' is in this folder.")
exit()
 # 2. The Multi-Frequency Hunt
# We will check 150kHz and -150kHz (just in case)
for freq in [150000, -150000]:
print(f"[*] Checking frequency: {freq} Hz...")
t = np.arange(len(data)) / fs
baseband = data * np.exp(-1j * 2 * np.pi * freq * t)
  # 3. Aggressive Alignment Check
# We slide the start by small increments to find the byte boundary
for offset in np.linspace(0, samples_per_bit, 20).astype(int):
bits = ""
for i in range(offset, len(baseband) - samples_per_bit, samples_per_bit):
chunk = baseband[i : i + samples_per_bit]
t_chunk = np.arange(len(chunk)) / fs

# Simple Correlation
m = np.abs(np.sum(chunk * np.exp(-1j * 2 * np.pi * mark_f * t_chunk)))
s = np.abs(np.sum(chunk * np.exp(-1j * 2 * np.pi * space_f * t_chunk)))
bits += "1" if m > s else "0"
# 4. Check for flag
decoded = bits_to_text(bits)
if "tomctf{" in decoded.lower():
print(f"\n[!!!] SUCCESS!")
print(f"Freq: {freq} | Offset: {offset}")
idx = decoded.lower().find("tomctf{")
print(f"FLAG: {decoded[idx:].split('}')[0] + '}'}")
exit()
 print("\n[-] Still nothing. Is it possible the baud rate or frequencies were changed?"

```


These are the steps for finding flag.....:)



