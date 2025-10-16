# Rmap:  RustScan + Nmap wrapper

Rmap is a small Bash tool that simplifies port scanning by performing two steps:

1. **Fast discovery** — detect open ports (optionally with RustScan).  
2. **Targeted scan** — run a focused Nmap scan only on the discovered ports to get service/version details.

This two-step approach speeds up large-range discovery while keeping Nmap output focused and readable.

---

## Key points

- First pass: discover open ports and extract a compact list (grep-like extraction).  
- Second pass: run Nmap with `-sC -sV` only on the discovered ports for detailed service detection.  
- Output is filtered to remove repetitive/log noise and present a clean service table.  
- RustScan usage is optional — script works in Nmap-only mode if RustScan is not installed.

---

##  Key additions

- **`--short` (`-s`) mode**: run discovery only and **skip** the targeted Nmap scan. Useful to quickly get the list of open ports without launching a full Nmap probe.
- `--nmap-oN` and `--nmap-oA` let you save Nmap output (human-readable or all formats).
- `--rust-args` allows appending custom arguments to RustScan.
- `--save-rust` saves raw RustScan stdout for later analysis.

---

## Requirements

- `bash`, `nmap`, `awk`, `sed`, `grep`, `sort`, `paste`, `mktemp`, `pkill`
- `figlet` (optional, for banner)
- RustScan (optional but recommended for faster discovery)

---

## Installation


Clone the repository and make the script executable:

```bash
git clone https://github.com/alakazamh/Rmap.git
cd Rmap
chmod +x rmap


## Install requirements (Debian / Ubuntu)
## Update package lists and install core tools:
sudo apt update
sudo apt install -y nmap figlet
```

RustScan may not be available in the distribution repositories. See below for RustScan installation options.

---

### RustScan: how to get it

Installation guide (wiki): https://github.com/bee-san/RustScan/wiki/Installation-Guide

- **linux:**
Download the latest .deb from the releases page and install:
```bash
# download from the releases page, then:
sudo dpkg -i rustscan_<version>_amd64.deb
sudo apt-get install -f -y    # fix dependencies if needed
```

- **macOS (Homebrew):**
  ```bash
  brew install rustscan
  ```

---

## Important — privileges for Nmap-Only mode

The targeted Nmap scan uses a SYN scan (-sS) by default. SYN scans require raw socket privileges (root / sudo). Run Rmap with sudo (or as root) to ensure Nmap can perform -sS. Running without sufficient privileges may force Nmap to use a connect scan (-sT) or fail, which changes scan characteristics and may affect accuracy.

---

## Usage

```bash
./rmap [options] <target>

Main options:
-s, --short
Discovery only. Skip the targeted Nmap scan. Output: compact comma-separated list of discovered ports.

-r, --rustscan
Use RustScan for discovery (default RustScan invocation: rustscan -a <target> -r 1-65535 --ulimit=5000).

--rust-args "<args>"
Extra arguments appended to RustScan invocation (quote the string).

--save-rust <file>
Save raw RustScan stdout to <file>.

--nmap-oN <file>
Save Nmap human-readable output to <file> (Nmap -oN).

--nmap-oA <prefix>
Save Nmap outputs in all formats (-oA <prefix> produces <prefix>.nmap, <prefix>.gnmap, <prefix>.xml).

-h, --help
Show usage.
```

Exemples:
```bash
## Rustscan mode:
# 1) Quick discovery only (Rustscan + short mode)
./rmap -s -r 10.10.10.10 # for saving raw output add flag '--save-rust <file>'

# 2) Full discovery (RustScan) + targeted Nmap scan, saving human-readable output
./rmap -r --nmap-oN results.txt 10.10.10.0/24

# 3) Full discovery (RustScan) with extra args and save raw output
./rmap -r --rust-args "--timeout 2000" --save-rust rust_raw.log 10.10.10.10

# 4) Save all Nmap outputs (.nmap .gnmap .xml)
./rmap -r --nmap-oA scans/10.10.10.10 10.10.10.10

## Nmap mode only
# 1) Full discovery, saving human-readable output
sudo ./rmap --nmap-oN 10.10.10.10 

---

Output & Files

Temporary files are created under `/tmp/rmap.*` and cleaned automatically on exit.

`--save-rust` will copy RustScan's raw stdout to the path you provide.

`--nmap-oN` / `--nmap-oA` control where Nmap writes its final scan outputs.

---


## Safety and best practices:

**Authorization:** Only scan targets you have explicit permission to test. Unauthorized scanning can be illegal.

**Testing environment:** When modifying parsing rules, test in an isolated lab (VMs / test ranges like 10.0.0.0/8 or VulnHub) before using on production.

**Rate limits & noise:** Default probe uses fairly aggressive rates (--min-rate / --ulimit). Adjust rust_args or the nmap probe flags if you need to reduce network load or avoid IDS triggers.

**Logging:** Keep --save-rust or Nmap outputs when reproducing findings; raw data helps triage parsing anomalies.

**Privilege awareness:** Running with sudo changes scan behavior and network side effects — document the privilege used when reporting.