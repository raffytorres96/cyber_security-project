# 🔐 Cybersecurity — Red Team & Blue Team

> A complete cybersecurity case study covering an offensive **Red Team** attack simulation against a public school institution, and a defensive **Blue Team** serious game concept — designed for security awareness training.

---

## 📌 Overview

This project was developed for the course **Cyber Security 2023/2024** at the **University of Bari** (Prof.ssa Vita Barletta). It explores the two fundamental pillars of cybersecurity operations — the **Red Team** (offensive) and the **Blue Team** (defensive) — applied to a concrete, realistic case study.

**Author:** Raffaele Gatta
**Team:** NetNerd
**Target:** Ente pubblico scolastico primario (Scuola Montello, Bari)
**Course:** Cyber Security — University of Bari (A.Y. 2023/24)

---

## 🗂️ Table of Contents

1. [Red Team — Attack Simulation](#-red-team--attack-simulation)
   - [Context & Objective](#context--objective)
   - [Kill Chain — 7 Phases](#kill-chain--7-phases)
   - [Tools Used](#tools-used)
   - [Final Considerations](#final-considerations)
2. [Blue Team — Serious Game: DigitalDefender](#-blue-team--serious-game-digitaldefender)
   - [Learning Aspects](#learning-aspects)
   - [Gaming Aspects](#gaming-aspects)
   - [Technological Aspects](#technological-aspects)
3. [Key Takeaways](#-key-takeaways)

---

## 🔴 Red Team — Attack Simulation

The Red Team simulates a real-world offensive operation to identify and exploit vulnerabilities in a target organization, testing its defensive capabilities.

### Context & Objective

The attack targets the **Argo** school management platform used by the "Scuola Montello" primary school in Bari. The objective is to gain unauthorized access to the school's internal network, specifically targeting the accounts of **secretariat staff and teachers**, in order to:

- Read and modify student grades and disciplinary notes
- Exfiltrate credentials and sensitive documents
- Navigate the employee hierarchy to reach higher-privilege accounts

The attack vector chosen is **spear phishing via email**, exploiting the **human factor** as the primary vulnerability — particularly relevant given that Italian teachers have an average age of 51, making them statistically less familiar with modern cybersecurity threats.

---

### Kill Chain — 7 Phases

#### 1️⃣ Reconnaissance

**Goal:** Collect information about the target.

| Source | Information gathered |
|---|---|
| LinkedIn | School employees identified by changing university affiliation on attacker's profile |
| Facebook / Instagram | Personal email addresses and contact details |
| Public school website | Staff roles, department structure |

A large list of employee targets (secretariat, teachers, management) was assembled before moving to the next phase.

**Attacker OS:** Kali Linux — chosen for its built-in offensive security toolset (vulnerability scanning, password analysis, network probing).

**Victim OS:** Microsoft Windows — targeted as the most widespread OS in Italian public institutions, and statistically the most vulnerable due to misconfiguration and low user security awareness.

---

#### 2️⃣ Weaponization

**Goal:** Create the malicious payload.

**Tool:** Social Engineering Toolkit (SET)
**Technique:** `Windows Meterpreter Reverse_TCP X64` (MITRE ATT&CK: T0865)

**Steps:**
1. Open SET → `Social-Engineering Attacks (1)`
2. Select `Create a Payload and Listener (4)`
3. Choose payload `Windows Meterpreter Reverse_TCP X64 (5)`
4. Configure attacker IP (`ifconfig`) and port (discovered via Nmap)
5. Activate the Meterpreter listener via Metasploit — remains on standby

**Output:** `payload.exe` — a Windows executable that, once run by the victim, opens a reverse TCP session to the attacker's machine, granting full remote control.

---

#### 3️⃣ Delivery

**Goal:** Deliver the payload to the target.

**Method:** Spear phishing email impersonating the official Argo platform.

| Element | Detail |
|---|---|
| Spoofed sender | `info@argosoft.info` (visually indistinguishable from official) |
| Email content | Fake security alert: "Your Argo password has been changed" |
| Attachment | `Location.rar` (password-protected to bypass email scanners) |
| Password | Included in the email body — lowers suspicion |
| Archive contents | `Report.pdf` (fake login alert) + `Location.exe` (payload) |

The **direct download link** was generated via Google Drive using the following technique:

```
Original link:  https://drive.google.com/file/d/{FILE_ID}/view?usp=sharing
Direct link:    https://drive.google.com/uc?export=download&id={FILE_ID}
```

This bypasses Google's preview page and triggers an automatic download on click.

The email footer was cloned from Argo's official website, with the contact email replaced by the attacker's — ensuring any suspicious user who attempts to verify would contact the attacker instead.

---

#### 4️⃣ Exploitation

**Goal:** Get the victim to execute the payload.

The exploitation relies entirely on the **human factor**. The phishing email includes a call to action to "locate the device that accessed your account in real time" — designed to trigger curiosity or alarm in the victim.

Once `Location.exe` is executed:
- **Victim side:** The file appears to silently fail — no visible effect on the desktop, unlikely to raise suspicion
- **Attacker side:** A new Meterpreter session opens for each execution, showing the victim's IP

> ⚠️ Note: This exploit is effective only on **outdated/unpatched Windows systems** and is unlikely to be detected by traditional antivirus tools on legacy machines.

---

#### 5️⃣ Installation

**Goal:** Maintain access on the compromised system.

Traditional persistence mechanisms (malware running at startup) are difficult to implement reliably on modern Windows. The strategy adopted is **session-based persistence**: remain active during the open session, extracting all necessary data before the connection closes.

---

#### 6️⃣ Command & Control (C2)

**Goal:** Control the compromised system.

The attacker uses the victim's own authenticated accounts (email, social networks) — captured during the session — to expand lateral movement and communicate through trusted channels, blending in with legitimate traffic.

---

#### 7️⃣ Actions on Objectives

**Goal:** Achieve the attack's final objectives.

Full list of Meterpreter commands available post-exploitation:

| Command | Action |
|---|---|
| `pwd` / `cd` | Navigate remote filesystem |
| `download` / `upload` | Transfer files to/from victim |
| `search` | Find files on remote filesystem |
| `screenshot` | Capture victim's desktop |
| `keyscan_start` / `keyscan_dump` | Keylogging — capture all keystrokes |
| `webcam_snap` / `webcam_stream` | Activate victim's webcam |
| `record_mic` | Record microphone audio |
| `shell` | Open victim's command prompt |
| `getsystem` | Attempt privilege escalation |
| `killav` | Disable antivirus |
| `getcontermeasure` | Disable firewall and other security measures |
| `clearev` | Clear all system, application and security logs |
| `execute enum_firefox` | Extract Firefox cookies, history, saved passwords |
| `execute get_application_list` | List all installed software |
| `ifconfig` / `sysinfo` / `getuid` | System reconnaissance |
| `ps` | List running processes |

---

### Tools Used

| Tool | Purpose |
|---|---|
| **Kali Linux** | Attacker operating system |
| **Social Engineering Toolkit (SET)** | Payload and listener generation |
| **Metasploit / Meterpreter** | C2 framework and post-exploitation |
| **Nmap** | Port scanning and service discovery |
| **Google Drive** | Hosting the payload via direct download link |
| **LinkedIn / Facebook / Instagram** | OSINT — employee reconnaissance |

---

### Final Considerations

The attack demonstrated that:

- **Human factor** is the most exploitable vulnerability in any organization
- The **Cyber Kill Chain** framework proved essential for structured, methodical execution
- The technique is **scalable** — the same approach applies to any school using the Argo platform
- Lateral movement toward higher-privilege accounts is a natural attack progression
- Limitations include session-based (non-persistent) access and reduced effectiveness on patched, modern OS

---

## 🔵 Blue Team — Serious Game: DigitalDefender

The Blue Team response takes the form of a **serious game concept** designed to train and sensitize employees and general users on cybersecurity best practices.

### Learning Aspects

**Target audience:** Employees of public/private organizations with access to sensitive data (16+), general tech users.

**Topics covered:**

| Topic | Learning objective |
|---|---|
| **Phishing** | Recognize suspicious emails, links, and attachments |
| **Malware** | Understand infection vectors; use and maintain antivirus |
| **Social Engineering** | Identify manipulation tactics; avoid unauthorized disclosure |
| **Data Protection** | Strong passwords, data encryption, regular backups |
| **Network Security** | Firewalls, secure Wi-Fi, access management |
| **User Responsibility** | Cybersecurity as a shared, daily practice |

**Overall learning goal:** Equip players with practical knowledge to protect their personal data, organizational assets, and digital identity against real-world cyber threats.

---

### Gaming Aspects

#### Setting — NeoCentral

A futuristic virtual metropolis of glowing skyscrapers, foggy skylines, elevated bridges and interactive holograms. Technology permeates every aspect of daily life. This is the world the player must protect.

#### Characters

| Character | Role |
|---|---|
| **Evilhack** | The antagonist — a dangerous hacker specializing in phishing and social engineering, hiding in the shadows of NeoCentral |
| **NeoProtectors** | The player's team — digital warriors trained to defend NeoCentral from cyber threats |
| **Kyborg** | The AI guide — introduces the player to NeoCentral and the NeoProtectors mission |

#### Story

NeoCentral is hit by a wave of phishing attacks orchestrated by **Evilhack**. The player joins the **NeoProtectors** and undergoes training missions to identify threats, analyze digital traces, and ultimately locate and defeat Evilhack in his secret hideout — restoring security and awareness across the city.

#### Gameplay Mechanics

| Element | Description |
|---|---|
| **Tutorial** | Introduces phishing risks, antivirus importance, executable file dangers |
| **Missions** | Real-world scenarios — analyzing suspicious emails, identifying malware, resolving cryptographic puzzles |
| **Challenge types** | Drag-and-drop puzzles, simulated CLI quizzes, multiple-choice questions |
| **Progression** | XP accumulation → unlock new skills, advanced security tools, new missions |
| **Customization** | NeoProtector status and inventory managed via the in-game menu |
| **Leaderboard** | Global ranking with real discount vouchers for top players (partner companies) |

---

### Technological Aspects

The game is designed for multi-platform deployment:

| Platform | Notes |
|---|---|
| **PC / Laptop** | Full graphical experience with advanced rendering |
| **Console** (PS / Xbox / Switch) | Local multiplayer support |
| **Mobile** (iOS / Android) | Accessible anywhere, optimized for touch |

---

## 💡 Key Takeaways

> *"Uno degli argomenti di riflessione è stato quanto il fattore e la psicologia umana incida sugli sviluppi degli attacchi."*

- A technically simple phishing campaign can be devastatingly effective when aimed at low-awareness targets
- The **Cyber Kill Chain** provides a structured methodology that prevents critical oversights during an attack — or a defense
- Cybersecurity is not only a technical problem — it is a **human and organizational one**
- Serious games like **DigitalDefender** offer an engaging, scalable approach to closing the human vulnerability gap

---

## 👤 Author

**Raffaele Gatta** — Team: NetNerd
Computer Science — University of Bari
Course: Cyber Security (A.Y. 2023/24) — Prof.ssa Vita Barletta

---

<p align="center">
  <img src="https://img.shields.io/badge/Kali_Linux-557C94?style=flat-square&logo=kalilinux&logoColor=white"/>
  <img src="https://img.shields.io/badge/Metasploit-2596CD?style=flat-square&logo=metasploit&logoColor=white"/>
  <img src="https://img.shields.io/badge/Red_Team-CC0000?style=flat-square&logoColor=white"/>
  <img src="https://img.shields.io/badge/Blue_Team-0057A8?style=flat-square&logoColor=white"/>
  <img src="https://img.shields.io/badge/MITRE_ATT%26CK-FF6600?style=flat-square&logoColor=white"/>
</p>
