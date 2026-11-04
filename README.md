# BlueMoon: 2021 - CTF Lab Report

## Target Overview
* **Target System:** BlueMoon: 2021 (VulnHub)
* **Goal:** Full System Compromise (Root Access)
* **Final Status:** Completed / Root Obtained

---

## Executive Summary
The target system was fully compromised by identifying exposed web assets, extracting embedded authentication material, performing local credential enumeration, and leveraging system misconfigurations to achieve elevated privileges on the host filesystem.

---

## Key Findings & Attack Lifecycle

### 1. Reconnaissance & Web Discovery
* **Vector:** HTTP Service (Port 80)
* **Details:** Discovered a hidden directory (`/hidden_text`) hosting an image asset containing an embedded QR code (`.QR_C0d3.png`).

### 2. Credential Extraction & Initial Access
* **Vector:** QR Code Decoding / FTP / SSH
* **Details:** Decoded the QR code using `zbarimg` to recover valid FTP credentials. Retrieved internal documentation (`information.txt`) and user password lists (`p_lists.txt`) to authenticate via SSH as user `robin`.

### 3. Lateral Movement & Script Exploitation
* **Vector:** Local Script Execution
* **Details:** Analyzed a local feedback script (`/home/robin/project/feedback.sh`). Executed the script via `sudo` as user `jerry` and passed `/bin/bash` into the prompt to pivot from user `robin` to user `jerry`.

### 4. Privilege Escalation to Root
* **Vector:** Group Misconfiguration (Docker)
* **Details:** Identified user `jerry`'s membership in the `docker` administrative group. Spawns an interactive container mounting the host filesystem (`docker run -v /:/mnt --rm -it alpine chroot /mnt sh`) to access the host as `root`.

---

## Remediation Recommendations
* **Input Validation:** Ensure user input inside administrative shell scripts is strictly sanitized and quoted to prevent command execution.
* **Least Privilege:** Avoid adding standard user accounts to the `docker` group without strict access controls, as local Docker privileges are equivalent to full root access on the host.
* **Sensitive File Exposure:** Secure web server directories to prevent publicly indexing sensitive operational images and embedded credentials.
