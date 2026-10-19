# Incident 001: NGINX Website Outage

## Environment
- Cloud: Google Cloud Platform
- Region: asia-south1 (Mumbai)
- VM: devops-web-01 (e2-micro)
- OS: Ubuntu 24.04 LTS
- Web server: NGINX 1.24.0
- Network: Custom VPC with HTTP firewall and IAP SSH access

## Incident Summary
A controlled application outage was simulated by stopping NGINX.
The VM remained operational, but the public website became unreachable.

## Symptoms
- Browser reported that the website was unreachable.
- NGINX service status showed `inactive (dead)`.

## Investigation
Checked the application service:

    sudo systemctl status nginx --no-pager

The service was inactive even though it was enabled at boot.

## Root Cause
NGINX was intentionally stopped using:

    sudo systemctl stop nginx

This was an application-level failure, not a VM failure.

## Resolution
Restarted the service:

    sudo systemctl start nginx

## Verification
- NGINX welcome page became accessible again.
- Before the incident, `curl -I http://localhost` returned HTTP 200.
- NGINX was confirmed listening on TCP port 80.

## Lessons Learned
1. A running VM does not guarantee application availability.
2. `enabled` and `active (running)` have different meanings.
3. Investigate application health before rebooting infrastructure.
4. Verify recovery from the end-user perspective.

## Cleanup
Deleted the lab VM after the exercise to avoid unnecessary cloud charges.
