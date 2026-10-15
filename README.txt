# SBT-DF203 Lab 9: WEP40 Wireless Packet Decryption and Aircrack Forensics

![Course](https://img.shields.io/badge/Course-SBT--DF203-blue)
![Platform](https://img.shields.io/badge/Platform-Kali%20Linux%20VM-557C94)
![Encryption](https://img.shields.io/badge/Encryption-WEP40-red)
![Mode](https://img.shields.io/badge/Analysis-Offline%20Only-green)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen)


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

> **Screenshot:** Working directory (`pwd`)
>
> ![Working directory](screenshots/01_pwd_working_directory.png)

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

> **Screenshot:** Folder structure listing
>
> ![Folder structure](screenshots/02_folder_structure.png)

### 2.3 Evidence Description

| Evidence Item | Description | Size |
|---|---|---|
| `evidence/file.xz` | Original compressed CodeGate CTF capture | 12,837,660 bytes |
| `working/file_working` | Decompressed pcap capture | 14,234,002 bytes |
| `working/file_working-dec` | Decrypted capture produced by airdecap-ng | 12,784,501 bytes |

> **Screenshot:** Evidence download with `wget`
>
> ![Evidence download](screenshots/03_evidence_download.png)

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

> **Screenshot:** Folder structure verification
>
> ![Folder structure verification](screenshots/04_folder_structure_verification.png)

### 3.3 Evidence Acquisition

```bash
sudo wget -O evidence/file.xz \
'https://raw.githubusercontent.com/ctfs/write-ups-2015/master/codegate-ctf-2015/programming/good-crypto/file.xz'
```

The download completed successfully (12.24 MB in 4.4 seconds).

> **Screenshot:** Download output
>
> ![Download output](screenshots/05_download_output.png)

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

> **Screenshot:** SHA-256 of the original evidence
>
> ![Evidence hash](screenshots/06_evidence_hash.png)

The file was then copied to the working directory and decompressed:

```bash
cp --preserve=timestamps evidence/file.xz working/file_working.xz
unxz -k working/file_working.xz
file working/file_working
```

```
working/file_working: pcap capture file, microsecond ts (little-endian) - version 2.4 (802.11, capture length 65535)
```

> **Screenshot:** Decompressed capture identification
>
> ![File identification](screenshots/07_file_identification.png)

The working files were hashed:

```bash
sha256sum working/file_working.xz working/file_working | tee reports/working_hashes.txt
```

```
dd54144caef34f228bfb4b87a9101ca7173969376f7ed357bd068061d4f4b8d6  working/file_working.xz
c17a3f9b955e84f5befd476dbd55c67286d1e3eea9ab402d5359cac0874ebb2d  working/file_working
```

> **Screenshot:** Working file hashes
>
> ![Working file hashes](screenshots/08_working_hashes.png)

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

> **Screenshot:** capinfos output
>
> ![capinfos output](screenshots/09_capinfos.png)

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

> **Screenshot:** Protocol hierarchy
>
> ![Protocol hierarchy](screenshots/10_protocol_hierarchy.png)

### 5.3 802.11 Frame Categories

```bash
tshark -r "$CAP" -Y 'wlan' -T fields \
  -e frame.number -e frame.time -e wlan.fc.type -e wlan.fc.subtype \
  -e wlan.sa -e wlan.da -e wlan.bssid \
  | head -n 100 | tee reports/wlan_frame_sample.tsv
```

The sample showed type 1 frames (control frames) with subtype 11, meaning the first 100 frames were control frames used for flow control and acknowledgements.

> **Screenshot:** 802.11 frame sample
>
> ![802.11 frame sample](screenshots/11_wlan_frame_sample.png)

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

> **Screenshot:** Protected WEP frames and IV evidence
>
> ![Protected WEP frames](screenshots/12_wep_protected_frames.png)

### 6.2 Repeated IV Summary

```bash
tshark -r "$CAP" -Y 'wlan.fc.protected==1' -T fields -e wlan.wep.iv \
  | sort | uniq -c | sort -nr | head -n 30 | tee reports/repeated_iv_summary.txt
```

Many IVs appeared repeatedly, which is a known weakness of WEP. Repeated IVs mean the same RC4 keystream is reused, which leaks relationships between plaintexts and enables statistical key recovery.

> **Screenshot:** Repeated IV summary
>
> ![Repeated IV summary](screenshots/13_repeated_iv_summary.png)

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

> **Screenshot:** Aircrack-ng result
>
> ![Aircrack-ng result](screenshots/14_aircrack_result.png)

### 7.2 Key Validation

The recovered key was `A4:3D:F6:F3:74`. This matches the historical training key provided in the course slide deck.

```bash
printf 'A4:3D:F6:F3:74\n' | tee reports/validated_wep40_key_masked.txt
```

> **Note:** This key belongs to the historical CTF dataset and is not authorisation to attack any wireless network.

> **Screenshot:** Aircrack-ng result (key validation)
>
> ![Aircrack-ng key validation](screenshots/15_aircrack_key_validation.png)

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

> **Screenshot:** Airdecap-ng decryption result
>
> ![Airdecap-ng result](screenshots/16_airdecap_result.png)

### 8.2 Decrypted File Inventory

```bash
find working -maxdepth 1 -type f -ls | tee reports/decrypted_files_inventory.txt
```

- `working/file_working.xz`: 12,837,660 bytes
- `working/file_working-dec`: 12,784,501 bytes (decrypted capture)
- `working/file_working`: 14,234,002 bytes
- `working/file_working_copy`: 14,234,002 bytes

> **Screenshot:** Decrypted file inventory
>
> ![Decrypted file inventory](screenshots/17_decrypted_file_inventory.png)

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

> **Screenshot:** Hashes of all working files
>
> ![Working file hashes](screenshots/18_all_working_file_hashes.png)

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

> **Screenshot:** Ethernet endpoints
>
> ![Ethernet endpoints](screenshots/19_ethernet_endpoints.png)

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

> **Screenshot:** IP endpoints
>
> ![IP endpoints](screenshots/20_ip_endpoints.png)

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

> **Screenshot:** TCP conversations
>
> ![TCP conversations](screenshots/21_tcp_conversations.png)

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

> **Screenshot:** IP/MAC mapping sample
>
> ![IP/MAC mapping sample](screenshots/22_ip_mac_mapping_sample.png)

### 9.6 ARP and DHCP Evidence

The decrypted capture also revealed ARP requests and DHCP transactions:

```
0.000000 EFMNetworks_55:97:d4 > Broadcast ARP 42 Who has 192.168.0.15? Tell 192.168.0.1
0.944186 Apple_68:96:7c > NetisTechnol_48:e5:b4 ARP 42 Who has 192.168.0.1? Tell 192.168.0.15
0.944185 0.0.0.0 > 255.255.255.255 DHCP 342 DHCP Request - Transaction ID 0x21e6be1c
0.954435 192.168.0.1 > 255.255.255.255 DHCP 590 DHCP NAK - Transaction ID 0x21e6be1c
1.030202 192.168.0.15 > 8.8.8.8 DNS 73 Standard query 0x9583 A www.apple.com
```

> **Screenshot:** ARP, DHCP and DNS evidence
>
> ![ARP and DHCP evidence](screenshots/23_arp_dhcp_evidence.png)

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

> **Screenshot:** Exported file types
>
> ![Exported file types](screenshots/24_exported_file_types.png)

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

> **Screenshot:** Hashes of recovered files
>
> ![Exported file hashes](screenshots/25_exported_file_hashes.png)

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
