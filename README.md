# Real-Time Interactive Monitoring System

A live observability dashboard powered by Splunk Enterprise. This solution integrates endpoint telemetry via the Splunk Universal Forwarder to ingest granular security events (such as Sysmon logs) for real-time monitoring and analysis.

---

## Prerequisites & Installation

### System Requirements

- **Monitored Endpoint:**  
  - Windows 10, Windows Server 2016, or newer (admin privileges required).
- **Central Splunk Server:**  
  - Linux or Windows machine running Splunk Enterprise (or Splunk Free).
  - Sufficient RAM/disk for log indexing.
- **Network:**  
  - TCP Port **9997** open from Endpoint to Server.

---

## Setup Steps

### 1. Central Splunk Server

- **Install Splunk Enterprise:**  
  Download and install on your central server.
- **Enable Receiver Port:**  
  - Navigate: `Settings → Data → Forwarding and Receiving`.  
  - Add receiver on port **9997**.
- **Create Index:**  
  - `Settings → Data → Indexes → New Index`  
  - Name: `sysmon_index`
- **Install Add-on (Recommended):**  
  - Install the **Splunk Add-on for Microsoft Sysmon** on the Search Head for field extractions.

---

### 2. Endpoint Setup (Sysmon & Universal Forwarder)

- **Install Sysmon:**  
  - Download from [Sysinternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon).
  - Acquire a robust config file (`config.xml`).
  - Install via elevated prompt:  
    ```
    Sysmon.exe -i <path-to-config.xml> -accepteula
    ```
- **Install Splunk Universal Forwarder (UF):**  
  - Download & install UF on the endpoint.
  - Point UF to Central Server (Hostname/IP, Port 9997).
- **Configure UF Inputs:**  
  Edit `inputs.conf` in the UF configuration directory. Add:
    ```
    [WinEventLog://Microsoft-Windows-Sysmon/Operational]
    disabled = false
    index = sysmon_index
    sourcetype = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
    renderXml = 1
    ```
  - `renderXml = 1` ensures all event fields are forwarded.
- **Restart UF Service:**
    ```
    net stop SplunkForwarder
    net start SplunkForwarder
    ```

---

## Data Validation

- **Basic Search (on Central Server):**
    ```
    index=sysmon_index | timechart count by host limit=1
    ```
- **Check Sysmon Data Parsing:**
    ```
    index=sysmon_index | head 10
    ```
  - Ensure EventCode and key fields (e.g., `Image`, `CommandLine`, `DestinationIp`) are extracted.

---

## Defining Key Performance Indicators (KPIs)

| Security Question                                | Sysmon EventCode | Example Panel/Query                |
|--------------------------------------------------|------------------|------------------------------------|
| What new processes are running?                  | 1 (Process Create) | Top 10 New Processes               |
| Is a process connecting externally?              | 3 (Network Connect) | Top Dest. IPs by Process           |
| Are suspicious files being created/modified?     | 11 (File Create)   | Top File Modifications by Path     |

---

## Dashboard Panels & SPL Queries

### **Dashboard Name:** Endpoint Activity Monitor  
*For Tier 1 Security Analysts*

---

#### **Panel 1: Top 10 Executed Processes**

**Purpose:** Spot anomalies/new executables in last 24h  
**Visualization:** Bar Chart  
**SPL:**
```
index=sysmon_index EventCode=1 
| stats count by Image 
| sort - count 
| head 10
```

---

#### **Panel 2: Network Connection Summary**

**Purpose:** Internal vs. External network connections  
**Visualization:** Pie Chart  
**SPL:**
```
index=sysmon_index EventCode=3 
| eval Connection_Type=if(match(DestinationIp, "192\\.168\\.|10\\.|172\\.(1[6-9]|2[0-9]|3[0-1])"), "Internal", "External")
| stats count by Connection_Type
```

---

#### **Panel 3: PowerShell Activity Over Time**

**Purpose:** Track PowerShell executions (potential IOA)  
**Visualization:** Time Chart  
**SPL:**
```
index=sysmon_index EventCode=1 Image="*powershell.exe" 
| timechart count as "PowerShell Executions"
```

---

*You can create additional dashboard panels to suit specific monitoring needs.*

---

## Final Notes

Once your dashboard is live, continue to **optimize panels, queries, and alerts** to meet evolving security and monitoring requirements.

---

**Thank you!**
