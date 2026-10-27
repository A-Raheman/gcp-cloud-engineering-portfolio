# Incident 006: NGINX Port Conflict — Address Already in Use

## Environment

- Cloud: Google Cloud Platform
- VM: devops-port-conflict-lab-01
- Zone: asia-south1-a
- Machine type: e2-micro
- OS: Ubuntu 24.04.5 LTS
- Web server: NGINX 1.24.0
- Service manager: systemd

## Incident Summary

A controlled application port conflict prevented NGINX
from starting because another application occupied TCP port 80.

The objective was to identify the conflicting process,
confirm the root cause through system logs, restore NGINX,
and verify HTTP availability.

## Healthy Baseline

Command:

    sudo ss -ltnp '( sport = :80 )'

Observed:

- NGINX listening on 0.0.0.0:80 and [::]:80
- NGINX process IDs: 1887, 1890, 1891

## Incident Simulation

Stopped NGINX:

    sudo systemctl stop nginx

Started a temporary Python HTTP server:

    sudo python3 -m http.server 80 --bind 0.0.0.0 > /tmp/incident-006-python.log 2>&1 & echo $!

The shell reported background job PID 1999.

## Investigation

Identified the process occupying port 80:

    sudo ss -ltnp '( sport = :80 )'

Observed:

    LISTEN 0 5 0.0.0.0:80 0.0.0.0:* users:(("python3",pid=2001,fd=3))

Attempted to start NGINX:

    sudo systemctl start nginx

Result: Service startup failed.

Investigated systemd logs:

    sudo journalctl -u nginx.service -n 25 --no-pager

Observed error:

    bind() to 0.0.0.0:80 failed (98: Address already in use)

NGINX repeatedly attempted to bind to port 80,
then exited with status 1/FAILURE.

## Root Cause

A Python HTTP server was already listening on
0.0.0.0:80, preventing NGINX from binding to the
same address and port.

## Resolution

Terminated the known conflicting process:

    sudo kill -15 2001

Verified that port 80 was released:

    sudo ss -ltnp '( sport = :80 )'

The command returned no listening processes.

Started NGINX:

    sudo systemctl start nginx

## Recovery Verification

Commands:

    systemctl is-active nginx
    curl -s -o /dev/null -w 'HTTP %{http_code}\n' http://localhost

Results:

    active
    HTTP 200

The web service was successfully restored.

## Lessons Learned

1. Use ss -ltnp to identify applications listening on TCP ports.
2. Check the actual application PID rather than assuming the shell job PID.
3. Use journalctl to confirm the exact service startup error.
4. Error 98 indicates that the requested address is already in use.
5. Investigate process ownership before terminating services.
6. Verify that the conflicting port is released before restarting.
7. Confirm both service health and HTTP availability after recovery.
8. Avoid stopping production services without authorization.

## Cleanup

The temporary GCP Compute Engine VM was deleted successfully.
