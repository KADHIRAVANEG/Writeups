# Break It - TryHackMe Writeup

## Overview
- **Room**: Break it
- **Difficulty**: Medium
- **Category**: Cryptography / Encodings / Ciphers
- **Objective**: Decode and crack complex multi-layered encodings, classical ciphers, and low-level bitwise operations.

---

## Task 1: Bases

This section explores multiple base encodings (Base32, Base64, Base58, Base85, Base91, Hexadecimal, and Decimal ASCII representations) chained together.

### 1. [Super easy]
**Ciphertext:**
```text
MVQXG6K7MJQXGZJTGI======
```

**Analysis & Decoding:**
The string consists of uppercase letters `A-Z` and numbers `2-7`, ending with `=` padding characters, which is the standard format for **Base32**.

```bash
echo "MVQXG6K7MJQXGZJTGI======" | base32 -d
```

**Answer:** `easy_base32`

---

### 2. [Easy]
**Ciphertext:**
```text
TVJYWEtZVE1NVlBXRVlMVE1WWlE9PT09
```

**Analysis & Decoding:**
The ciphertext is encoded with **Base64**. Decoding the Base64 layer reveals an intermediate Base32 string with padding `====`:

```bash
# Step 1: Base64 decode
echo "TVJYWEtZVE1NVlBXRVlMVE1WWlE9PT09" | base64 -d
# Output: MRXXKYTMMVPWEYLTMVZQ====

# Step 2: Base32 decode
echo "TVJYWEtZVE1NVlBXRVlMVE1WWlE9PT09" | base64 -d | base32 -d
```

**Answer:** `double_bases`

---

### 3. [Moderate]
**Ciphertext:**
```text
GM4HOU3VHBAW6OKNJJFW6SS2IZ3VAMTYORFDMUC2G44EQULIJI3WIVRUMNCWI6KGK5XEKZDTN5YU2RT2MR3E45KKI5TXSOJTKZJTC4KRKFDWKZTZOF3TORJTGZTXGNKCOE======
```

**Analysis & Decoding:**
This challenge features a pipeline of four distinct encoding schemes:
1. **From Base32**: Decodes to an alphanumeric Base58 string: `38wSu8Ao9MJKoJZFwP2xtJ6PZ78HQhJ7dV4cEdyFWnEdsoqMFzdvNuJGgy93VS1qQQGefyqw7E36gs5Bq`
2. **From Base58**: Decodes to space-separated hex bytes: `59 6d 46 7a 5a 54 45 32 58 32 6c 7a 58 32 68 6c 65 41 3d 3d`
3. **Hex to ASCII**: Converts to a Base64 string: `YmFzZTE2X2lzX2hleA==`
4. **From Base64**: Yields the final plaintext.

```python
import base64

s = "GM4HOU3VHBAW6OKNJJFW6SS2IZ3VAMTYORFDMUC2G44EQULIJI3WIVRUMNCWI6KGK5XEKZDTN5YU2RT2MR3E45KKI5TXSOJTKZJTC4KRKFDWKZTZOF3TORJTGZTXGNKCOE======"
# 1. Base32
b58_str = base64.b32decode(s).decode()

# 2. Base58
ALPHABET = "123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz"
num = 0
for char in b58_str:
    num = num * 58 + ALPHABET.index(char)
hex_bytes = bytes(reversed([num % 256 for num in iter(lambda: (num := num // 256) and num % 256 or 0, 0)]))

# 3. Hex & 4. Base64
b64_str = bytes.fromhex(hex_bytes.decode().replace(" ", "")).decode()
flag = base64.b64decode(b64_str).decode()
print("Flag:", flag)
```

**Answer:** `base16_is_hex`

---

### 4. [Hard]
**Ciphertext:**
```text
HRBUGQDUHFWDIXKUIBWXIJTHIE3DCY3BIE2FKQSZHNDE6MRUIA2TWWDMHRBV2ZKCHQWFCTLPIE2EEJDBIBZCEW3OHUSTOLRCHNFGMVC6IFJXIQ2AHVMVONSBHVOVIM2MHVPV42J4HQVCQ4REIFHVIJ2WHFWDYQSUHROGILJCIFIU23CXHNCEIXK2HRDVSXKOHV2SQJLC
```

**Analysis & Decoding:**
A deep chain of alternating base representations:
1. **Base32**: Decodes into ASCII85 (`<CC@t9l4]T@mt&g...`)
2. **ASCII85 (Base85)**: Decodes into Base64 (`UDdUMmlacEh5dW9KdHhx...`)
3. **Base64**: Decodes into Base58 (`P7T2iZpHyuoJtxqDBuki...`)
4. **Base58**: Decodes into ASCII85 (`>")t?@7O3c:KTqu9i...`)
5. **ASCII85**: Decodes into Base64 (`ZEdoaGRGOXBjMTloWDJ4dmRGOXZabDlpWVhObGN3PT0=`)
6. **Base64**: Decodes into Base64 (`dGhhdF9pc19hX2xvdF9vZl9iYXNlcw==`)
7. **Base64**: Decodes into the final plaintext.

