# Incident 007: NGINX HTTP 403 — Permission Denied

## Environment

- Cloud: Google Cloud Platform
- VM: devops-permission-lab-01
- Zone: asia-south1-a
- Machine type: e2-micro
- OS: Ubuntu 24.04.5 LTS
- Web server: NGINX 1.24.0
- Service manager: systemd

## Incident Summary

A controlled Linux file-permission error caused NGINX
to return HTTP 403 Forbidden.

The web server remained running, but it could not read
the default homepage.

## Healthy Baseline

Command:

    curl -s -o /dev/null -w 'HTTP %{http_code}\n' http://localhost

Result:

    HTTP 200

## Failure Simulation

Removed permissions from the NGINX homepage:

    sudo chmod 000 /var/www/html/index.nginx-debian.html

## Incident Detection

Command:

    curl -I http://localhost

Result:

    HTTP/1.1 403 Forbidden

NGINX was reachable but refused to serve the requested file.

## Investigation

Inspected NGINX error logs:

    sudo tail -n 20 /var/log/nginx/error.log

Observed:

    open() "/var/www/html/index.nginx-debian.html" failed (13: Permission denied)

Inspected file permissions:

    ls -l /var/www/html/index.nginx-debian.html

Observed:

    ---------- 1 root root 615 index.nginx-debian.html

The file existed, but no user category had read permission.

## Root Cause

The homepage file's permissions were changed to 000,
preventing the NGINX worker process from reading it.

This resulted in HTTP 403 Forbidden.

## Resolution

Restored appropriate read permissions:

    sudo chmod 644 /var/www/html/index.nginx-debian.html

Permissions 644 allow:

- Owner: read and write
- Group: read only
- Others: read only

NGINX did not require a restart.

## Recovery Verification

Command:

    curl -s -o /dev/null -w 'HTTP %{http_code}\n' http://localhost

Result:

    HTTP 200

The website was restored successfully.

## Lessons Learned

1. HTTP 403 does not necessarily mean the web server is stopped.
2. Check the HTTP response before changing services.
3. NGINX error logs can identify permission failures.
4. Use ls -l to inspect file permissions.
5. Apply the minimum permissions necessary for application access.
6. Avoid chmod 777 as a troubleshooting shortcut.
7. File permission changes usually do not require an NGINX restart.
8. Verify application recovery using an HTTP request.
9. Prefer targeted fixes over unnecessary service restarts.

## Cleanup

The temporary GCP Compute Engine VM was deleted successfully.
