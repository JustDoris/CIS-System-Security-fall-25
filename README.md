# CIS-System-Security-fall-25
securing a containerized office network by implementing and validating network segmentation, firewall rules, and intrusion detection.
-------------------------------------------------------------------------------------------------------------------------------------
# Docker-Based Network Security Lab: Segmentation & IDS
This repository contains all code and documentation for a student project to design, implement, and validate a secure, containerized small-office network. This lab uses Docker and docker-compose to simulate network segmentation and deploys a containerized firewall (iptables) and IDS (Suricata) to defend a vulnerable web application from a simulated attacker.

This project was successfully pivoted from a VirtualBox/pfSense model to a 100% Docker-based environment to improve speed, reduce resource consumption, and increase reproducibility.

Project Team
Nathaniel Betancourt: Lead Engineer / Blue Team

Esai Uribe: Security Analyst / Red Team

Asia Williams: Documentation & Reporting / Blue Team Analyst

Technology Stack
Virtualization: Docker Desktop

Orchestration: docker-compose

Firewall/Router: Alpine Linux container with iptables

IDS: Suricata (running in the router container)

Attacker Node: kalilinux/kali-rolling container

Target Node: bkimminich/juice-shop (OWASP Juice Shop)

Staff Node: dorowu/ubuntu-desktop-lxde-vnc (Browser-based GUI)