**Answer:** `that_is_a_lot_of_bases`

---

### 5. [Insane]
**Analysis & Decoding:**
The input is a massive 17,108-character Base32 block. The multi-tiered decryption pipeline unwraps as follows:
1. **From Base32** $\rightarrow$ ASCII85 block (`2)6:/0JFV*1*AG1+>c&#1GU...`)
2. **From ASCII85** $\rightarrow$ Space-separated decimal ASCII values (`52 100 32 53 48 ...`)
3. **Decimal to Hex Characters** $\rightarrow$ Hex string (`4d 50 70 62 7a 23 ...`)
4. **Hex to ASCII** $\rightarrow$ Base91 string (`MPpbz#c0#8}8:*6cJx_H|maIPV<=...`)
5. **From Base91** $\rightarrow$ Base58 string (`aE8aZL9YYk4vQt1Zd7yaxV77...`)
6. **From Base58** $\rightarrow$ Hex values
7. **Hex to ASCII** $\rightarrow$ Base64 string
8. **From Base64** $\rightarrow$ Space-separated Decimal numbers
9. **Decimal to Hex / Plaintext** $\rightarrow$ Final string.

**Answer:** `defense_the_base`

---

## Task 2: Base and cipher

This section layers classical substitution and polyalphabetic ciphers (ROT13, ROT47, Vigenère) over standard base encodings.

### 1. [Easy] (Base32 + ROT13)
**Ciphertext:**
```text
PJXHQ4S7GEZV6ZTDOZQQ====
```

**Steps to Reproduce:**
1. Base32 decode:
   ```bash
   echo "PJXHQ4S7GEZV6ZTDOZQQ====" | base32 -d
   # Output: znxr_13_fcva
   ```
2. Apply ROT13:
   ```bash
   echo "znxr_13_fcva" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
   ```

**Answer:** `make_13_spin`

---

### 2. [Moderate] (Vigenère Cipher)
**Steps to Reproduce:**
1. Decode from **Base32**, then decode from **Base64**.
2. The output yields a ciphertext along with the Vigenère key.
3. Decrypting the ciphertext with the recovered key resolves the plaintext.

**Answer:** `I luv vigenere cipher`

---

### 3. [Hard] (Bases and ROTs)
**Steps to Reproduce:**
1. **From Base85**
2. **ROT47**
3. **From Base64**
4. **From Base32**
5. **ROT13**

**Answer:** `decode_and_rot`

---

### 4. [Insane] (Multi-Base + Vigenère Key Extraction)
**Steps to Reproduce:**
1. **From Base32** $\rightarrow$ **From Base85**
2. **From Base91**
3. **From Decimal** $\rightarrow$ **From Hex** $\rightarrow$ **From Base64**
4. Extract the derived Vigenère encryption key from the intermediate stream.
5. Decode from **Base58** and decrypt using the recovered Vigenère key.

**Answer:** `you are a real code cracker`

---

## Task 3: Base, cipher, bit shift

This section incorporates low-level bit manipulation (logical bit shifts and 8-bit arithmetic rotations) alongside ciphers and bases.

### 1. [Moderate] (Logical Bit Shift)
**Steps to Reproduce:**
1. **Block Shift Left** (Logical shift)
2. **From Base64**
3. **ROT13**

**Answer:** `shift logic like a boss`

---

### 2. [Hard] (Arithmetic Bit Shift)
**Steps to Reproduce:**
1. **Bit Rotate Left (8 Bits)** (Arithmetic shift)
2. **From Base58**
3. **ROT13**

**Answer:** `shift arithmetic like a boss`

---

### 3. [Insane] (Compound Rotations & Shifts)
**Steps to Reproduce:**
1. **Bit Rotate Left (8 Bits)** $\times 3$
2. **From Base64**
3. **Block Shift Left** $\times 2$
4. **From Base85**
5. **ROT13**

**Answer:** `God of shifs`

---

## Summary of Answers

| Task | Category | Level | Answer |
|---|---|---|---|
| **Task 1** | Bases | Super easy | `easy_base32` |
| **Task 1** | Bases | Easy | `double_bases` |
| **Task 1** | Bases | Moderate | `base16_is_hex` |
| **Task 1** | Bases | Hard | `that_is_a_lot_of_bases` |
| **Task 1** | Bases | Insane | `defense_the_base` |
| **Task 2** | Base and cipher | Easy | `make_13_spin` |
| **Task 2** | Base and cipher | Moderate | `I luv vigenere cipher` |
| **Task 2** | Base and cipher | Hard | `decode_and_rot` |
| **Task 2** | Base and cipher | Insane | `you are a real code cracker` |
| **Task 3** | Bit shift | Moderate | `shift logic like a boss` |
| **Task 3** | Bit shift | Hard | `shift arithmetic like a boss` |
| **Task 3** | Bit shift | Insane | `God of shifs` |
