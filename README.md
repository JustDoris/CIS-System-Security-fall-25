# CIS-System-Security-fall-25
securing a containerized office network by implementing and validating network segmentation, firewall rules, and intrusion detection.
-------------------------------------------------------------------------------------------------------------------------------------

# Docker-Based Network Security Lab: Segmentation & IDS

This repository contains all code and documentation for a student project to design, implement, and validate a secure, containerized small-office network. This lab uses Docker and `docker-compose` to simulate network segmentation and deploys a containerized firewall (`iptables`) and IDS (Suricata) to defend a vulnerable web application from a simulated attacker.

## Project Team

  * **Nathaniel Betancourt:** Lead Engineer / Blue Team
  * **Esai Uribe:** Security Analyst / Red Team
  * **Asia Williams:** Documentation & Reporting / Blue Team Analyst

## Technology Stack

  * **Virtualization:** Docker Desktop
  * **Orchestration:** `docker-compose`
  * **Firewall/Router:** Alpine Linux container with `iptables`
  * **IDS:** Suricata (running in the router container)
  * **Attacker Node:** `kalilinux/kali-rolling` container
  * **Target Node:** `bkimminich/juice-shop` (OWASP Juice Shop)
  * **Staff Node:** `dorowu/ubuntu-desktop-lxde-vnc` (Browser-based GUI)

## Lab Architecture

This project uses Docker's custom bridge networks to create three fully isolated logical networks, simulating VLANs. All traffic *must* pass through the central `router-ids` container, which acts as the firewall and intrusion detection system.

```
                      +-----------------------+

| router-ids |
| (Alpine Linux) |
| - iptables (Firewall) |
| - Suricata (IDS) |
                      +--+--------+--------+--+

| | |
(eth0 - 192.168.10.1) | | | (eth2 - 192.168.30.1)
  +----------------------+ | +-----------------------+

| | |
(eth1 - 192.168.20.1) |

| |
+--+-----------------------------+--+-----------------------------+--+

| | |
| [Network: staff_net] | [Network: servers_net] | [Network: iot_net]
| (192.168.10.0/24) | (192.168.20.0/24) | (192.168.30.0/24)
| | |
+---------------+               +---------------+              +---------------+

| staff-pc | | web-server | | attacker |
| (192.168.10.10) | | (Juice Shop) | | (Kali Linux) |
| (VNC Desktop) | | (192.168.20.10) | | (192.168.30.10) |
+---------------+               +---------------+              +---------------+
```

##  Quickstart

This entire lab can be built and deployed with a single command.

