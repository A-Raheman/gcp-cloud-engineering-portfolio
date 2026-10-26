# Incident 005: NGINX Service Failure — systemd Troubleshooting

## Environment

- Cloud: Google Cloud Platform
- VM: devops-systemd-lab-01
- Zone: asia-south1-a
- OS: Ubuntu 24.04
- Web server: NGINX 1.24.0
- Service manager: systemd

## Incident Summary

A controlled configuration error prevented NGINX from restarting.

The service entered a failed state, resulting in a simulated
web application outage.

## Initial Health Check

Command:

    systemctl status nginx --no-pager

Observed:

- Service enabled
- Active state: running
- Main PID: 1884
- Memory usage: 2.4 MiB

## Failure Simulation

Created a deliberately invalid configuration:

    echo 'invalid_directive on;' | sudo tee /etc/nginx/conf.d/incident-test.conf

Validated the configuration:

    sudo nginx -t

Result:

    unknown directive "invalid_directive"
    /etc/nginx/conf.d/incident-test.conf:1

Attempted service restart:

    sudo systemctl restart nginx

Result: Restart failed.

## Investigation

Commands:

    sudo systemctl status nginx --no-pager -l
    sudo journalctl -u nginx.service -n 20 --no-pager

Findings:

- Active: failed (Result: exit-code)
- ExecStartPre configuration validation failed
- Exit status: 1/FAILURE
- NGINX rejected the invalid directive

## Root Cause

An unsupported directive was introduced into the NGINX
configuration.

The service restart stopped the existing NGINX process,
but the configuration validation failure prevented startup.

## Resolution

Removed the faulty configuration:

    sudo rm /etc/nginx/conf.d/incident-test.conf

Validated the repaired configuration:

    sudo nginx -t

Started the service:

    sudo systemctl start nginx

## Recovery Verification

Commands:

    systemctl is-active nginx
    curl -s -o /dev/null -w 'HTTP %{http_code}\n' http://localhost

Results:

    active
    HTTP 200

The service was restored successfully.

## Lessons Learned

1. Validate NGINX configuration before restarting services.
2. systemctl status identifies service state and startup failures.
3. journalctl provides historical service logs.
4. ExecStartPre can prevent startup when validation fails.
5. A failed restart can turn a configuration issue into an outage.
6. Verify both service state and HTTP response after recovery.
7. Diagnose the root cause before making corrective changes.

## Cleanup

The temporary Compute Engine VM was deleted successfully.

## Incident Classification

Controlled training simulation; no production systems affected.
