# Incident 004: Linux CPU and Memory Troubleshooting

## Environment
- Cloud: Google Cloud Platform
- VM: devops-performance-lab-01
- Region: asia-south1
- Machine type: e2-micro
- OS: Ubuntu 24.04 LTS
- RAM: Approximately 953 MiB
- Swap: Not configured

## Incident Summary

Two controlled Linux performance incidents were simulated:
1. A CPU-intensive background process.
2. A memory-consuming Python process.

The objective was to identify resource-intensive processes,
investigate their behavior, and verify recovery.

## Baseline Investigation

Commands:

    uptime
    top -b -n 1 | head -n 20
    free -h

Initial load average:

    0.02, 0.04, 0.04

Initial memory:

    Total: 953 MiB
    Used: 314 MiB
    Available: 639 MiB
    Swap: 0 MiB

## CPU Incident

### Simulation

    yes > /dev/null & echo $!

### Investigation

    ps -p 1411 -o pid,ppid,%cpu,%mem,stat,etime,cmd

Observed:

    PID: 1411
    CPU: 99.9%
    Memory: 0.1%
    State: Running
    Process: yes

### Root Cause

The controlled `yes` process continuously executed,
consuming approximately one logical CPU's capacity.

### Resolution

    kill -15 1411

The process was terminated successfully.

### Verification

    ps -p 1411 -o pid,%cpu,stat,cmd

The PID was no longer present.

## Memory Incident

### Simulation

    python3 -c 'import time; data = bytearray(200 * 1024 * 1024); print("Allocated 200 MiB", flush=True); time.sleep(300)' & echo $!

### Investigation

    ps -p 1457 -o pid,%cpu,%mem,rss,stat,etime,cmd

Observed:

    PID: 1457
    CPU: 0.5%
    Memory: 22.0%
    RSS: 215116 KiB
    State: Sleeping

### Root Cause

A controlled Python process allocated approximately
200 MiB of RAM and held the allocation temporarily.

### Resolution

The process completed its scheduled execution.

An attempted SIGTERM returned "No such process"
because the process had already exited.

### Verification

    free -h

Final memory:

    Total: 953 MiB
    Used: 319 MiB
    Available: 633 MiB
    Swap: 0 MiB

Memory availability returned close to baseline.

## Lessons Learned

1. High CPU and high memory usage are different symptoms.
2. Use uptime to examine system load averages.
3. Use top to inspect system-wide resource consumption.
4. Use ps to identify resource-intensive processes.
5. Check whether a process is business-critical before terminating it.
6. SIGTERM requests graceful process termination.
7. Processes may exit naturally before an engineer intervenes.
8. Verify recovery after corrective actions.
9. Avoid excessive memory allocation on small VMs without swap.
10. Remove temporary cloud resources after completing labs.

## Cleanup

The temporary Compute Engine VM was deleted successfully.
