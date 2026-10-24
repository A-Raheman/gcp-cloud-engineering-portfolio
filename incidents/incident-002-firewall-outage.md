# Incident 002: Website Unreachable Due to GCP Firewall Misconfiguration

## Environment
- Platform: Google Cloud Platform
- Region: asia-south1 (Mumbai)
- VM: devops-web-01 (e2-micro)
- OS: Ubuntu 24.04 LTS
- Web server: NGINX 1.24.0
- VPC: devops-vpc
- Subnet: web-subnet
- Firewall rule: allow-web-http

## Incident Summary
A controlled network outage was simulated by disabling the
GCP firewall rule responsible for permitting inbound HTTP traffic.

The Compute Engine VM and NGINX remained operational,
but external users could no longer access the website.

## Symptoms
- Public website displayed a connection error.
- NGINX remained accessible locally.
- No VM restart or application crash occurred.

## Investigation

### Step 1: Test application locally
    curl -I http://localhost

Result:
    HTTP/1.1 200 OK

### Step 2: Identify the network change
The allow-web-http firewall rule had been deliberately disabled.

### Step 3: Isolate the failure
Local HTTP requests succeeded while public HTTP requests failed.

## Root Cause
The allow-web-http ingress firewall rule was disabled,
preventing inbound TCP port 80 traffic from reaching the VM.

## Resolution
Restored the firewall rule:

    gcloud compute firewall-rules update allow-web-http --no-disabled

## Verification
- Public NGINX welcome page became accessible again.
- Local NGINX service remained healthy throughout the incident.
- No VM reboot or NGINX restart was required.

## Lessons Learned
1. A running VM does not guarantee external application availability.
2. Local HTTP success helps isolate application versus network failures.
3. Firewall rules can interrupt traffic without affecting application health.
4. Reversing the specific faulty change can restore service quickly.
5. Always verify recovery from the user's perspective.

## Cleanup
Deleted the temporary Compute Engine VM after the lab.
