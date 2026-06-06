# Pikaptcha: A Lab About Fake Captchas

**Course:** Network Security  
**Type:** Project Presentation / Lab Report

---

## Table of Contents

- [Overview](#overview)
- [Task 1: Registry Artifacts – Payload Execution Method](#task-1-registry-artifacts--payload-execution-method)
- [Task 2: Time of Malicious Payload Execution (UTC)](#task-2-time-of-malicious-payload-execution-utc)
- [Task 3: SHA256 Hash of the Downloaded PowerShell Script](#task-3-sha256-hash-of-the-downloaded-powershell-script)
- [Task 4: Reverse Shell Port](#task-4-reverse-shell-port)
- [Task 5: Duration of Reverse Shell Connection](#task-5-duration-of-reverse-shell-connection)
- [Task 6: Malicious Clipboard Function Name](#task-6-malicious-clipboard-function-name)

---

## Overview

The objective of this lab is the understanding and analysis of a **Fake Captcha** attack. The lab teaches how to detect suspicious behaviour logged on a system and trace it back to its source — in this case, a fake captcha page.

**Scenario:**  
A user contacted the sysadmin because of issues downloading what appeared to be the latest version of Microsoft Office. The user had received an email saying they needed to update and clicked the link. They reported visiting the website and solving a captcha, but no Office download page appeared. The sysadmin, aware of attacker tactics, immediately notified the security team to isolate the machine.

**Provided artifacts:**
- A `.pcap` network capture file
- An encrypted collection of system-related (KAPE) files  
  *(Password provided on the lab webpage)*

---

## Task 1: Registry Artifacts – Payload Execution Method

**Goal:** Identify how the malicious payload was executed using registry artifacts.

Since the investigation targets a specific user, the KAPE files were examined for the relevant user profile.

**Tool used:** Registry Explorer (to examine hive files and log files)

Under the **Available Bookmarks** tab, forensically important registry data can be reviewed. Payload execution typically appears in one of three registry keys:

| Registry Key | Purpose |
|---|---|
| `Run` | Programs that execute on every login |
| `RunOnce` | Programs that execute once on next login |
| `RunMRU` | Most Recently Used entries from the Run dialog |

Of the three, only **RunMRU** showed a payload being passed. This indicates the script was **copied and pasted into the Windows Run dialog** by the malicious captcha page (clipboard injection technique).

---

## Task 2: Time of Malicious Payload Execution (UTC)

**Goal:** Determine the UTC timestamp of when the malicious payload executed.

The execution timestamp is stored alongside the RunMRU entry in the registry hive. The time can be read directly from the Registry Explorer interface next to the script entry.

---

## Task 3: SHA256 Hash of the Downloaded PowerShell Script

**Goal:** Identify the PowerShell script downloaded and executed in memory by the initial payload.

**Tool used:** Wireshark

Steps taken:
1. Loaded the `.pcap` file into Wireshark.
2. Filtered traffic by the attacker's known IP address.
3. Observed initial HTTP traffic followed by repeated TCP streams on a single unique port — indicative of a persistent C2 connection.
4. Exported the suspicious file from the HTTP traffic.

The downloaded file was **Base64-encoded**. After decoding, the content revealed a **PowerShell-based reverse shell** designed to give the attacker an interactive remote code execution session.

The **SHA256 hash** was computed from the decoded script file using an online hashing tool.

---

## Task 4: Reverse Shell Port

**Goal:** Identify the port to which the reverse shell connected.

The target port is hardcoded inside the malicious PowerShell script retrieved in Task 3. Reviewing the decoded script reveals the outbound port used for the C2 connection.

---

## Task 5: Duration of Reverse Shell Connection

**Goal:** Determine how long the reverse shell connection was active (in seconds).

**Tool used:** Wireshark (filtered by attacker IP + reverse shell port)

By examining the timestamps of:
- The **first packet** of the connection
- The **final packet** of the connection

The calculated duration was:

> **6 minutes and 43 seconds = 403 seconds**

---

## Task 6: Malicious Clipboard Function Name

**Goal:** Identify the JavaScript function on the fake captcha page that injected the payload into the victim's clipboard.

**Tool used:** Wireshark (filtered by attacker IP + HTTP traffic)

Steps taken:
1. Filtered traffic to show only HTTP requests/responses from the attacker's server IP.
2. Located the packet where the attacker's server responded with **HTTP 200 OK**.
3. Expanded the packet details and examined the HTTP response body (the fake captcha page source).
4. Identified the JavaScript function responsible for writing the malicious payload to the clipboard.

---

## Key Takeaways

- **Fake Captcha attacks** leverage social engineering to trick users into running malicious commands via clipboard injection.
- **RunMRU** registry entries are a valuable forensic artifact for detecting Run dialog abuse.
- **PowerShell reverse shells** encoded in Base64 are a common evasion technique.
- Network analysis (pcap) combined with endpoint forensics (KAPE/registry) provides a complete picture of the attack chain.

---

## Tools Used

| Tool | Purpose |
|---|---|
| Registry Explorer | Analyse Windows registry hive files |
| Wireshark | Network traffic analysis (pcap) |
| Base64 decoder | Decode the obfuscated PowerShell payload |
| SHA256 hash tool | Generate hash of the downloaded script |
