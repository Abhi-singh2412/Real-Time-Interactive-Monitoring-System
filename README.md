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
 
  - Example You can extract data in the form of tables like this (Note: Remember to handle Null values when dealing with lat lon because it can cause problems ahead ) <img width="1920" height="950" alt="Screenshot (2036)" src="https://github.com/user-attachments/assets/6e0e89f0-b6da-4005-ad3e-92998f878c30" />
<img width="1920" height="949" alt="Screenshot (2031)" src="https://github.com/user-attachments/assets/9d9fd7cc-f091-40fd-af2b-bd46a29cc67c" />



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
<img width="1920" height="1080" alt="Screenshot (2046)" src="https://github.com/user-attachments/assets/88114ac1-ea49-4223-881f-cfe293155624" />

<img width="1920" height="1080" alt="Screenshot (2041)" src="https://github.com/user-attachments/assets/c9e57a4d-dc0b-4a9d-95a8-5b4c61681637" />



---

## Final Notes

Once your dashboard is live, continue to **optimize panels, queries, and alerts** to meet evolving security and monitoring requirements.

<img width="1065" height="1018" alt="Screenshot (2047)" src="https://github.com/user-attachments/assets/6167ce7f-15fc-4c2e-94d0-c6d4fe541e4c" />


---

**Thank you!**
