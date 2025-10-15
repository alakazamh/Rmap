# Rmap — RustScan + Nmap wrapper

Rmap is a small Bash tool that simplifies port scanning by doing two steps:

1. Fast discovery: detect open ports (optionally using RustScan).  
2. Targeted scan: run a focused Nmap scan only on the discovered ports.

This two-step approach speeds up large-range discovery while keeping Nmap output focused and readable.

---

## Key points

- First pass: discover open ports and extract a compact list (grep-like extraction).  
- Second pass: run Nmap with `-sC -sV` only on the discovered ports for detailed service detection.  
- Output is filtered to remove repetitive/log noise and present a clean service table.  
- RustScan usage is optional — script works in Nmap-only mode if RustScan is not installed.

---

## Installation

Clone the repository and make the script executable:

```bash
git clone https://github.com/alakazamh/Rmap.git
cd Rmap
chmod +x rmap


## Install requirements (Debian / Kali / Ubuntu)
## Update package lists and install core tools:
sudo apt update
sudo apt install -y nmap figlet
```

RustScan may not be available in the distribution repositories. See below for RustScan installation options.

---

## RustScan: how to get it

Official RustScan GitHub:

Releases: https://github.com/RustScan/RustScan/releases
Installation guide (wiki): https://github.com/bee-san/RustScan/wiki/Installation-Guide

Download the latest .deb from the releases page and install:
```bash
# download from the releases page, then:
sudo dpkg -i rustscan_<version>_amd64.deb
sudo apt-get install -f -y    # fix dependencies if needed
```

Build from source (if you need the latest or no package is available):
```
git clone https://github.com/RustScan/RustScan.git
cd RustScan
cargo build --release
sudo cp target/release/rustscan /usr/local/bin/
```

---

## Usage

```bash
./rscanv2 [options] <target>

Main options:
-r, --rustscan
Use RustScan for discovery (default RustScan invocation: rustscan -a <target> -r 1-65535 --ulimit=5000).

--rust-args "<args>"
Extra arguments appended to RustScan invocation (quote the string).

--save-rust <file>
Save raw RustScan stdout to <file>.

--nmap-oN <file>
Save Nmap human-readable output to <file> (Nmap -oN).

--nmap-oA <prefix>
Save Nmap outputs in all formats using <prefix> (Nmap -oA).

-h, --help
Show usage.
```

Exemples
```bash
# Nmap-only discovery and targeted scan
./rmap 10.10.10.10

# RustScan discovery + Nmap targeted, save human-readable output
./rmap 10.10.10.10 -r --nmap-oN results.txt

# RustScan with extra args and save raw output
./rmap 10.10.10.10 -r --rust-args "--timeout 2000" --save-rust rust_raw.log

# Save all nmap outputs (.nmap .gnmap .xml)
./rmap 10.10.10.10 -r --nmap-oA scans/10.10.10.10
```


Notes and best practices:

Only scan hosts for which you have permission.

If RustScan is not installed, the script falls back to Nmap-only discovery; RustScan is optional, not required.

If you customize parsing rules, test them in an isolated lab first.

License

This project is licensed under the MIT License. See the LICENSE file for details.
