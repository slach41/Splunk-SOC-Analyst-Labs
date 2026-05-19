# Splunk-SOC-Analyst-Labs
🛡️ Hands-on SOC Analyst labs focusing on SIEM (Splunk) log analysis, alert triage, and security incident investigation.
# 🛡️ Splunk SOC Analyst Labs

Welcome to my security operations repository! This space is dedicated to documenting my hands-on experience, practical training, and cybersecurity labs. Here, I simulate and analyze real-world security incidents using Splunk SIEM, Windows/Linux logs, and network traffic.

---

## 📂 Lab 1: Windows Log Analysis & Event Correlation in Splunk

### 🔍 Incident Overview
- **Objective:** Analyze Windows Security Logs inside Splunk SIEM to detect suspicious authentication behavior and unauthorized process creations.
- **Platform/Tools:** Splunk Enterprise, Windows Security Event Logs.
- **Key Event IDs Investigated:** - `4624` (Successful Logon) - To track active sessions.
  - `4625` (Failed Logon Attempt) - To identify potential brute-force or unauthorized access attempts.
  - `4672` (Special Privileges Assigned to New Logon) - To detect administrative/privilege escalation activities.
  - `4688` (A New Process Has Been Created) - To audit command-line executions and system activity post-logon.

---

### 🛠️ Investigation & Log Correlation Steps

By timeline-correlating the events captured in the SIEM platform, a clear chain of suspicious activities can be mapped out:

1. **External Administrative Logon:**
   A successful login (`EventID 4624`) was recorded for the user `admin` originating from an external IP address (`185.220.101.4`). Immediate elevation followed with `EventID 4672` (Privileges assigned).

2. **Suspicious Process Spawning:**
   Right after the admin login, `EventID 4688` captured the creation of a `powershell.exe` process. In a production environment, an external login immediately spawning PowerShell is a critical indicator of compromise (IoC) that warrants immediate triage.

3. **Internal Account Activity:**
   Simultaneously, local account monitoring showed a failed login attempt (`EventID 4625`) for user `john` from an internal IP (`10.0.0.55`), followed immediately by a successful login (`EventID 4624`) and the execution of `cmd.exe` (`EventID 4688`).

---

### 📊 Proof of Work (SIEM Dashboard)

*Below is the actual Splunk search instance analyzing the correlated `windows_security.log` events:*

<img width="1600" height="748" alt="1" src="https://github.com/user-attachments/assets/5c3f9d78-2ca3-4a23-859e-cfb61c908575" />


---

### 🎯 Analyst Conclusion & Retrospective
- **Verdict:** **Suspicious Activity / Potential Lateral Movement**. The correlation of successful external administrative logins followed by sudden command-line execution (`powershell.exe` and `cmd.exe`) indicates highly anomalous behavior.
- **Next Steps (Triage Workflow):** - Verify if the external IP (`185.220.101.4`) belongs to an authorized VPN/employee or a known malicious network (e.g., Tor exit node).
  - Inspect the specific command-line arguments passed to `powershell.exe` to check for malicious scripts or encoded commands.
  - Isolate the host system if malicious intent is confirmed to prevent further lateral movement.



---

## 📂 Lab 2: Endpoint Security Auditing Via Windows Event Viewer

### 🔍 Incident Overview
- **Objective:** Analyze raw Windows Security Event Logs directly from the endpoint to trace authentication anomalies and identify host-level Indicators of Compromise (IoCs).
- **Platform/Tools:** Windows Event Viewer (`eventvwr.msc`), Windows Security Log Auditing.
- **Key Event IDs Investigated:**
  - `4625` (Audit Failure - Failed Logon Attempt)
  - `4624` (Audit Success - Successful Logon)
  - `4688` (Audit Success - A New Process Has Been Created)

---

### 🛠️ Host-Level Log Triage Workflow

When investigating potential host compromises directly on the endpoint, analyzing the raw properties of Windows Security Events is crucial for determining the timeline of an attack:

1. **Analyzing Authentication Failures (`Event ID 4625`):**
   Inspected failed logon logs to check the subject security ID and logon type. High-frequency generation of Event 4625 helps isolate brute-force or credential-stuffing attempts targeting specific machine accounts.

2. **Verifying Logon Success (`Event ID 4624`):**
   Correlated the exact timestamp of failed attempts with subsequent successful logons. Confirming an `Audit Success` right after multiple failures indicates a potentially compromised account.

3. **Auditing Post-Exploitation Activity (`Event ID 4688`):**
   Tracked new process creations immediately following successful sessions. Auditing the target system logs for spawned processes (such as native utilities like `Splunkd` auditing infrastructure or potential living-off-the-land binaries) provides insight into what the execution context did post-auth.

---

### 📊 Proof of Work (Event Properties)

*Below are the raw event properties captured directly from the Windows Security Log during auditing practice:*

#### ❌ Event 4625: Audit Failure - An account failed to log on
<img width="922" height="649" alt="2" src="https://github.com/user-attachments/assets/6d653e41-d3a8-4d27-85b1-93420f6e620d" />


####  Event 4624: Audit Success - An account was successfully logged on
<img width="926" height="645" alt="3" src="https://github.com/user-attachments/assets/2ec1ff42-3bc1-4246-a5b9-b207eca840c7" />


#### ⚙️ Event 4688: Audit Success - A new process has been created
<img width="924" height="646" alt="4" src="https://github.com/user-attachments/assets/df14a34f-86cf-4df4-b76e-20dbbbf7c344" />


---

### 🎯 Analyst Takeaway & Core Skillset
- **Host vs. Network Analysis:** Mastering raw Windows event properties allows a SOC analyst to perform deep-dive forensics directly on endpoints when remote SIEM collectors are blind or when validating alerting logic validity.
- **Correlation:** Knowing how to pivot from a raw Windows Event log to a Splunk SPL query is a foundational capability for effective Tier 1 alert triage.
