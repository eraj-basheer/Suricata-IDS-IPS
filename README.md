# Suricata IDS/IPS

## Overview

This lab involved installing and configuring Suricata as an IDS/IPS on pfSense.

The Suricata configuration was used to monitor traffic on the SECURITY interface and detect specific network activity using custom rules.

The lab builds on the configuration from the (pfSense Lab)[https://github.com/eraj-basheer/pfSense-Firewall] 

## Objectives

* Verify the existing pfSense environment
* Install Suricata
* Configure Suricata on the REDLAN interface
* Disable hardware offloading required for Suricata
* Create custom Suricata rules
* Generate test traffic from Kali
* Review Suricata alerts
* Disable Suricata after testing

## Lab Environment

| Component      | Role                           |
| -------------- | ------------------------------ |
| pfSense        | Firewall and IDS/IPS           |
| Kali Linux     | Test/attack machine            |
| Metasploitable | Target server                  |
| Debian         | pfSense management workstation |
| Hyper-V        | Virtualisation platform        |

## Network Configuration

| Network | Subnet         |
| ------- | -------------- |
| BLUE    | 192.168.1.0/24 |
| PURPLE  | 10.30.0.0/24   |
| RED     | 192.168.2.0/24 |
| WAN     | 192.168.0.0/24 |

## Task 1 – Verify Environment

Before installing Suricata, I verified the DHCP scopes, VM network connections and pfSense interfaces from the previous firewall lab.

### pfSense Interfaces

| Interface   | Operating System Interface | IP Address     |
| ----------- | -------------------------- | -------------- |
| BLUELAN     | hn0                        | 192.168.1.1/24 |
| PURPLE_DMZ  | hn1                        | 10.30.0.1/24   |
| REDLAN      | hn2                        | 192.168.2.1/24 |
| INTERNETWAN | hn3                        | DHCP           |

### Evidence

![pfSense Interfaces](screenshots/02-pfsense-interfaces.png)

**Result:** The pfSense interfaces were verified before continuing.

---

# Task 2 – Install Suricata

Suricata was installed through the pfSense Package Manager.

### Procedure

1. Opened pfSense WebConfigurator.
2. Selected **System → Package Manager**.
3. Opened **Available Packages**.
4. Searched for **Suricata**.
5. Installed the Suricata package.
6. Confirmed successful installation.

### Evidence

![Suricata Installation](screenshots/03-suricata-installed.png)

**Result:** Suricata was successfully installed and appeared under the installed packages.

---

# Task 3 – Configure Suricata

## 3.1 Disable Hardware Offloading

The following hardware offloading options were disabled:

* Hardware checksum offloading
* Hardware TCP segmentation offloading
* Hardware large receive offloading

### Evidence

![Hardware Offloading](screenshots/04-hardware-offloading.png)

The pfSense system was then rebooted.

---

## 3.2 Configure REDLAN Monitoring

Suricata was configured to monitor the **REDLAN (hn2)** interface.

### Configuration

```text
Interface: REDLAN
Operating system interface: hn2
Status: Enabled/Running
```

### Evidence

![Suricata REDLAN](screenshots/05-redlan-monitoring.png)

**Result:** Suricata was successfully enabled and running on REDLAN.

The lab specifies REDLAN as the interface to monitor because the scenario is designed to observe traffic sent from Kali toward Metasploitable.

---

# 3.3 Custom Suricata Rules

Two custom rules were created.

### Rule 1 – ICMP

```text
alert icmp any any -> [192.168.1.0/24,10.30.0.0/24] any (msg:"PING connection attempt to BLUE/PURPLE"; sid:2000001; rev:1;)
```

### Rule 2 – Telnet

```text
alert tcp any any -> [192.168.1.0/24,10.30.0.0/24] 23 (msg:"TELNET connection attempt to BLUE/PURPLE"; sid:2000002; rev:1;)
```

The ICMP rule detects ping traffic directed toward the BLUE or PURPLE networks.

The Telnet rule detects TCP traffic destined for port 23 on the BLUE or PURPLE networks.

### Evidence

![Custom Suricata Rules](screenshots/06-custom-rules.png)

---

# Task 4 – Testing Suricata

## 4.1 Ping Test

The Kali VM was used to generate ICMP traffic toward the Metasploitable VM.

Command:

```bash
ping <METASPLOITABLE-IP>
```

### Evidence

![Kali Ping Test](screenshots/07-kali-ping.png)

### Expected Result

Suricata should generate an alert matching the custom ICMP rule.

---

## 4.2 Telnet Test

A Telnet connection was then attempted from Kali to Metasploitable.

Command:

```bash
telnet <METASPLOITABLE-IP>
```

### Evidence

![Kali Telnet Test](screenshots/08-kali-telnet.png)

### Expected Result

Suricata should generate an alert matching the custom Telnet rule.

The lab specifically instructs the user to perform both a ping and Telnet connection from Kali to Metasploitable and then review the Suricata Alerts tab.

---

# 4.3 Suricata Alerts

The Suricata Alerts page was refreshed after generating the test traffic.

### Evidence

![Suricata Alerts](screenshots/09-suricata-alerts.png)

### Results

| Test   | Rule                    | Alert Generated |
| ------ | ----------------------- | --------------- |
| Ping   | ICMP custom rule        | [YES/NO]        |
| Telnet | TCP port 23 custom rule | [YES/NO]        |

The alerts demonstrate that Suricata detected traffic matching the configured signatures.

---

# 4.4 Telnet Intrusion Detection

A separate Telnet alert was captured as evidence for the practice activity.

![Telnet Alert](screenshots/10-telnet-alert.png)

**Result:** The Telnet connection attempt from Kali to Metasploitable generated the expected custom Suricata alert.

---

# Task 5 – Disable Suricata

After completing the testing, Suricata was disabled on the REDLAN interface.

### Procedure

1. Opened **Services → Suricata**.
2. Selected the REDLAN interface.
3. Opened the interface configuration.
4. Disabled the **Enable** option.
5. Saved the configuration.
6. Applied the changes.

### Evidence

![Suricata Disabled](screenshots/11-suricata-disabled.png)

The lab recommends disabling Suricata after completing the activities to reduce resource usage on the pfSense VM.

---

# Key Learnings

* Learned how to install Suricata on pfSense.
* Learned how an IDS/IPS can monitor network traffic.
* Learned how to configure Suricata to monitor a specific interface.
* Learned how custom signatures can identify specific traffic.
* Learned how ICMP and Telnet traffic can trigger alerts.
* Learned how to review Suricata alerts for security monitoring.
* Learned the difference between monitoring traffic and actively blocking traffic.

## Conclusion

This lab demonstrated how Suricata can be integrated with pfSense to provide IDS/IPS capabilities.

Custom ICMP and Telnet rules were created and tested using traffic generated from Kali toward Metasploitable. The resulting alerts provided evidence that Suricata was able to detect traffic matching the configured rules.

The completed configuration demonstrated how a firewall and IDS/IPS can work together to provide network protection and visibility.
