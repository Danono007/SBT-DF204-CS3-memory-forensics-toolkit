# SBT-DF204 Case Study 3: Windows Memory Forensics Toolkit

Reusable shell tooling and method notes for examining a Windows memory image with **Volatility 2** in a computer forensics workflow. Built for **SBT-DF204 Computer Forensics** coursework (case study: reconstructing a suspected file transfer from memory evidence).

> **Academic note.** This is student coursework on a training dataset. The repository contains **no memory image, no case findings, no screenshots and no report**. It only contains generic, read-only tooling. Do not copy it as submitted work for your own assignment.

## Contents

| Path | Description |
|------|-------------|
| `README.md` | This file |
| `scripts/collect_evidence.sh` | Runs a standard set of read-only Volatility 2 plugins and saves each output to `output/` |
| `.gitignore` | Keeps memory images, evidence, screenshots, reports and outputs out of git |
| `LICENSE` | MIT |

## What the script does

| Section | Plugins | Purpose |
|---------|---------|---------|
| 1 | `sha256sum`, `imageinfo` | Integrity record and profile identification |
| 2 | `pslist`, `psscan`, `pstree`, `hivelist` | Processes and registry hives (profile validation) |
| 3 | `printkey`, `getsids` | Time zone, user environment, profile list, SIDs |
| 4 | `netscan`, `cmdline`, `cmdscan`, `consoles` | Network artefacts and command history |
| 5 | `printkey`, `shellbags` | USBSTOR, DeviceClasses, MountedDevices, MountPoints2, ShellBags |
| 6 | `filescan`, `strings` | Optional file-object scan and string search |

## Evidence handling (recommended workflow)

1. Hash the original image: `sha256sum image.mem`, then `chmod 444 image.mem`.
2. Work only on a copy (`working/image_working.mem`) and compare the hashes.
3. Run every plugin against the working copy.
4. Re-hash the original at the end and confirm it is unchanged.

## Requirements

- Linux (developed on Kali)
- Volatility 2.6.x run with `python2 vol.py`
- `bash`, `sha256sum`, `strings`
- Python 2 package `pycryptodome` for plugins such as `shellbags`, `getsids` and `userassist`
- `distorm3` is optional (only needed for `apihooks`, `threads`, `ssdt`)

## Usage

```bash
mkdir -p original working output
cp /path/to/image.mem original/image.mem
sha256sum original/image.mem
chmod 444 original/image.mem
cp original/image.mem working/image_working.mem

chmod +x scripts/collect_evidence.sh
export IMAGE=working/image_working.mem
export VOL_DIR=~/tools/volatility

./scripts/collect_evidence.sh 1                       # hash and imageinfo
export PROFILE=<profile chosen from imageinfo and validated>
./scripts/collect_evidence.sh                         # all sections

# optional settings
HIVE=<NTUSER.DAT virtual offset> ./scripts/collect_evidence.sh 5
SEARCH=<string> ./scripts/collect_evidence.sh 6
```

## Method notes

- Choose the profile from `imageinfo`, then validate it: `pslist`, `psscan`, `pstree` and `hivelist` should agree and look sensible.
- Volatility times are UTC. Read the image time zone from the registry (`TimeZoneInformation`) and convert separately.
- Registry key last-write times are not event times. ShellBags show folder views, not file modification times.
- A command in command history or a console buffer does not by itself prove the action completed. It has no timestamp.
- USBSTOR instance IDs with `&` as the second character are Windows-generated and carry no unique hardware serial.
- A memory capture is not an instant snapshot. Check process start times for clock anomalies.
- Artefacts identify an account or profile, not the person at the keyboard.

## Safety

Do not contact any IP address, host, account or service found in a memory image. Do not execute recovered files and do not use password or hash-cracking steps. Analyse everything offline, as text.


## Licence

MIT. See `LICENSE`.