**Prerequisites:**

  * ([https://www.docker.com/products/docker-desktop/](https://www.docker.com/products/docker-desktop/)) (for Windows, Mac, or Linux)

**Installation:**

1.  Clone this repository:
    ```bash
    git clone <your-repo-url>
    ```
2.  Navigate to the project directory:
    ```bash
    cd docker-security-lab
    ```
3.  Build and run the containers in detached mode:
    ```bash
    docker-compose up -d --build
    ```
    *(Note: The first build will take several minutes to download the Kali image and install its tools.)*

### Accessing the Lab

  * **Attacker (Red Team):** `docker exec -it attacker bash`
  * **Firewall/IDS (Blue Team):** `docker exec -it router-ids sh`
  * **Staff PC (Victim):** Open a web browser on your host machine and go to `http://localhost:6080`

-----

##  The 18-Day Project Sprint

### Sprint 1: Documentation & Baseline (Days 1-5)

**Goal:** Document the lab as-built and establish a baseline of isolation.

  * **Task 1 (GitHub Setup):** [Lead: **Nathaniel**]
      * Create the GitHub repository, push initial files (`docker-compose.yml`, `router-ids/Dockerfile`, `kali-attacker/Dockerfile`).
      * Set up GitHub Issues and Milestones for all tasks in this plan.
  * **Task 2 (Project README):** [Lead: **Asia**]
      * Write the new `README.md` file (this file\!) based on the Docker architecture, replacing all old VirtualBox/pfSense content.
  * **Task 3 (GitHub Wiki Creation):** [Lead: **Esai**]
      * Enable and structure the GitHub Wiki with five key pages: `Home`, `Network Architecture`, `Firewall & IDS Configuration`, `Attack Plan`, and `Incident Log & Case Studies`.
  * **Task 4 (Baseline Test & Diagram):** [Lead: **Esai**, **Nathaniel**]
      * `docker exec` into the `attacker` container and confirm pings to `192.168.20.10` (server) and `192.168.10.10` (staff) **fail**.
      * Create the final `Network Architecture` diagram (like the one above) and post it to the Wiki.
  * **Task 5 (Document Baseline):** [Lead: **Asia**]
      * Capture screenshots of the failed ping tests from Task 4.
      * Upload screenshots to the `Network Architecture` Wiki page as "Proof of Initial Segmentation."

### Sprint 2: Implement, Attack & Defend (Days 6-12)

**Goal:** Build the firewall rules, configure the IDS, and execute simulated attacks to validate the defenses.

  * **Task 6 (Firewall Implementation - Blue Team):** [Lead: **Nathaniel**]
      * `exec` into the `router-ids` container.
      * Use `iptables` to write and test the firewall policy.
      * **Rule 1:** Allow `staff_net` (192.168.10.0/24) to access `servers_net` (192.168.20.0/24).
      * **Rule 2:** Block `iot_net` (192.168.30.0/24) from accessing *any* other internal network.
      * **Deliverable:** Post the final, working `iptables` script to the `Firewall & IDS Configuration` Wiki page.
  * **Task 7 (Define Attack Plan - Red Team):** [Lead: **Esai**]
      * Research and document 2-3 specific attack scenarios on the `Attack Plan` Wiki page.
      * **Scenario 1 (Recon):** `nmap -sS 192.168.20.10` to scan the server.[1, 2, 3]
      * **Scenario 2 (Exploitation):** Use browser on Staff PC (`http://localhost:6080`) to find a login page on the Juice Shop and attempt a basic SQL injection.[4]
      * **Scenario 3 (Repeat Recon):** Re-run `nmap` scan from `attacker` to prove it is now blocked by the firewall.
  * **Task 8 (Configure IDS - Blue Team):** [Lead: **Nathaniel**]
      * `exec` into `router-ids`.
      * Run `suricata-update` to download rules.
      * Run Suricata to listen on all internal interfaces (`eth0`, `eth1`, `eth2`).
      * Open the live alert log: `tail -f /var/log/suricata/fast.log`.
  * **Task 9 (Coordinated Attack & Defense):** [Lead: **All**]
      * **Red Team (Esai):** `exec` into `attacker` and execute attack scenarios from Task 7.
      * **Blue Team (Nathaniel):** `exec` into `router-ids` and watch the `tail -f` Suricata log for alerts.
      * **Analyst (Asia):** Screen-share and capture screenshots of:
        1.  Esai's terminal (showing the attack).
        2.  Nathaniel's terminal (showing the real-time `iptables` logs or Suricata alerts).
        3.  The VNC Staff PC (`localhost:6080`) showing the successful SQL injection.
  * **Task 10 (Log Evidence):** [Lead: **Asia**]
      * Upload all screenshots from Task 9 to the `Incident Log` Wiki page, creating one entry per attack.

### Sprint 3: Final Report & Presentation (Days 13-18)

**Goal:** Synthesize all documentation and evidence into a professional final presentation.

  * **Task 11 (Write Case Studies):** [Lead: **Asia**]
      * Convert the `Incident Log` entries into 2-3 formal "Case Studies" on the Wiki.
      * **Template:** *Attack Scenario*, *Red Team Action* (Esai's command), *Blue Team Defense* (Nathaniel's `iptables` rule), *Evidence* (her screenshots), *Conclusion*.
  * **Task 12 (Build Presentation Slides):** [Lead: **Esai**]
      * Build the final PowerPoint/Google Slides deck using all the content from the GitHub Wiki.
  * **Task 13 (Finalize Repository):** [Lead: **Nathaniel**]
      * Clean up this `README.md` file and ensure the "How to Run" steps are perfect.
  * **Task 14 (Rehearse Final Demo):** [Lead: **All**]
      * Practice the final presentation and live demo.
      * **Flow:** Esai (Intro/Architecture), Asia (Case Studies/Evidence), Nathaniel (Live Demo).
  * **Task 15 (Day 18):**
      * Deliver final presentation to the professor.

## Final Deliverables

1.  A fully functional, containerized network lab defined in `docker-compose.yml`.
2.  A complete GitHub repository containing all code and documentation.
3.  A populated GitHub Wiki detailing the project's architecture, firewall rules, attack plans, and incident logs.
4.  A final presentation slide deck.
5.  A live, 10-minute demonstration of the Red Team attacking and the Blue Team's IDS/firewall detecting and blocking the attack in real-time.
