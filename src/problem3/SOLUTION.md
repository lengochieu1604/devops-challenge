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
        
        <img width="1291" alt="image" src="https://github.com/user-attachments/assets/9e3f51b7-3917-429f-9797-6dcdb02a4908" />
        
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

### **Confirm the Alert**

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
    

### **Start Troubleshooting Process**

| **Order to troubleshoot** | **Causes/scenario expect to encounter** | **Solution to detect this element is a cause or not** | **Description** | **Recovery steps** | **Impact** |
| --- | --- | --- | --- | --- | --- |
| **1** | **Traffic Overhead** | Check live traffic using `nginx access logs` (`tail -f /var/log/nginx/access.log`), analyze system load using `htop` or `top` | Sudden spike in user traffic (e.g., live stream, flash sale) | Scale VM resources if possible (Memory), enable rate-limiting, configure WAF/CDN | Prevents overload, ensures system stability but it can lead to the downtime for a NGINX service based on the HA property of the Cloud  scalability services |
| **2** | **NGINX LB - Logging Overhead** | Check logging level in `nginx.conf` (`grep error_log /etc/nginx/nginx.conf`) and inspect log file size (`du -sh /var/log/nginx/`) | Excessive logging enabled (debug mode) | Change `error_log` level to `warn` or `error` in `nginx.conf` | Reduces disk I/O and memory consumption |
|  | **NGINX LB - Child Processes** | Check active worker processes using `ps aux --sort=-%mem` | Worker processes consuming too much memory | Reduce `worker_processes` value, optimize `worker_connections` | Optimizes memory allocation, improves efficiency |
|  | **NGINX LB - Proxy Buffers** | Check buffer settings: `grep -i proxy_buffer /etc/nginx/nginx.conf` | Large proxy buffer settings | Reduce `proxy_buffer_size`, `proxy_buffers` in `nginx.conf` | Prevents excessive memory use for buffering responses |
|  | **NGINX LB - Zombie Processes** | Identify zombies: `ps aux` | Defunct processes not terminating | Identify and kill zombie processes using `ps aux` and `kill` command | Frees up wasted memory and system resources |
|  | **NGINX LB - Misconfiguration** | Check worker settings: `grep worker_processes /etc/nginx/nginx.conf` | Inefficient worker/thread allocation | Optimize worker settings based on CPU cores and traffic load | Improves request handling efficiency |
|  | **NGINX LB - Memory Leak** | Monitor memory growth: `top` Check memory allocation: `pmap -x <NGINX_PID>` | Unreleased memory due to faulty modules | Restart NGINX periodically, inspect memory allocation with `pmap` | Prevents gradual memory exhaustion over time |
|  | **NGINX LB - Third-Party** | Check module usage: `nginx -V 2>&1` | JWT authentication module consuming memory | Disable/optimize JWT processing or offload to a dedicated service | Reduces authentication overhead, frees up memory |
| **3** | **System - Kernel Memory Leak** | Check kernel slab usage: `cat /proc/slabinfo`Monitor memory leaks: `dmesg` | Kernel not releasing memory properly | Check `slabinfo`, restart VM if necessary, apply kernel updates | Ensures proper memory management and system stability |
|  | **System - OOM (Out of Memory)** | Check for OOM Killer Events:
`dmesg | grep -i "oom"` | System running out of available memory | Optimize NGINX settings, limit process memory usage, enable OOM killer | Prevents system crashes and performance degradation |
|  | **System - Swap Overhead** | Check swap usage: `swapon -s` or `free -m` | System relying heavily on swap | Increase swap space or optimize memory allocation to avoid swapping | Improves response times, prevents slowdowns |
|  | **System - Zombie Processes** | Identify zombies: `ps aux` | Orphaned processes consuming memory | Use `ps aux` | Reclaims system memory for active processes |
| **4** | **External Threats - DOS/DDOS** | Check active connections: `netstat -ant` | Attackers flooding NGINX with requests | Implement rate limiting, configure WAF | Protects against service disruption and high memory usage |
|  | **External Threats - Slow Client Attacks** | Check long-lived connections: `netstat -antp` | Clients holding connections open | Reduce `keepalive_timeout`, limit max connections per client | Ensures connection efficiency and prevents resource exhaustion |
|  | **External Threats - Rate-Limiting** | Check rate-limiting: `grep limit_req_zone /etc/nginx/nginx.conf` | No request limits in place | Enable NGINX `limit_req_zone` to throttle excessive requests | Prevents abuse and ensures fair resource allocation |
|  | **External Threats - Malformed HTTP Requests** | Check logs: `grep 'invalid request' /var/log/nginx/error.log` | Attackers sending malformed requests | Block invalid requests using `if` conditions in `nginx.conf`. | Enhances security and reduces processing overhead |

---

### Follow-up actions

By following the above approach, we can **quickly identify the root cause** of high memory usage on the NGINX load balancer VM. Continuous monitoring, proper configuration, and preventive measures will ensure **stable system performance** and **mitigate future incidents**.

# Refers

- https://nginx.org/en/docs/beginners_guide.html
- https://github.com/engintron/engintron/issues/1398
