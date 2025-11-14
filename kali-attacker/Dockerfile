# Start from the official Kali image
FROM kalilinux/kali-rolling:latest

# Update and install the 'headless' bundle of tools
# This includes nmap, scapy, metasploit, etc.
RUN apt-get update && apt-get install -y kali-linux-headless

# Keep the container alive
CMD ["/bin/bash"]