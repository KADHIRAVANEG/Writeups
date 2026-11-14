# Cybersecurity & CTF Writeups

A curated collection of detailed challenge writeups, exploitation methodologies, reverse engineering walkthroughs, and practical security assessments organized into two dedicated categories: **Tomctf** and **TRYHACKME_Labs**.

---

## 📂 Repository Structure

```text
├── README.md
├── Tomctf/
│   ├── B2R.md
│   ├── BlackHole.md
│   ├── FerricOxide.md
│   ├── Intercepted Transmission.md
│   ├── QuiteRare.md
│   ├── The Ghost in the Static.md
│   └── Won The Fate.md
└── TRYHACKME_Labs/
    ├── Binex.md
    ├── BreakIt.md
    ├── REloaded.md
    └── screenshots/
        ├── Binex/
        └── REloaded/
```

---

## 🎯 TRYHACKME_Labs

Practical, hands-on walkthroughs for challenge machines and boot-to-root labs on [TryHackMe](https://tryhackme.com).

| Machine / Lab | Category | Difficulty | Key Concepts & Techniques | Writeup |
|---|---|---|---|---|
| **Binex** | Binary Exploitation & Linux Privesc | Medium | SMB user enumeration (`rpcclient`), SSH brute force, SUID GTFOBins, 64-bit stack buffer overflow (GDB), PATH hijacking | [Binex.md](TRYHACKME_Labs/Binex.md) |
| **Break It** | Web Security & System Pentesting | Medium | Web application enumeration, credential discovery, privilege escalation, post-exploitation | [BreakIt.md](TRYHACKME_Labs/BreakIt.md) |
| **REloaded** | Reverse Engineering | Hard | Windows PE x86 binary analysis, Ghidra XREF tracing, assembly patching (`JNZ` → `JZ`), stack string deobfuscation, CRT startup (`entry` → `main`) tracing, prime-indexed XOR reversing | [REloaded.md](TRYHACKME_Labs/REloaded.md) |

---

## 🚩 Tomctf

Comprehensive solutions and methodologies for challenges from **Tom CTF**, categorized by discipline:

| Challenge | Category | Description / Core Technique | Writeup |
|---|---|---|---|
| **Boot 2 Root (B2R)** | Boot-to-Root / Web | Network reconnaissance, HTTP service enumeration on port 8080, SSH pivoting, privilege escalation | [B2R.md](Tomctf/B2R.md) |
| **BlackHole** | Forensics / Steganography | Image metadata analysis, hidden layer extraction, steganographic data recovery | [BlackHole.md](Tomctf/BlackHole.md) |
| **Ferric-Oxide** | Reverse Engineering | Rust compiled binary reverse engineering, control flow analysis, key validation | [FerricOxide.md](Tomctf/FerricOxide.md) |
| **Intercepted Transmission** | Audio Forensics / Signals | Radio transmission analysis, audio spectrogram rendering, hidden frequency decoding | [Intercepted Transmission.md](Tomctf/Intercepted%20Transmission.md) |
| **Quite-Rare** | Steganography / QR | Corrupted/partial QR code reconstruction, error correction exploitation, payload decoding | [QuiteRare.md](Tomctf/QuiteRare.md) |
| **The Ghost in the Static** | Signal Forensics | Frequency Shift Keying (FSK) demodulation, radio waveform analysis, signal extraction | [The Ghost in the Static.md](Tomctf/The%20Ghost%20in%20the%20Static.md) |
| **Won The Fate** | Reverse Engineering / Logic | Binary control flow reconstruction, constraint satisfaction, logic reversing | [Won The Fate.md](Tomctf/Won%20The%20Fate.md) |

---

## 🛠️ Tools & Environments

- **Reverse Engineering**: NSA Ghidra, GDB, GNU Binutils (`objdump`, `strings`, `readelf`), IDA Pro, Rizin / Cutter
- **Network & Recon**: Nmap, `rpcclient`, Wireshark / TShark, Netcat
- **Web & Exploitation**: Burp Suite, Python3 custom exploit scripting, GTFOBins
- **Signals & Forensics**: Sonic Visualiser, Audacity, CyberChef, Steghide, Zsteg, ExifTool
