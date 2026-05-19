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
