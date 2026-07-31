# Project 01 — Router & Switch Hardening

## Objective
Harden a Cisco router and switch against unauthorised access and Layer 2 attacks.
Disable Telnet, enforce SSH-only management, configure local AAA, enable port security,
DHCP snooping, and Dynamic ARP Inspection. Prove the hardening works using Wireshark
captures comparing plaintext Telnet credentials vs encrypted SSH traffic.

## Network Security Course Modules Applied
- **Module 4: Secure Device Access** — SSH-only remote management, securing console/VTY lines
- **Module 5: Assign Administrative Roles** — privilege levels and role-based CLI access
- **Module 6: Device Monitoring and Management** — Syslog and NTP for security logging
- **Module 7: Authentication, Authorization, and Accounting (AAA)** — local authentication
- **Module 14: Layer 2 Security Considerations** — port security, DHCP snooping, DAI

## Tools Used
- Cisco Packet Tracer (topology, CLI configuration)
- Cisco CSE-LABVM — Wireshark (Telnet vs SSH traffic capture proof)

## Topology
![Topology](screenshots/topology.png)

| Device | Interface | IP Address     | Subnet Mask     | Default Gateway |
|--------|-----------|----------------|-----------------|-----------------|
| Router | Gi0/0     | 192.168.1.1    | 255.255.255.0   | —               |
| Switch | VLAN 1    | 192.168.1.2    | 255.255.255.0   | 192.168.1.1     |
| PC0    | NIC       | 192.168.1.10   | 255.255.255.0   | 192.168.1.1     |
| PC1    | NIC       | 192.168.1.20   | 255.255.255.0   | 192.168.1.1     |
| PC2    | NIC       | 192.168.1.30   | 255.255.255.0   | 192.168.1.1     |

## Configuration Summary

### Router Hardening
- [x] hostname R1
- [x] Enable secret + `service password-encryption`
- [x] Custom banner MOTD (legal warning)
- [x] Console line: password + login + `exec-timeout 5 0`
- [x] VTY lines: `transport input ssh` + `login local` + `exec-timeout 5 0`
- [x] Local user: `username admin privilege 15 secret`
- [x] RSA key generation: `ip domain-name homelab.local` + `crypto key generate rsa` (1024-bit)
- [x] `ip ssh version 2`
- [x] Interface Gi0/0 configured + `no shutdown`
- [x] `no ip http server` + `no ip http secure-server` + `no cdp run`

Full config: [`configs/router-config.txt`](configs/router-config.txt)

### Switch Hardening
- [x] hostname SW1
- [x] Enable secret + `service password-encryption`
- [x] Banner MOTD
- [x] Console + VTY lines hardened (SSH only, login local)
- [x] Local user created
- [x] RSA key + `ip ssh version 2`
- [x] Interface VLAN 1 IP + `ip default-gateway`
- [x] Port security on Fa0/1–3: sticky MAC, max 1, violation shutdown
- [x] Unused ports (Fa0/4–24): access VLAN 99 + shutdown
- [x] `ip dhcp snooping` + `ip dhcp snooping vlan 1`
- [x] `ip dhcp snooping trust` on Gi0/1 (uplink to router)

Full config: [`configs/switch-config.txt`](configs/switch-config.txt)

## Verification

| Test | Expected Result | Screenshot |
|------|-----------------|------------|
| `show ip ssh` | SSH v2 enabled | ![](screenshots/ssh-verification.png) |
| Attempt Telnet from PC | Connection refused | ![](screenshots/telnet-blocked.png) |
| Wireshark: Telnet capture | Credentials visible in plaintext | ![](screenshots/wireshark-telnet.png) |
| Wireshark: SSH capture | Encrypted payload — no readable credentials | ![](screenshots/wireshark-ssh.png) |
| `show port-security` | Sticky MAC learned, 0 violations | ![](screenshots/port-security.png) |
| Plug unauthorised device | Port goes err-disabled | ![](screenshots/port-violation.png) |
| `show ip dhcp snooping` | Snooping enabled on VLAN 1 | ![](screenshots/dhcp-snooping.png) |

## Why Telnet vs SSH Matters
The Wireshark captures below demonstrate the core reason this hardening config matters.
Telnet sends all data — including login credentials — in plaintext. SSH encrypts
everything. Any attacker with access to the network could capture Telnet traffic and
read admin passwords directly from the packet payload.

| | Telnet | SSH |
|---|---|---|
| Credentials on wire | Plaintext — readable | Encrypted — unreadable |
| Safe for production | Never | Yes |
| Config applied | `transport input none` (disabled) | `transport input ssh` + RSA key |

## Lessons Learned / Production Notes
- Packet Tracer does not fully simulate DAI logging — noted as a simulation limitation,
  not a config gap. In production, DAI logging would confirm dropped ARP spoofing attempts.
- Local AAA is a starting point only. In a production environment, centralised AAA via
  RADIUS or TACACS+ should be used — this is covered in Project 07 of this portfolio.
- BPDU Guard was configured but Packet Tracer's simulation of STP attack prevention is
  limited. Verified conceptually via config rather than live attack simulation.

## References
- Cisco NetAcad — Network Security Course (Modules 4, 5, 6, 7, 14)
- Cisco IOS Security Configuration Guide
- [Cisco Packet Tracer Download](https://www.netacad.com/resources/lab-downloads)
- [CSE-LABVM Download](https://www.netacad.com/resources/lab-downloads)
