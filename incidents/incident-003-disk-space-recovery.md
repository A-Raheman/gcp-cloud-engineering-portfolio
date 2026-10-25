# Incident 003: Linux Disk Space Investigation and Recovery

## Environment

- Cloud: Google Cloud Platform
- VM: devops-disk-lab-01
- Region: asia-south1
- Operating system: Ubuntu 24.04 LTS
- Machine type: e2-micro
- Boot disk: 10 GB standard persistent disk

## Incident Summary

A controlled disk-space consumption incident was simulated
on a Linux VM using a disposable 2 GB test file.

Root filesystem utilization increased from 28% to 51%.

The incident was investigated using standard Linux
filesystem diagnostic commands.

## Initial Investigation

Command:

    df -h /

Result:

    Filesystem  Size  Used  Avail  Use%  Mounted on
    /dev/root   8.7G  2.4G  6.3G   28%  /

The root filesystem was healthy before the simulation.

## Controlled Simulation

Command:

    sudo fallocate -l 2G /var/tmp/finpay-disk-test.img

After allocation:

    Filesystem  Size  Used  Avail  Use%  Mounted on
    /dev/root   8.7G  4.4G  4.3G   51%  /

## Investigation

Checked top-level directories:

    sudo du -xhd1 / 2>/dev/null

Investigated /var:

    sudo du -xhd1 /var 2>/dev/null

Identified approximately 2.1 GB in /var/tmp.

Located the large file:

    sudo find /var/tmp -xdev -type f -size +100M -exec ls -lh {} \;

Result:

    /var/tmp/finpay-disk-test.img (2.0 GB)

## Root Cause

A deliberately allocated 2 GB test file consumed additional
space on the root filesystem.

This was a controlled lab exercise, not an actual production
disk exhaustion event.

## Recovery

After confirming that the file was disposable:

    sudo rm -i /var/tmp/finpay-disk-test.img

## Recovery Verification

Command:

    df -h /

Result:

    Filesystem  Size  Used  Avail  Use%  Mounted on
    /dev/root   8.7G  2.4G  6.3G   28%  /

Approximately 2 GB of disk space was recovered.

## Additional Observation

An SSH connection was interrupted during the investigation,
returning a broken-pipe error.

Access was subsequently restored through Google Cloud IAP SSH.

The cause of the connection interruption was not established.

## Lessons Learned

1. Use df to identify filesystem utilization.
2. Use du to narrow down directory-level storage consumption.
3. Use find to locate large files.
4. Investigate file ownership and purpose before deletion.
5. Verify available space after recovery.
6. Avoid deleting system directories without understanding their role.
7. Document the evidence and root cause.
8. Remove temporary cloud resources after completing the exercise.

## Cleanup

The temporary Compute Engine instance was deleted successfully.
