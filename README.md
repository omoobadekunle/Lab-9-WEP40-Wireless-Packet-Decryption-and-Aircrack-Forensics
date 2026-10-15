# SBT-DF203 Lab 9: WEP40 Wireless Packet Decryption and Aircrack Forensics

![Course](https://img.shields.io/badge/Course-SBT--DF203-blue)
![Platform](https://img.shields.io/badge/Platform-Kali%20Linux%20VM-557C94)
![Encryption](https://img.shields.io/badge/Encryption-WEP40-red)
![Mode](https://img.shields.io/badge/Analysis-Offline%20Only-green)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)

A digital forensics investigation of a historical WEP40-protected 802.11 packet capture (CodeGate CTF 2015, "Good Crypto" challenge). The work covers evidence preservation, key validation with Aircrack-ng, offline decryption with airdecap-ng, endpoint and conversation extraction, and artefact carving.

---

## Table of Contents

1. [Course Information](#course-information)
2. [Executive Summary](#1-executive-summary)
3. [Authority, Scope and Evidence](#2-authority-scope-and-evidence-description)
4. [Environment and Setup](#3-environment-and-setup)
5. [Evidence Integrity and Chain of Custody](#4-evidence-integrity-and-chain-of-custody)
6. [Part A: Inventory the Wireless Capture](#5-part-a--inventory-the-wireless-capture)
7. [Part B: Identify WEP Protection and Key Parameters](#6-part-b--identify-wep-protection-and-key-parameters)
8. [Part C: Recover or Validate the WEP40 Key](#7-part-c--recover-or-validate-the-wep40-key)
9. [Part D: Decrypt the Capture Offline](#8-part-d--decrypt-the-capture-offline)
10. [Part E: Extract Endpoints and Conversations](#9-part-e--extract-endpoints-and-conversations)
11. [Part F: Extract Images, HTML and Other Objects](#10-part-f--extract-images-html-and-other-objects)
12. [Part G: Optional Historical Passphrase Challenge](#11-part-g--optional-historical-passphrase-challenge)
13. [WEP Security Analysis](#12-wep-security-analysis)
14. [Required Findings Worksheet](#13-required-findings-worksheet)
15. [Required Screenshots Checklist](#14-required-screenshots-checklist)
16. [Academic Integrity and Authorisation Statement](#15-academic-integrity-and-authorisation-statement)

---

## Course Information

| Field | Details |
|---|---|
| Course Code | SBT-DF203 |
| Course Title | Basic Networking Skills for Digital Forensics |
| Lab Number | 9 |
| Lab Title | WEP40 Wireless Packet Decryption and Aircrack Forensics |
| Platform | Kali Linux Virtual Machine |
| Student | Adekunle Ogunyemi |
| Student ID | 2025/FWSD/11479 |
| Instructor | Aminu Idris, AMCPN |
| Date | 25 September 2026 |

---

## 1. Executive Summary

This lab focused on the forensic analysis of a historical wireless packet capture protected with WEP40 encryption. The evidence was a compressed CodeGate CTF 2015 capture file, which was downloaded, preserved, decompressed and analysed entirely offline inside an isolated Kali Linux VM.

The goals were to identify IEEE 802.11 traffic, validate the WEP40 key with Aircrack-ng, decrypt the capture with airdecap-ng, and extract forensic artefacts such as IP/MAC endpoints, TCP conversations, images and HTML files.

**Key results**

| Metric | Result |
|---|---|
| Total packets | 45,169 |
| Data frames | 15,713 |
| WEP-protected data packets | 15,477 |
| Recovered 40-bit key | `A4:3D:F6:F3:74` (15,477 IVs) |
| airdecap-ng key format | `A43DF6F374` (no colons) |
| Decryption success | 100% of WEP data packets (0 corrupted) |
| Main internal host | 192.168.0.15 (Apple_68:96:7c) |
| Main external server | 198.90.20.111, port 80 |
| Carved artefacts | Images, HTML, CSS, JavaScript, font files |

After decryption, higher-layer protocols (ARP, DHCP, DNS, TCP, TLSv1, IGMPv2) became visible. The lab confirmed that WEP is cryptographically broken and should be replaced with WPA2-AES/CCMP or WPA3.

<img width="1517" height="228" alt="Screenshot 2026-09-26 114643" src="https://github.com/user-attachments/assets/84ac8b50-9f09-4e78-80e9-430328cf5737" />  

<img width="1538" height="445" alt="Screenshot 2026-09-26 114731" src="https://github.com/user-attachments/assets/841123b2-3852-4b88-bb1f-99499ebc9dc1" />


---

## 2. Authority, Scope and Evidence Description

### 2.1 Authority

This lab was completed as part of the SBT-DF203 course at the International Cybersecurity and Digital Forensics Academy (ICDFA). All work was performed inside an authorised Kali Linux virtual machine. The capture was a supplied historical training file from the CodeGate CTF 2015 Good Crypto challenge. No live wireless networks were targeted, and no third-party systems were accessed.

### 2.2 Scope

- Preserving and hashing the supplied `file.xz` evidence
- Decompressing the capture and verifying its file type
- Identifying 802.11 management, control and data frames
- Confirming WEP protection and extracting IV-related fields
- Recovering or validating the WEP40 key with Aircrack-ng
- Decrypting the capture offline with airdecap-ng
- Extracting IP/MAC endpoints and TCP conversations
- Carving images, HTML and other objects from the decrypted traffic
- Documenting the security weaknesses of WEP




### 2.3 Evidence Description

| Evidence Item | Description | Size |
|---|---|---|
| `evidence/file.xz` | Original compressed CodeGate CTF capture | 12,837,660 bytes |
| `working/file_working` | Decompressed pcap capture | 14,234,002 bytes |
| `working/file_working-dec` | Decrypted capture produced by airdecap-ng | 12,784,501 bytes |

<img width="1670" height="661" alt="Screenshot 2026-09-26 115002" src="https://github.com/user-attachments/assets/84ab86f9-5423-4704-b4ab-2c9b24d33c70" />

---

## 3. Environment and Setup

### 3.1 Working Environment

- **Operating System:** Kali Linux VM
- **Working Directory:** `/home/kali/Desktop/Adekunle_Ogunyemi/SBT-DF203-Lab9`
- **Analysis Date:** 26 September 2026
- **Tools Used:** Aircrack-ng, airdecap-ng, Wireshark, tshark, foremost, binwalk, file, strings, xz-utils, wget
- **Network Mode:** Offline analysis only

### 3.2 Folder Structure

```bash
mkdir -p ~/SBT-DF203-Lab9/{evidence,working,exported,reports,screenshots,scripts}
cd ~/SBT-DF203-Lab9
pwd
find . -maxdepth 1 -type d -print
```

<img width="1538" height="445" alt="Screenshot 2026-09-26 114731" src="https://github.com/user-attachments/assets/c4d1bd01-3885-436f-b6a1-f24d7d4cc00b" />


### 3.3 Evidence Acquisition

```bash
sudo wget -O evidence/file.xz \
'https://raw.githubusercontent.com/ctfs/write-ups-2015/master/codegate-ctf-2015/programming/good-crypto/file.xz'
```

The download completed successfully (12.24 MB in 4.4 seconds).

<img width="1670" height="661" alt="Screenshot 2026-09-26 115002" src="https://github.com/user-attachments/assets/e35bc9b8-dccb-404d-890d-8f3386146115" />


---

## 4. Evidence Integrity and Chain of Custody

### 4.1 Chain of Custody Worksheet

| Field | Entry |
|---|---|
| Case/Lab Identifier | SBT-DF203-Lab9-AdekunleOgunyemi |
| Trainee Name | Adekunle Ogunyemi |
| ID Number | 2025/FWSD/11479 |
| Date/Time Started | 26 September 2026 06:49 UTC |
| Evidence File Name | `file.xz` |
| Source/Method | Downloaded from CodeGate CTF 2015 GitHub repository |
| Original SHA-256 | `dd54144caef34f228bfb4b87a9101ca7173969376f7ed357bd068061d4f4b8d6` |
| Working-copy SHA-256 | Same as original (verified) |
| Analysis VM | Kali Linux |
| Notes on Changes | None. Original preserved; working copy used for analysis |

### 4.2 Hash Verification

```bash
sha256sum evidence/file.xz | tee reports/file_xz_sha256.txt
```

```
dd54144caef34f228bfb4b87a9101ca7173969376f7ed357bd068061d4f4b8d6  evidence/file.xz
```

<img width="1666" height="243" alt="Screenshot 2026-09-26 115017" src="https://github.com/user-attachments/assets/d5d25d45-1989-48ff-9fb2-eebed6f24064" />


The file was then copied to the working directory and decompressed:

```bash
cp --preserve=timestamps evidence/file.xz working/file_working.xz
unxz -k working/file_working.xz
file working/file_working
```

```
working/file_working: pcap capture file, microsecond ts (little-endian) - version 2.4 (802.11, capture length 65535)
```

<img width="1667" height="218" alt="Screenshot 2026-09-26 115245" src="https://github.com/user-attachments/assets/210ff3f1-98d4-42f2-8718-5f007b0124a2" />


The working files were hashed:

```bash
sha256sum working/file_working.xz working/file_working | tee reports/working_hashes.txt
```

```
dd54144caef34f228bfb4b87a9101ca7173969376f7ed357bd068061d4f4b8d6  working/file_working.xz
c17a3f9b955e84f5befd476dbd55c67286d1e3eea9ab402d5359cac0874ebb2d  working/file_working
```

<img width="1662" height="192" alt="Screenshot 2026-09-26 115532" src="https://github.com/user-attachments/assets/0af8e539-2eb9-458f-9309-9297944af5ed" />


The original and working copy hashes matched, confirming that the evidence was preserved without alteration.

---

## 5. Part A – Inventory the Wireless Capture

### 5.1 Capture Metadata

```bash
capinfos "$CAP" | tee reports/capinfos.txt
```

- **File type:** Wireshark/tcpdump pcap
- **Encapsulation:** IEEE 802.11 Wireless LAN
- **Number of packets:** 45,169
- **File size:** 14 MB
- **Capture duration:** 272.969795 seconds
- **Earliest packet:** 2015-03-06 02:51:54.364525
- **Latest packet:** 2015-03-06 02:56:27.334320
- **Data bit rate:** 395 kbps
- **Average packet size:** 299.13 bytes
- **SHA-256:** `c17a3f9b955e84f5befd476dbd55c67286d1e3eea9ab402d5359cac0874ebb2d`

<img width="1275" height="742" alt="Screenshot 2026-09-26 115701" src="https://github.com/user-attachments/assets/1b949a66-a702-40d1-bdc5-b194bba0eaf7" />


### 5.2 Protocol Hierarchy

```bash
tshark -r "$CAP" -q -z io,phs | tee reports/protocol_hierarchy.txt
```

| Layer | Frames | Bytes |
|---|---|---|
| frame | 45,169 | 13,511,274 |
| wlan | 45,169 | 13,511,274 |
| data | 15,713 | 13,104,128 |

This confirmed that the capture is entirely 802.11 wireless traffic, with a large portion being data frames.

<img width="1063" height="347" alt="Screenshot 2026-09-26 115748" src="https://github.com/user-attachments/assets/111f31d2-39de-43d1-b0fe-845af29aa2c4" />


### 5.3 802.11 Frame Categories

```bash
tshark -r "$CAP" -Y 'wlan' -T fields \
  -e frame.number -e frame.time -e wlan.fc.type -e wlan.fc.subtype \
  -e wlan.sa -e wlan.da -e wlan.bssid \
  | head -n 100 | tee reports/wlan_frame_sample.tsv
```

The sample showed type 1 frames (control frames) with subtype 11, meaning the first 100 frames were control frames used for flow control and acknowledgements.

<img width="1786" height="933" alt="Screenshot 2026-09-26 115827" src="https://github.com/user-attachments/assets/e4f52676-2be9-49d1-980b-df3b8691735f" />


| Frame Category | Type Value | Purpose | Example Evidence |
|---|---|---|---|
| Management | 0 | Association, authentication, beacons | Not dominant in sample |
| Control | 1 | Flow control and acknowledgements | Type 1, subtype 11 |
| Data | 2 | Carries network payload | 15,713 data frames |

---

## 6. Part B – Identify WEP Protection and Key Parameters

### 6.1 Protected Frames

```bash
tshark -r "$CAP" -Y 'wlan.fc.protected==1' -T fields \
  -e frame.number -e frame.time_epoch -e wlan.sa -e wlan.da \
  -e wlan.bssid -e wlan.wep.iv -e wlan.wep.key \
  | head -n 200 | tee reports/wep_protected_frames.tsv
```

Key observations:

- **BSSID:** `00:26:66:55:97:d6`
- **Source MAC examples:** `00:26:66:55:97:d4`, `04:8d:38:48:8e:b4`, `08:10:77:92:7c:f2`
- **Destination MAC examples:** `ff:ff:ff:ff:ff:ff`, `00:26:66:55:97:d6`
- **IV values (examples):** `0x6f7fcc`, `0x2cbcbb`, `0x3cbb`, `0x4cbb`, `0x5cbb`, `0x6c6bb`, `0x8cbb`, `0x6effc`, `0x69ffc`, `0x9cbb`

<img width="1832" height="1031" alt="Screenshot 2026-09-26 115941" src="https://github.com/user-attachments/assets/02365854-afc4-4699-94bc-d325c1fe7034" />


### 6.2 Repeated IV Summary

```bash
tshark -r "$CAP" -Y 'wlan.fc.protected==1' -T fields -e wlan.wep.iv \
  | sort | uniq -c | sort -nr | head -n 30 | tee reports/repeated_iv_summary.txt
```

Many IVs appeared repeatedly, which is a known weakness of WEP. Repeated IVs mean the same RC4 keystream is reused, which leaks relationships between plaintexts and enables statistical key recovery.

<img width="1391" height="877" alt="Screenshot 2026-09-26 120024" src="https://github.com/user-attachments/assets/325bc890-b0c0-48d2-9eb3-a740cfbc38ec" />

### 6.3 WEP40 Composition

WEP40 (64-bit WEP) combines a 24-bit Initialisation Vector (IV) with a 40-bit secret key. The IV is transmitted in plaintext with each frame. Because the IV space is only 2^24, busy networks reuse IVs quickly. RC4 also has known keystream biases. Together, these weaknesses make WEP unsuitable for protecting wireless traffic.

---

## 7. Part C – Recover or Validate the WEP40 Key

### 7.1 Aircrack-ng Execution

```bash
aircrack-ng "$CAP" | tee reports/aircrack_output.txt
```

```
1  00:26:66:55:97:D6  cgnetwork  WEP (15477 IVs)
Choosing first network as target.
Opening working/file_working
Read 45169 packets.
1 potential targets
Attack will be restarted every 5000 captured ivs.
Aircrack-ng 1.7
[00:00:00] Tested 83 keys (got 15477 IVs)
KEY FOUND! [ A4:3D:F6:F3:74 ]
Decrypted correctly: 100%
```

<img width="1791" height="955" alt="Screenshot 2026-09-26 120228" src="https://github.com/user-attachments/assets/55e30a55-5b02-4990-889e-6c47342ec914" />



### 7.2 Key Validation

The recovered key was `A4:3D:F6:F3:74`. This matches the historical training key provided in the course slide deck.

```bash
printf 'A4:3D:F6:F3:74\n' | tee reports/validated_wep40_key_masked.txt
```

> **Note:** This key belongs to the historical CTF dataset and is not authorisation to attack any wireless network.

<img width="1891" height="615" alt="Screenshot 2026-09-26 120202" src="https://github.com/user-attachments/assets/0373db84-2e2e-4b92-b700-4aade3dd14e9" />


---

## 8. Part D – Decrypt the Capture Offline

### 8.1 Airdecap-ng Decryption

```bash
cd working
airdecap-ng -w A43DF6F374 file_working | tee ../reports/airdecap_output.txt
cd ..
```

| Statistic | Count |
|---|---|
| Total number of stations seen | 10 |
| Total number of packets read | 45,169 |
| Total number of WEP data packets | 15,477 |
| Total number of WPA data packets | 0 |
| Number of plaintext data packets | 0 |
| Number of decrypted WEP packets | 15,477 |
| Number of corrupted WEP packets | 0 |
| Number of decrypted WPA packets | 0 |
| Number of bad TKIP (WPA) packets | 0 |
| Number of bad CCMP (WPA) packets | 0 |

All 15,477 WEP data packets were decrypted successfully with zero corrupted packets.

<img width="1726" height="436" alt="Screenshot 2026-09-26 121109" src="https://github.com/user-attachments/assets/d78e78fc-801b-40cc-bc0f-18d604ee6a58" />


### 8.2 Decrypted File Inventory

```bash
find working -maxdepth 1 -type f -ls | tee reports/decrypted_files_inventory.txt
```

- `working/file_working.xz`: 12,837,660 bytes
- `working/file_working-dec`: 12,784,501 bytes (decrypted capture)
- `working/file_working`: 14,234,002 bytes
- `working/file_working_copy`: 14,234,002 bytes

<img width="1717" height="285" alt="Screenshot 2026-09-26 121140" src="https://github.com/user-attachments/assets/e68bc4d0-6099-4df1-9d4f-086ddf5bbe0f" />


### 8.3 Hashes of Working Files

```bash
sha256sum working/* | tee reports/all_working_file_hashes.txt
```

```
c17a3f9b955e84f5befd476dbd55c67286d1e3eea9ab402d5359cac0874ebb2d  working/file_working
c17a3f9b955e84f5befd476dbd55c67286d1e3eea9ab402d5359cac0874ebb2d  working/file_working_copy
167c91994c269777f9048227deb89882caf3cf3c763977f2059604f9a6a40b04  working/file_working-dec
dd54144caef34f228bfb4b87a9101ca7173969376f7ed357bd068061d4f4b8d6  working/file_working.xz
```

<img width="1721" height="265" alt="Screenshot 2026-09-26 121229" src="https://github.com/user-attachments/assets/82764ffc-7ce5-420c-b5e5-7316aa1896a3" />


---

## 9. Part E – Extract Endpoints and Conversations

### 9.1 Decrypted Capture Path

```bash
DEC=$(find working -maxdepth 1 -type f -name '*-dec*' | head -n 1)
echo "Decrypted capture: $DEC" | tee reports/decrypted_capture_path.txt
```

### 9.2 Ethernet Endpoints

```bash
tshark -r "$DEC" -q -z endpoints,eth | tee reports/ethernet_endpoints.txt
```

| Endpoint | Packets | Bytes |
|---|---|---|
| Apple_68:96:7c | 15,407 | 12 MB |
| EFMNetworks_55:97:d4 | 15,020 | 12 MB |
| ASUSTekCOMPU_2a:c2:7a | 358 | 281 kB |

<img width="1740" height="758" alt="Screenshot 2026-09-26 122523" src="https://github.com/user-attachments/assets/a8a67133-aee3-47e6-8a3e-316ba44f795b" />


### 9.3 IP Endpoints

```bash
tshark -r "$DEC" -q -z endpoints,ip | tee reports/ip_endpoints.txt
```

| IP Address | Packets | Bytes | Tx Packets | Tx Bytes | Rx Packets | Rx Bytes |
|---|---|---|---|---|---|---|
| 192.168.0.15 | 15,309 | 12 MB | 6,995 | 608 kB | 8,314 | 11 MB |
| 198.90.20.111 | 8,057 | 6,916 kB | 4,446 | 6,637 kB | 3,611 | 279 kB |
| 199.27.79.193 | 5,694 | 4,825 kB | 3,131 | 4,603 kB | 2,563 | 221 kB |
| 192.168.0.9 | 329 | 277 kB | 197 | 265 kB | 132 | 11 kB |
| 173.194.127.141 | 153 | 109 kB | 80 | 102 kB | 73 | 7,364 bytes |
| 8.8.8.8 | 81 | 10 kB | 40 | 6,960 bytes | 41 | 3,391 bytes |

<img width="1667" height="1042" alt="Screenshot 2026-09-26 122613" src="https://github.com/user-attachments/assets/415687c2-decb-4288-a06e-a0a5116169cb" />


### 9.4 TCP Conversations

```bash
tshark -r "$DEC" -q -z conv,tcp | tee reports/tcp_conversations.txt
```

| Conversation | Frames | Bytes | Duration |
|---|---|---|---|
| 192.168.0.15:51007 <-> 198.90.20.111:80 | 2,235 | 1,958 kB | 45.4035 s |
| 192.168.0.15:50959 <-> 199.27.79.193:80 | 2,168 | 1,887 kB | 34.8633 s |
| 192.168.0.15:51008 <-> 198.90.20.111:80 | 2,106 | 1,810 kB | 46.0179 s |
| 192.168.0.15:51001 <-> 199.27.79.193:80 | 2,072 | 1,791 kB | 26.6088 s |
| 192.168.0.15:51009 <-> 198.90.20.111:80 | 1,259 | 1,072 kB | 28.7983 s |

<img width="1805" height="1052" alt="Screenshot 2026-09-26 122712" src="https://github.com/user-attachments/assets/596fa2d0-b53e-468b-a625-2cd4a67192ab" />

### 9.5 IP/MAC Mapping Sample

```bash
tshark -r "$DEC" -Y 'ip' -T fields \
  -e frame.number -e wlan.sa -e wlan.da -e ip.src -e ip.dst -e _ws.col.Protocol \
  | head -n 200 | tee reports/ip_mac_mapping_sample.tsv
```

The output showed:

- DHCP traffic between `0.0.0.0` and `255.255.255.255`
- DNS queries from `192.168.0.15` to `8.8.8.8`
- IGMPv2 traffic to `224.0.0.2` and `224.0.0.251`
- TCP and TLSv1 traffic between `192.168.0.15` and `17.172.239.31`

<img width="1812" height="993" alt="Screenshot 2026-09-26 122752" src="https://github.com/user-attachments/assets/fd6d893a-aee1-42b6-9b36-97c163d9878a" />


### 9.6 ARP and DHCP Evidence

The decrypted capture also revealed ARP requests and DHCP transactions:

```
0.000000 EFMNetworks_55:97:d4 > Broadcast ARP 42 Who has 192.168.0.15? Tell 192.168.0.1
0.944186 Apple_68:96:7c > NetisTechnol_48:e5:b4 ARP 42 Who has 192.168.0.1? Tell 192.168.0.15
0.944185 0.0.0.0 > 255.255.255.255 DHCP 342 DHCP Request - Transaction ID 0x21e6be1c
0.954435 192.168.0.1 > 255.255.255.255 DHCP 590 DHCP NAK - Transaction ID 0x21e6be1c
1.030202 192.168.0.15 > 8.8.8.8 DNS 73 Standard query 0x9583 A www.apple.com
```

<img width="1880" height="943" alt="Screenshot 2026-09-26 123950" src="https://github.com/user-attachments/assets/978cb258-fb71-43ab-9861-6791d547d88a" />


---

## 10. Part F – Extract Images, HTML and Other Objects

### 10.1 HTTP Object Export

```bash
mkdir -p exported/http_objects exported/foremost
tshark -r "$DEC" --export-objects http,exported/http_objects 2>&1 | tee reports/http_export_log.txt
```

### 10.2 Exported File Types

```bash
find exported -type f -exec file {} \; | tee reports/exported_file_types.txt
```

Recovered files included:

- **GIF images:** `00001260.gif`, `00010876.gif`, `00003719.gif`, `00003653.gif`, `00000406.gif`, `00002450.gif`, `00010456.gif`
- **HTML documents:** `00001639.htm`, `00024427.htm`
- **PNG images:** `00020736.png` (19x19), `00001831.png` (12x12), `00001765.png` (44x44), `00010214.png` (1x1), `00024242.png` (19x19), `00010671.png` (1x400)
- **JPEG images:** `00004538.jpg` (718x404), `00000104.jpg` (2492x2492)
- **Text file:** `audit.txt`
- **HTTP objects:** `of_destiny.png`, `bootstrap.min.css`, `accordion.min.js`, `trackalyze.js`, `dynamik-min.css`, `lumbar.gif`

<img width="1917" height="975" alt="Screenshot 2026-09-26 124152" src="https://github.com/user-attachments/assets/1bea55f4-3194-44ef-a5bc-e49a13739955" />


### 10.3 Exported File Hashes

```bash
find exported -type f -exec sha256sum {} \; | tee reports/exported_file_hashes.txt
```

Example hashes:

| SHA-256 | File |
|---|---|
| `8337212354871836e6763a41e615916c89bac5b3f1f0adf60ba43c7c806e1015` | `exported/foremost/gif/00001260.gif` |
| `a0d3a0aff7dc3bf32d2176fc3dcda6e7aba2867c4f4d1f7af6355d2cfc6c44f8` | `exported/foremost/gif/00000406.gif` |
| `3f9bed324e04c1735b498dab0f138673e20591a5b39201e731a91dd180929ba6` | `exported/foremost/jpg/00004538.jpg` |
| `e97ffa450c3e5b483fd008453b6170f97dbec5c6cf096c8223a6b332c861376b` | `exported/http_objects/YDAoLskQQ5MOAgvHUQCcLQa6gm6bS00u2Qn-iPLo1Go.woff` |

<img width="1912" height="1076" alt="Screenshot 2026-09-26 124233" src="https://github.com/user-attachments/assets/2a2d44eb-c045-4c17-a753-50873a6b2b1c" />


---

## 11. Part G – Optional Historical Passphrase Challenge

This part was optional and instructor-assessed. The course slide discussed deriving a seed and cracking the CTF passphrase using supplied challenge information. The 40-bit WEP key `A4:3D:F6:F3:74` is different from a human-readable router passphrase. The known key, derived seed, character constraints and challenge hash were recorded. Only the instructor-provided `fndseed.py` or `crackWEP40` script would be used against the historical dataset. No live networks were targeted.

---

## 12. WEP Security Analysis

WEP is cryptographically unsafe for several reasons:

1. **Short 24-bit IV:** The IV space is only 2^24, so IVs repeat quickly on busy networks.
2. **RC4 keystream reuse:** When the same IV is used, the same RC4 keystream is reused, leaking relationships between plaintexts.
3. **Statistical key recovery:** Repeated IVs and RC4 biases allow statistical attacks that recover the key.
4. **Weak integrity:** WEP uses CRC-32 for integrity, which is not cryptographically secure and can be forged.
5. **No replay protection:** WEP does not prevent replay attacks.
6. **No protected management frames:** Management frames are not protected.

**Recommendation:** Replace WEP with WPA2-AES/CCMP or WPA3 with strong authentication and protected management frames where supported. Legacy WEP should be removed rather than strengthened with longer passphrases.

---

## 13. Required Findings Worksheet

| Question | Finding |
|---|---|
| Capture format and duration | pcap, IEEE 802.11, 272.97 seconds |
| BSSID | `00:26:66:55:97:d6` |
| Primary station MAC addresses | `00:26:66:55:97:d4`, `04:8d:38:48:8e:b4`, `08:10:77:92:7c:f2` |
| Protected frame count | 15,477 WEP data packets |
| Repeated IV evidence | Many IVs repeated; repeated IVs confirmed WEP weakness |
| Recovered/validated WEP40 key | `A4:3D:F6:F3:74` |
| Decrypted capture filename/hash | `working/file_working-dec`: `167c91994c269777f9048227deb89882caf3cf3c763977f2059604f9a6a40b04` |
| Host IP and MAC | 192.168.0.15 (Apple_68:96:7c) |
| Server IP and MAC | 198.90.20.111 (external HTTP server) |
| Key protocols observed | ARP, DHCP, DNS, TCP, TLSv1, IGMPv2 |
| Recovered images/HTML | GIF, PNG, JPEG, HTML, CSS, JS, WOFF |
| Optional router passphrase | Protected appendix only |
| Security conclusion | WEP is broken; migrate to WPA2-AES/CCMP or WPA3 |

---

## 14. Required Screenshots Checklist

| # | Screenshot Description | Status |
|---|---|---|
| 1 | Compressed evidence file and hash | Done |
| 2 | Decompressed capture identification | Done |
| 3 | Protocol hierarchy | Done |
| 4 | 802.11 frame categories | Done |
| 5 | Protected WEP frames and IV evidence | Done |
| 6 | Aircrack-ng result | Done |
| 7 | Airdecap-ng decryption result | Done |
| 8 | Decrypted IP/MAC endpoint inventory | Done |
| 9 | Exported image or HTML objects | Done |
| 10 | Hashes of decrypted capture and recovered files | Done |

---

## 15. Academic Integrity and Authorisation Statement

I confirm that I completed this lab in an authorised environment, preserved the supplied evidence, did not target any third-party system, and accurately documented my own commands, observations and conclusions.

**Signed:** Adekunle Ogunyemi
**Date:** 25 September 2026

---

## Repository Structure

```
SBT-DF203-Lab9/
├── evidence/       # Original file.xz (preserved, untouched)
├── working/        # Decompressed and decrypted captures
├── exported/       # Carved files (foremost) and HTTP objects
├── reports/        # Command outputs, hashes, and logs
├── screenshots/    # Lab evidence screenshots
└── scripts/        # Helper scripts
```

## Disclaimer

This repository is for **educational and forensic training purposes only**. The capture and key come from a historical CTF dataset and were analysed offline in an isolated VM. Do not use these techniques against networks you do not own or have explicit written permission to test.
