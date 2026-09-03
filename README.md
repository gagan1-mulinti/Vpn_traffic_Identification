# 🔍 VPN Traffic Identification

> A machine learning-based network traffic classifier that detects VPN flows in real time using Random Forest and NFStream.

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python&logoColor=white)
![License](https://img.shields.io/badge/License-Educational-green)
![Windows](https://img.shields.io/badge/Platform-Windows-0078D6?logo=windows&logoColor=white)
![Linux](https://img.shields.io/badge/Platform-Linux-FCC624?logo=linux&logoColor=black)
![ML](https://img.shields.io/badge/Model-Random%20Forest-orange?logo=scikit-learn)

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Project Structure](#-project-structure)
- [Prerequisites](#-prerequisites)
- [Installation](#-installation)
- [Training the Model](#-training-the-model)
- [Running Live Detection](#-running-live-detection)
- [Feature Reference](#-feature-reference)
- [How It Works](#-how-it-works)
- [Troubleshooting](#-troubleshooting)
- [License](#-license)

---

## 🧠 Overview

**VPN Traffic Identification** captures live network flows and classifies each one as either `VPN` or `Non-VPN` using a trained Random Forest model. It leverages **NFStream** for flow-level packet capture and **scikit-learn** for model training and inference.

This project is suitable for:
- Network traffic analysis research
- Understanding VPN detection techniques
- Experimenting with flow-based ML classification

Supported platforms: **Windows 10/11** and **Linux** (Ubuntu, Debian, Fedora, Arch, and similar distributions).

---

## 📁 Project Structure

```
vpn-traffic-identification/
│
├── model.py                      # Train the Random Forest classifier
├── test.py                       # Run live traffic capture and classification
├── consolidated_traffic_data.csv # Labeled dataset for training
├── requirements.txt              # Python dependencies
│
├── vpn_rf_model.pkl              # ← Generated after training
└── label_encoder.pkl             # ← Generated after training
```

---

## ✅ Prerequisites

### Common (both platforms)

| Requirement | Details |
|---|---|
| Python | 3.10 or later |
| Privileges | Administrator / root rights for live capture |
| Network adapter | A compatible adapter with a known interface name |

> **Tip:** Always use a Python virtual environment to avoid dependency conflicts.

---

### 🪟 Windows-specific

| Requirement | Details |
|---|---|
| OS | Windows 10 or Windows 11 |
| Packet capture driver | **Npcap** (replaces WinPcap) |

Download Npcap from: https://npcap.com/#download

During installation:
- ✅ Check **"Install Npcap in WinPcap API-compatible Mode"**
- Reboot if prompted

> Without Npcap, `test.py` will fail with a driver or interface error immediately on launch.

---

### 🐧 Linux-specific

| Requirement | Details |
|---|---|
| OS | Ubuntu 20.04+, Debian 11+, Fedora 36+, Arch, or similar |
| Packet capture library | `libpcap-dev` |
| Runtime privileges | `sudo` or `CAP_NET_RAW` capability |

Install `libpcap` using your distro's package manager:

```bash
# Ubuntu / Debian
sudo apt update && sudo apt install libpcap-dev -y

# Fedora / RHEL / CentOS
sudo dnf install libpcap-devel -y

# Arch Linux
sudo pacman -S libpcap
```

> `nfstream` on Linux uses `libpcap` directly — no additional driver is needed beyond this.

---

## ⚙️ Installation

### 1. Clone the repository

```bash
git clone https://github.com/your-username/vpn-traffic-identification.git
cd vpn-traffic-identification
```

---

### 2. Create and activate a virtual environment *(recommended)*

<table>
<tr><th>🪟 Windows (PowerShell)</th><th>🐧 Linux / macOS (bash)</th></tr>
<tr>
<td>

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

</td>
<td>

```bash
python3 -m venv .venv
source .venv/bin/activate
```

</td>
</tr>
</table>

---

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

#### `requirements.txt` — annotated

```text
# Core dependencies for training and live detection
# Use compatible versions for broad platform support.
# Pin exact versions if you need reproducible environments.

pandas>=1.5.3,<=2.1.0          # DataFrame handling for CSV loading and preprocessing
numpy>=1.24.0,<=2.4.2          # Numerical operations and feature arrays
joblib>=1.2.0                  # Model serialization (.pkl save/load)
scikit-learn>=1.2.2            # Random Forest classifier and label encoder
nfstream>=6.6.0                # Live packet capture and flow-level feature extraction

# Optional utilities
tqdm>=4.60.0                   # Progress bars during training (optional but recommended)
python-dateutil>=2.8.2         # Date parsing utilities used internally by pandas
```

> **Version pinning tip:** For a fully reproducible environment (e.g., CI/CD, shared deployments), freeze your exact installed versions:
> ```bash
> pip freeze > requirements-lock.txt
> # Restore later with:
> pip install -r requirements-lock.txt
> ```

---

## 🏋️ Training the Model

Training is identical on both platforms.

### Step 1 — Prepare the dataset

Ensure `consolidated_traffic_data.csv` is present in the repository root. This file must contain labeled network flow data with the expected feature columns (see [Feature Reference](#-feature-reference)).

### Step 2 — Run the training script

```bash
python model.py        # Windows
python3 model.py       # Linux
```

### What happens during training

```
consolidated_traffic_data.csv
        │
        ▼
  Load & Preprocess
  (fill missing feature columns with 0)
        │
        ▼
  Train Random Forest Classifier
        │
        ├──► vpn_rf_model.pkl      (trained model)
        └──► label_encoder.pkl     (class label encoder)
```

Once complete, both `.pkl` files will appear in the project root and are required to run live detection.

---

## 🚀 Running Live Detection

### 🪟 Windows

**Step 1** — Open **PowerShell as Administrator**

**Step 2** — Activate your virtual environment:
```powershell
.\.venv\Scripts\Activate.ps1
```

**Step 3** — Run the classifier:
```powershell
python test.py
```

---

### 🐧 Linux

**Step 1** — Activate your virtual environment:
```bash
source .venv/bin/activate
```

**Step 2** — Run the classifier with root privileges:
```bash
sudo .venv/bin/python test.py
```

> **Why `sudo .venv/bin/python` and not just `sudo python3`?**
> Using `sudo python3` may point to the system Python instead of your virtual environment, causing `ModuleNotFoundError`. Always pass the full path to the venv interpreter when using `sudo`.

Alternatively, grant the Python binary raw socket access without running as full root:
```bash
sudo setcap cap_net_raw,cap_net_admin=eip .venv/bin/python3
# Then run normally:
python3 test.py
```

> Note: `setcap` must be re-applied each time you recreate the virtual environment.

---

### Expected terminal output

Once running, the classifier prints each classified flow in real time:

```
[FLOW] 192.168.1.5:443  → 8.8.8.8:53   | Prediction: VPN     | Confidence: 94.2%
[FLOW] 192.168.1.5:80   → 1.1.1.1:80   | Prediction: Non-VPN | Confidence: 88.7%
[FLOW] 10.0.0.2:51820   → 10.0.0.1:53  | Prediction: VPN     | Confidence: 97.1%
```

---

### 🔌 Configuring your network adapter

By default, `test.py` targets a specific adapter name. Update the `source` parameter to match your system's interface:

```python
streamer = NFStreamer(
    source="Your Interface Name Here",  # ← Update this
    promiscuous_mode=True,
    statistical_analysis=True,
    udps=FeatureReconstructor()
)
```

---

#### 🪟 Windows — How to find your adapter name

**Method 1 — PowerShell (recommended)**

```powershell
Get-NetAdapter | Select-Object Name, InterfaceDescription, Status
```

Example output:
```
Name       InterfaceDescription                              Status
----       --------------------                              ------
Wi-Fi      Realtek 8851BE Wireless LAN WiFi 6 PCI-E NIC     Up
Ethernet   Intel(R) Ethernet Connection (17) I219-LM         Disconnected
```

Use the value from the **`InterfaceDescription`** column — that is what `nfstream` expects on Windows.

---

**Method 2 — Python snippet**

```python
import psutil

adapters = psutil.net_if_stats()
for adapter, stats in adapters.items():
    status = "UP" if stats.isup else "DOWN"
    print(f"  [{status}] {adapter}")
```

---

**Method 3 — Device Manager (GUI)**

1. Press `Win + X` → select **Device Manager**
2. Expand **Network Adapters**
3. Right-click your adapter → **Properties**
4. Copy the name shown at the top of the Properties window

---

#### 🐧 Linux — How to find your interface name

**Method 1 — `ip` command (recommended)**

```bash
ip link show
```

Example output:
```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500    ← wired
3: wlan0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500   ← wireless
4: tun0: <POINTOPOINT,UP,LOWER_UP> mtu 1500            ← VPN tunnel
```

Use the interface name directly (e.g., `eth0`, `wlan0`, `tun0`).

---

**Method 2 — `ifconfig`**

```bash
ifconfig -a
```

> If `ifconfig` is not installed: `sudo apt install net-tools -y`

---

**Method 3 — Python snippet**

```python
import psutil

adapters = psutil.net_if_stats()
for adapter, stats in adapters.items():
    status = "UP" if stats.isup else "DOWN"
    print(f"  [{status}] {adapter}")
```

---

**Method 4 — List only active (UP) interfaces**

```bash
ip link show up
```

---

> **Which interface should I use?**
>
> | Traffic type | Windows example | Linux example |
> |---|---|---|
> | Wi-Fi | `Realtek 8851BE Wireless ...` | `wlan0` or `wlp3s0` |
> | Wired Ethernet | `Intel(R) Ethernet Connection ...` | `eth0` or `enp2s0` |
> | VPN tunnel | `TAP-Windows Adapter V9` | `tun0` or `wg0` |
> | Loopback (testing) | `Npcap Loopback Adapter` | `lo` |

---

## 📊 Feature Reference

The model is trained on **23 flow-level features** derived from packet timing and byte statistics:

| # | Feature | Description |
|---|---|---|
| 1 | `duration` | Total flow duration |
| 2 | `total_fiat` | Total forward inter-arrival time |
| 3 | `total_biat` | Total backward inter-arrival time |
| 4 | `min_fiat` | Minimum forward inter-arrival time |
| 5 | `min_biat` | Minimum backward inter-arrival time |
| 6 | `max_fiat` | Maximum forward inter-arrival time |
| 7 | `max_biat` | Maximum backward inter-arrival time |
| 8 | `mean_fiat` | Mean forward inter-arrival time |
| 9 | `mean_biat` | Mean backward inter-arrival time |
| 10 | `flowPktsPerSecond` | Packets per second for the flow |
| 11 | `flowBytesPerSecond` | Bytes per second for the flow |
| 12 | `min_flowiat` | Minimum flow inter-arrival time |
| 13 | `max_flowiat` | Maximum flow inter-arrival time |
| 14 | `mean_flowiat` | Mean flow inter-arrival time |
| 15 | `std_flowiat` | Standard deviation of flow inter-arrival time |
| 16 | `min_active` | Minimum active time |
| 17 | `mean_active` | Mean active time |
| 18 | `max_active` | Maximum active time |
| 19 | `std_active` | Standard deviation of active time |
| 20 | `min_idle` | Minimum idle time |
| 21 | `mean_idle` | Mean idle time |
| 22 | `max_idle` | Maximum idle time |
| 23 | `std_idle` | Standard deviation of idle time |

> If any feature column is missing from the CSV, `model.py` automatically fills it with zeros to maintain compatibility.

---

## 🔬 How It Works

```
Live Network Interface
        │
        ▼
   NFStreamer (nfstream)
   captures packet flows
        │
        ▼
  FeatureReconstructor (udps)
  rebuilds timing statistics
        │
        ▼
  23-Feature Vector assembled
        │
        ▼
  Random Forest Model
  (vpn_rf_model.pkl)
        │
        ├──► VPN
        └──► Non-VPN
```

**Noise filtering applied in `test.py`:**
- Broadcast and multicast addresses are skipped
- Very short flows (below a minimum packet threshold) are excluded
- Only flows with sufficient statistical data are classified

---

## 🛠️ Troubleshooting

### Common (both platforms)

| Problem | Cause | Fix |
|---|---|---|
| `ModuleNotFoundError` | Missing packages | Run `pip install -r requirements.txt` |
| `FileNotFoundError: vpn_rf_model.pkl` | Model not trained yet | Run `python model.py` first |
| `FileNotFoundError: label_encoder.pkl` | Encoder not saved yet | Run `python model.py` first |
| No flows captured | Wrong interface name | Find and update `source` in `test.py` |
| No flows captured | No network activity | Generate traffic (browse, ping, stream) |

---

### 🪟 Windows-specific

| Problem | Cause | Fix |
|---|---|---|
| Npcap/WinPcap error | Npcap not installed | Install from https://npcap.com |
| `PermissionError` during capture | Not running as admin | Open PowerShell as Administrator |
| Interface not found | Wrong `InterfaceDescription` string | Re-run `Get-NetAdapter` and copy exactly |

---

### 🐧 Linux-specific

| Problem | Cause | Fix |
|---|---|---|
| `PermissionError` / `Operation not permitted` | Missing root / capabilities | Use `sudo .venv/bin/python test.py` or apply `setcap` |
| `libpcap` not found | Missing system library | Run `sudo apt install libpcap-dev -y` |
| `sudo python3` ignores venv | System Python used instead of venv | Always use `sudo .venv/bin/python test.py` |
| Interface name changed after reboot | Predictable interface naming disabled | Verify with `ip link show` after each boot |
| `nfstream` build fails on install | Missing build tools | Run `sudo apt install build-essential python3-dev -y` |

---

## 📄 License

This project is provided **as-is** for educational and experimentation purposes only. It is not intended for use in production environments or for any activity that violates applicable laws or terms of service.

---

<p align="center">Made for network traffic research · Built with Python, NFStream & scikit-learn · Runs on Windows & Linux</p>
