# safe-squidlabs

#ASSENMENT :1

#!/bin/bash

#  top 10 most used applications
show_top_apps() {
    echo "Top 10 Most Used Applications:"
    ps -eo pid,comm,%cpu,%mem --sort=-%cpu | head -n 11 | awk '{ printf "%-10s %-20s %-6s %-6s\n", $1, $2, $3, $4 }'
    echo ""
}

#  network monitoring details
show_network() {
    echo "Network Monitoring:"
    echo "Number of concurrent connections:"
    ss -tn | grep ESTAB | wc -l

    echo "Packet drops:"
    netstat -i | awk 'NR>2 { print $1, $6, $7 }' | awk '{if ($2 > 0 || $3 > 0) print $1 " Rx drops: " $2 " Tx drops: " $3}'

    echo "Network usage (MB in and out):"
    vnstat --oneline | awk -F\; '{print "In: " $10 " MB, Out: " $11 " MB"}'
    echo ""
}

# disk usage
show_disk_usage() {
    echo "Disk Usage:"
    df -h | awk '{print $1, $5}' | grep -v 'Use%'
    echo ""

    echo "Partitions using more than 80% of space:"
    df -h | awk '$5+0 > 80 {print $1, $5}'
    echo ""
}

# system load
show_system_load() {
    echo "System Load:"
    uptime | awk -F'[a-z]:' '{ print $2 }'
    
    echo "CPU Usage:"
    mpstat | awk 'NR==4 {print "User: " $3 "%, System: " $5 "%, Idle: " $12 "%"}'
    echo ""
}

#  memory usage
show_memory_usage() {
    echo "Memory Usage:"
    free -h | awk 'NR==2 {print "Total: " $2 ", Used: " $3 ", Free: " $4}'
    echo "Swap Memory Usage:"
    free -h | awk 'NR==3 {print "Total: " $2 ", Used: " $3 ", Free: " $4}'
    echo ""
}

#  process monitoring
show_process_monitoring() {
    echo "Process Monitoring:"
    echo "Number of active processes:"
    ps -e | wc -l

    echo "Top 5 processes by CPU usage:"
    ps -eo pid,comm,%cpu --sort=-%cpu | head -n 6 | awk '{ printf "%-10s %-20s %-6s\n", $1, $2, $3 }'

    echo "Top 5 processes by memory usage:"
    ps -eo pid,comm,%mem --sort=-%mem | head -n 6 | awk '{ printf "%-10s %-20s %-6s\n", $1, $2, $3 }'
    echo ""
}

# service status
show_service_status() {
    echo "Service Monitoring:"
    for service in sshd nginx apache2 iptables; do
        echo -n "$service: "
        systemctl is-active $service 2>/dev/null || echo "Service not found"
    done
    echo ""
}

# refresh the dashboard
refresh_dashboard() {
    clear
    show_top_apps
    show_network
    show_disk_usage
    show_system_load
    show_memory_usage
    show_process_monitoring
    show_service_status
}

# argument processing
while getopts ":cpu memory network disk load processes services" opt; do
    case $opt in
        cpu)
            show_system_load
            ;;
        memory)
            show_memory_usage
            ;;
        network)
            show_network
            ;;
        disk)
            show_disk_usage
            ;;
        load)
            show_system_load
            ;;
        processes)
            show_process_monitoring
            ;;
        services)
            show_service_status
            ;;
        \?)
            echo "Invalid option: -$OPTARG" >&2
            exit 1
            ;;
        :)
            echo "Option -$OPTARG requires an argument." >&2
            exit 1
            ;;
    esac
    exit 0
done

# If no options are provided, show the full dashboard
refresh_dashboard




ASSENMENT-2 


#!/bin/bash

# Configuration
CONFIG_FILE="/etc/security-audit-config.conf"

# Utility functions
log() {
    echo "[INFO] $1"
}

error() {
    echo "[ERROR] $1" >&2
}

# User and Group Audits
audit_users_groups() {
    log "Listing all users and groups..."
    cat /etc/passwd
    cat /etc/group
    log "Checking for UID 0 users..."
    awk -F: '$3 == 0 { print $1 }' /etc/passwd
    log "Checking for users without passwords..."
    awk -F: '($2 == "" || $2 == "x") { print $1 }' /etc/shadow
}

# File and Directory Permissions
audit_permissions() {
    log "Scanning for world-writable files and directories..."
    find / -xdev -type f -perm -0002 -exec ls -l {} \;
    log "Checking .ssh directory permissions..."
    find /home -name ".ssh" -exec ls -ld {} \;
    log "Reporting SUID and SGID files..."
    find / -type f \( -perm -4000 -o -perm -2000 \) -exec ls -l {} \;
}

# Service Audits
audit_services() {
    log "Listing all running services..."
    systemctl list-units --type=service --state=running
    log "Checking critical services..."
    systemctl status sshd
    systemctl status iptables
    log "Checking for services on non-standard ports..."
    netstat -tuln
}

# Firewall and Network Security
audit_firewall_network() {
    log "Checking firewall status..."
    iptables -L -n
    log "Checking open ports..."
    ss -tuln
    log "Verifying IP forwarding..."
    sysctl net.ipv4.ip_forward
}

# IP and Network Configuration Checks
check_ip_addresses() {
    log "Identifying IP addresses..."
    ip -4 addr show
    ip -6 addr show
    log "Checking if IPs are public or private..."
    # Placeholder for IP classification logic
}

# Security Updates and Patching
check_updates() {
    log "Checking for available security updates..."
    apt-get update && apt-get upgrade -s | grep -i security
    log "Ensuring automatic updates are configured..."
    dpkg-query -W unattended-upgrades
}

# Log Monitoring
monitor_logs() {
    log "Checking for suspicious log entries..."
    grep -i "sshd.*failed" /var/log/auth.log
}

# Server Hardening Steps
hardening_steps() {
    log "Applying SSH configuration..."
    sed -i '/^PermitRootLogin/s/yes/no/' /etc/ssh/sshd_config
    log "Disabling IPv6 if required..."
    # Placeholder for IPv6 disabling logic
    log "Securing the bootloader..."
    # Placeholder for GRUB password setting logic
    log "Configuring iptables rules..."
    # Placeholder for iptables configuration logic
    log "Configuring automatic updates..."
    # Placeholder for unattended-upgrades configuration logic
}

# Custom Security Checks
custom_checks() {
    log "Running custom security checks..."
    # Placeholder for custom security checks
}

# Reporting
generate_report() {
    log "Generating security audit report..."
    # Placeholder for report generation logic
}

# Main
main() {
    audit_users_groups
    audit_permissions
    audit_services
    audit_firewall_network
    check_ip_addresses
    check_updates
    monitor_logs
    hardening_steps
    custom_checks
    generate_report
}

main















