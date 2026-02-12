# Practical Windows Forensics (PWF) Lab Repository

*Provided by Blue Cape Security*

<p align="center">
  <img src="https://github.com/bluecapesecurity/bluecapesecurity/blob/main/BCS_banner.png" />
</p>

This repository gives you a practical, DIY workflow to generate Windows forensic evidence and start a structured investigation.

It is designed for practitioners who want to:
- Build a Windows lab
- Simulate attacker behavior
- Acquire memory and disk images
- Begin forensic analysis with realistic artifacts

## Important Update
PWF is no longer a standalone DIY course.

PWF is now part of the **Analyst 1 Training Track** at Blue Cape Security, which includes:
- Practical Windows Forensics (PWF)
- FOR200 Windows Forensic Investigation Scenarios
- PWFA certification (a 7-day practical Windows forensics exam focused on delivering a meticulous forensic timeline)

If you want guided instruction for the analysis process, enroll in the Analyst 1 track:
- https://bluecapesecurity.com/analyst1/

You can also explore additional free and paid SOC/DFIR training at:
- https://bluecapesecurity.com/

## What You Can Do With This Repo (Free)
You can still use this repository for self-paced lab execution:
1. Set up your lab using Blue Cape Security free tutorials.
2. Run the attack simulation script.
3. Acquire memory and disk images.
4. Start analysis on your forensic workstation.

If you want expert-led analysis training, use the Analyst 1 Training Track link above.

## Prerequisites
- Virtualization platform: [VirtualBox or VMWare](https://bluecapesecurity.com/build-your-lab/virtualization/)
- Host system resources:
  - 4 GB+ RAM for running Windows VMs (the two VMs do not need to run at the same time)
  - Storage for:
    - 2 Windows VMs (~20 GB and ~40 GB)
    - Evidence files and working data (~30 GB additional)

## Investigation Roadmap
![Investigation Roadmap](Investigation-roadmap.png)

## Free Cheat Sheet (Quick Access)
This repository includes a free Practical Windows Forensics cheat sheet:
- PDF in this repo: `Resources/PracticalWindowsForensics-cheat-sheet.pdf`
- Markdown/Notion version: https://rogue-foundation-4c0.notion.site/Practical-Windows-Forensics-Cheat-Sheet-2601ffcde5e280d38d9bc4bfa56a4712?pvs=149

## Attack Scenario
The attack simulation script creates a realistic compromise scenario on a Windows target by executing selected Atomic Red Team tests.

- Script: `AtomicRedTeam/ART-attack.ps1`
- It installs [Invoke-AtomicRedTeam](https://github.com/redcanaryco/invoke-atomicredteam) and runs selected ATT&CK techniques.

![Attack Script](AtomicRedTeam/PWF_Analysis-MITRE.png)

## Workflow

### 1) Prepare Target System
1. Download and install a Windows 10 Enterprise Evaluation VM:
   - https://www.microsoft.com/en-us/evalcenter/evaluate-windows-10-enterprise
2. In VirtualBox, create a new VM and install Windows 10 from the ISO.
3. Take a clean snapshot after setup.
4. Pause Windows Updates to reduce noise:
   - `Settings -> Windows Update -> Advanced Options -> Pause updates`
5. Install Sysmon for detailed event logging:
   - Copy `Install-Sysmon/Install-Sysmon.ps1` to the target VM
   - Run PowerShell as Administrator and execute:
     - `./Install-Sysmon.ps1`
6. Temporarily disable Defender settings before attack execution:
   - `Virus & threat protection settings -> Manage settings -> Disable all shown features`

### 2) Execute Attack Script
1. Copy `AtomicRedTeam/ART-attack.ps1` to the target VM.
2. Run PowerShell as Administrator and execute:
   - `./ART-attack.ps1`
3. Ensure internet access on the target VM (required for downloading Invoke-AtomicRedTeam components).
4. Accept prompts if PowerShell asks to install additional features.
5. Verify atomic tests completed successfully in the script output.
6. Do not close spawned windows/processes; continue to acquisition.

### 3) Acquire Memory and Disk Evidence
1. Pause (VirtualBox) or suspend (VMWare) the target VM and take a snapshot.
2. Create an evidence folder on your host.

#### 3.1 Memory Acquisition
VMWare:
- Open the VM's `.vmwarevm` directory.
- Copy the `.vmem` and matching `.vmsn` snapshot file into your evidence folder.

VirtualBox:
- Identify VM UUID:
  - `vboxmanage list vms`
- Dump memory:
  - `vboxmanage debugvm <VM_UUID> dumpvmcore --filename win10-mem.raw`

#### 3.2 Disk Acquisition
- Unpause and shut down the target VM.

VMWare:
- Locate all split `*.vmdk` files.
- Export one of the following:
  - Copy all split files for the latest sequence (`Virtual Disk-xxx.vmdk` to `Virtual Disk-xxx-s00xx.vmdk`) to evidence.
  - Or create a single VMDK:
    - `"C:\Program Files (x86)\VMware\VMware Player\vmware-vdiskmanager.exe" -r "d:\VMLinux\vmdkname.vmdk" -t 0 MyNewImage.vmdk`

VirtualBox:
- Identify VM UUID:
  - `vboxmanage list vms`
- Identify disk UUID:
  - `vboxmanage showvminfo <VM_UUID>`
- Clone disk to RAW:
  - `vboxmanage clonemedium disk <disk_UUID> --format raw win10-disk.raw`

#### 3.3 Integrity Validation
Create SHA1 hashes and store them with your evidence.

Windows (PowerShell):
- `Get-FileHash -Algorithm SHA1 <file>`

Mac/Linux:
- `shasum <file>`

### 4) Set Up Forensic Workstation
Use this guide:
- https://bluecapesecurity.com/build-your-forensic-workstation/

Recommended baseline:
- Windows Server 2019 VM
- VirtualBox VM sizing: at least 4 GB RAM and 100 GB dynamically allocated disk
- Configure shared folders/clipboard and connect your evidence folder
- Install core DFIR tooling (for example):
  - Kali Linux subsystem + Volatility
  - Arsenal Image Mounter
  - FTK Imager
  - Eric Zimmerman tools
  - KAPE
  - RegRipper
  - EventLog Explorer
  - Notepad++
- Take a snapshot after tool setup

### 5) Start Analysis
Focus on artifacts such as:
- User accounts
- Program execution
- Persistence (run keys, scheduled tasks, startup scripts, services)
- NTFS file creation/deletion artifacts
- PowerShell activity
- DLL injection indicators
- Office document artifacts
- Timeline development

Helpful resources in this repository:
- `Resources/PracticalWindowsForensics-cheat-sheet.pdf`
- `Resources/PracticalWindowsForensics-Objectives.csv`
- `Resources/Analysis-Notes-Template.docx`
- `Resources/RegRipper-plugins.csv`
- `Resources/RegRipper-plugins.xlsx`

## Learn More (Blue Cape Security)
- Analyst 1 Training Track (includes PWF + FOR200 + PWFA): https://bluecapesecurity.com/analyst1/
- All Blue Cape Security trainings (free + paid): https://bluecapesecurity.com/
- Discord community: https://discord.gg/WKsaGE2CV3
- Intro video: https://youtu.be/JDzJHyBJIXk

## License
This project is provided by Blue Cape Security and is free for anyone to use under the terms in `License.md`.

## Disclaimer
This material is for educational purposes only. It is provided without warranty, and Blue Cape Security disclaims liability for damages or misuse.
