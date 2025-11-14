# Use Alpine Linux for a lightweight base
FROM alpine:latest

# Install iptables (firewall), suricata (IDS), and tcpdump (troubleshooting)
RUN apk add --no-cache iptables suricata tcpdump

# Keep the container alive so we can 'exec' into it
CMD ["/bin/sh", "-c", "sleep 360000"]