<aside>
⏰ Duration: You should not spend more than **12 hours** on this problem.
*Time estimation is for internship roles, if you are a software professional you should spend significantly less time.*

</aside>

# Task

Provide steps to troubleshoot two production issues.

### Scenario

You are working as a DevOps Engineer on a cloud-based infrastructure where a virtual machine (VM), running **Ubuntu 24.04**, with **64GB** of storage is under your management. Recently, your monitoring tools have reported that the VM is consistently running at **99% memory usage**. This VM is responsible for only running **one service - a NGINX load balancer as a traffic router** for **upstream services.**

### Challenge

Describe **how** you would **troubleshoot** the issue, and what **causes/scenario** do you expect to **encounter**, as well as what are the **impacts** and **recovery steps** for each possible root cause.

# Analysis

**Input**

- Cloud-based
- VM
    - Ubuntu 24.04
    - Mem total: 64GB
    - Mem usage: 99%
    - Service: ONLY Nginx LB as a traffic router for upstream services

**Assumption**

- Infrastructure architecture
    - AWS/Azure/GCP, etc.
    - Architecture diagram
      
        ![Diagrams-99Tech Nginx V1 0 0 drawio](https://github.com/user-attachments/assets/25e81283-d472-41fd-babe-1dde764d836f)
        
    - This diagram represents a **cloud-based assumption architecture** with a **multi-tiered network design**, which is divided into three key sections:
        1. **External Network (End Users & Attackers)**
            - **End User Devices:** These are external clients (browsers, mobile apps) trying to access services.
            - **Potential Attackers:** The architecture considers possible cyber threats that attempt unauthorized access, DOS/DDOS, slow client attacks, etc.
        2. **Cloud Provider Infrastructure**
            - **DMZ (Demilitarized Zone)**
                - Domain Service
                - **Firewall:** Controls incoming traffic, filtering out malicious requests before traffic enters the LAN.
            - **LAN (Local Area Network)**
                - **Tier 1 (Public Subnets):** Hosts an **NGINX Load Balancer (LB) VM**, which is the first point of contact for traffic from external users.
                - **Tier 2 (Can be Public or Private Subnets):** Contains **Upstream Services**
- Business flow
    - N/A
- Traffic flow
    - End user → Domain Service → Firewall → VM (NginxLB) → VM Services (Upstream service 1, 2, etc.)

**Troubleshooting process**

- Alert systems have raised a notification the VM memory usage is 99%
- DevOps / SRE Engineer have joined to troubleshoot
- Confirm the notification which is raised by an Alert systems
    - Go through a/an alert/email/message information
    - Verify memory usage
    - Check the top memory-consuming processes
    - etc.
- Start a troubleshoot process to identify the culprit
    - Traffic overhead
        - Sudden spike in user traffic (live stream, flash sale, etc.)
    - NGINX LB
        - Operation activies that consumed excessive memory
            - Logging
            - Child processes
                - Worker
                - Proxy buffers
            - Zombie processes
        - Misconfiguration
        - Memory Leak
        - Third-Party
            - JWT authentication
    - System
        - Kernel
            - Memory leak
        - OOM (Out of memory)
        - Swap overhead
        - Processes
            - Zombie
    - External threats
        - DOS/DDOS
        - Slow Client Attacks
        - Rate-limitting
        - Malformed HTTP Requests

# Solution

### **1. Confirm the Alert**

- Verify memory usage using:
    
    ```bash
    free -h
    vmstat -s
    ```
    
- Check the top memory-consuming processes:
    
    ```bash
    ps aux --sort=-%mem | head -n 10
    ```
    
- Confirm swap usage:
    
    ```bash
    swapon --summary
    ```
    
- Analyze system logs:
    
    ```bash
    journalctl -xe | tail -n 50
    dmesg | tail -n 50
    ```
    

### **2. Start Troubleshooting Process**

| **Causes/scenario expect to encounter order** | **Description** | **Recovery steps** | **Impact** |
| --- | --- | --- | --- |
| **Traffic Overhead** | Sudden spike in user traffic (e.g., live stream, flash sale) | Scale VM resources if possible (Memory), enable rate-limiting, configure WAF/CDN | Prevents overload, ensures system stability but it can lead to the downtime for a NGINX service based on the HA property of the Cloud  scalability services |
| **NGINX LB - Logging Overhead** | Excessive logging enabled (debug mode) | Change `error_log` level to `warn` or `error` in `nginx.conf` | Reduces disk I/O and memory consumption |
| **NGINX LB - Child Processes** | Worker processes consuming too much memory | Reduce `worker_processes` value, optimize `worker_connections` | Optimizes memory allocation, improves efficiency |
| **NGINX LB - Proxy Buffers** | Large proxy buffer settings | Reduce `proxy_buffer_size`, `proxy_buffers` in `nginx.conf` | Prevents excessive memory use for buffering responses |
| **NGINX LB - Zombie Processes** | Defunct processes not terminating | Identify and kill zombie processes using `ps aux` | Frees up wasted memory and system resources |
| **NGINX LB - Misconfiguration** | Inefficient worker/thread allocation | Optimize worker settings based on CPU cores and traffic load | Improves request handling efficiency |
| **NGINX LB - Memory Leak** | Unreleased memory due to faulty modules | Restart NGINX periodically, inspect memory allocation with `pmap` | Prevents gradual memory exhaustion over time |
| **NGINX LB - Third-Party** | JWT authentication module consuming memory | Disable/optimize JWT processing or offload to a dedicated service | Reduces authentication overhead, frees up memory |
| **System - Kernel Memory Leak** | Kernel not releasing memory properly | Check `slabinfo`, restart VM if necessary, apply kernel updates | Ensures proper memory management and system stability |
| **System - OOM (Out of Memory)** | System running out of available memory | Optimize NGINX settings, limit process memory usage, enable OOM killer | Prevents system crashes and performance degradation |
| **System - Swap Overhead** | System relying heavily on swap | Increase swap space or optimize memory allocation to avoid swapping | Improves response times, prevents slowdowns |
| **System - Zombie Processes** | Orphaned processes consuming memory | Use `ps aux` | Reclaims system memory for active processes |
| **External Threats - DOS/DDOS** | Attackers flooding NGINX with requests | Implement rate limiting, configure WAF | Protects against service disruption and high memory usage |
| **External Threats - Slow Client Attacks** | Clients holding connections open | Reduce `keepalive_timeout`, limit max connections per client | Ensures connection efficiency and prevents resource exhaustion |
| **External Threats - Rate-Limiting** | No request limits in place | Enable NGINX `limit_req_zone` to throttle excessive requests | Prevents abuse and ensures fair resource allocation |
| **External Threats - Malformed HTTP Requests** | Attackers sending malformed requests | Block invalid requests using `if`  | Enhances security and reduces processing overhead |

---

## **3. Resolution and Mitigation Steps**

### **Immediate Fixes**

1. Restart NGINX if memory is stuck at 99%:
    
    ```bash
    systemctl restart nginx
    ```
    
2. Clear memory cache:
    
    ```bash
    sync; echo 3 > /proc/sys/vm/drop_caches
    ```
    
3. Enable rate-limiting if under attack.
4. Adjust NGINX buffer settings to reduce memory usage.
    
    ### **Long-Term Fixes**
    
5. **Optimize NGINX Configuration:**
    - Adjust `worker_processes`, `worker_connections`, and buffer sizes.
    - Implement proper keepalive settings.
6. **Implement Auto-Healing:**
    - Create a cron job to restart NGINX if memory exceeds 90%.
7. **Set Up Additional Security Measures:**
    - Implement WAF to block malicious traffic.
8. **Upgrade System and Packages:**
    
    ```bash
    sudo apt update && sudo apt upgrade -y
    ```
    

---

### Follow-up actions

By following the above approach, we can **quickly identify the root cause** of high memory usage on the NGINX load balancer VM. Continuous monitoring, proper configuration, and preventive measures will ensure **stable system performance** and **mitigate future incidents**.

# Refers

- https://nginx.org/en/docs/beginners_guide.html
- https://github.com/engintron/engintron/issues/1398
