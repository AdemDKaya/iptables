

# Linux Firewall Lab: Step-by-Step Security Configuration with iptables

This project demonstrates the working principles of the Linux `iptables` firewall, including packet filtering, rule priority, protocol-based filtering, and the implementation of the widely adopted **Default Drop** security policy used in enterprise environments.

📺 **Video Walkthrough**

A complete step-by-step demonstration of this lab is available on YouTube:

**YouTube Video:** [▶ Watch on YouTube](https://www.youtube.com/watch?v=dQw4w9WgXcQ)

This video provides a practical walkthrough of the firewall configuration process, rule implementation, traffic filtering, and the Default Drop security policy demonstrated in this project.


## 💻 Lab Environment and Network Topology

The tests were conducted in a local laboratory environment consisting of two virtual machines connected to the same network segment:

* **Firewall / Target Machine:** Kali Linux (`192.168.1.104`)
* **Client / Testing Machine:** Windows 10 (`192.168.1.105`)

---

## 🛠️ Implementation Steps and Verification

### Step 1: Network Discovery and Connectivity Verification

First, the IP addresses of both machines were identified, and network connectivity was verified using the `ping` command. At this stage, no traffic restrictions were configured, allowing unrestricted communication between the systems.

### Step 2: SSH Service Management (TCP Port 22)

The SSH service was enabled on the Kali Linux machine, and a successful connection was established from Windows 10 using PuTTY and the terminal. To block SSH access, the following firewall rule was added:

```bash
sudo iptables -A INPUT -p tcp --dport 22 -j DROP
```

**Purpose of the Rule:**

* `INPUT`: Inspect incoming packets.
* `-p tcp`: Match the TCP protocol.
* `--dport 22`: Target SSH traffic on port 22.
* `-j DROP`: Silently discard matching packets.

**Result:**

Subsequent SSH connection attempts from Windows 10 failed with a **timeout error**, confirming that the rule was functioning correctly.

### Step 3: HTTP Service and Rule Priority Testing

An HTTP web server was started on Kali Linux using port 80, and successful access from Windows 10 was verified. To block HTTP traffic, the following rule was applied:

```bash
sudo iptables -A INPUT -p tcp --dport 80 -j DROP
```

After applying this rule, access to the website was interrupted.

To demonstrate the **top-down rule evaluation process** used by `iptables`, an allow rule was inserted above the existing drop rule (at position 2):

```bash
sudo iptables -I INPUT 2 -p tcp --dport 80 -j ACCEPT
```

**Result:**

Since packets matched the `ACCEPT` rule before reaching the lower `DROP` rule, HTTP access from Windows 10 was restored successfully.

### Step 4: Selective ICMP (Ping) Filtering

Instead of blocking all ICMP traffic, only incoming ping requests (`echo-request`) were filtered to preserve essential network diagnostic functionality.

```bash
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
```

**Result:**

Ping requests sent from Windows 10 to Kali Linux received no response, while other important ICMP functions remained operational, maintaining overall network health and troubleshooting capabilities.

### Step 5: Enterprise Security Approach – Default Drop Policy (Hardening)

In the final stage of the lab, the **Default Drop** security model was implemented. This approach follows the Zero Trust principle:

> Block everything by default and explicitly allow only trusted and necessary traffic.

The following commands were executed:

```bash
# 1. Remove all previously configured test rules
sudo iptables -F

# 2. Configure default policies
sudo iptables -P INPUT DROP
sudo iptables -P OUTPUT ACCEPT

# 3. Explicitly allow trusted traffic
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

#### Security Logic

* `INPUT DROP` closes all inbound access by default.
* `OUTPUT ACCEPT` allows the system to initiate outbound connections.
* `-i lo` permits localhost communication required for internal system operations.
* Ports **22 (SSH)**, **80 (HTTP)**, and **443 (HTTPS)** are explicitly allowed.
* All other ports, services, and protocols are automatically denied.

---

## 📌 Key Learning Outcomes

* Practical use of `iptables` chains (`INPUT`, `OUTPUT`) and parameters (`-A`, `-I`, `-p`, `--dport`, `-j`, `-P`).
* Understanding the impact of **rule ordering** on firewall behavior and packet processing.
* Implementing protocol-specific filtering instead of broad traffic blocking.
* Configuring an enterprise-style **Default Drop** security policy from scratch.
* Applying fundamental **Zero Trust** principles to Linux firewall hardening.

---

## 🔒 Conclusion

This lab provides hands-on experience with Linux firewall management using `iptables`. By progressively implementing filtering rules and ultimately enforcing a Default Drop policy, the exercise demonstrates how firewall configurations can be used to reduce the attack surface, control network access, and strengthen system security in real-world environments.
