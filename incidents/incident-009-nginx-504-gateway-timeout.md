# Incident 009: NGINX HTTP 504 Gateway Timeout

## Environment

- Cloud: Google Cloud Platform
- Project: gcp-devops-portfolio-2026
- VM: devops-504-lab-01
- Zone: asia-south1-a
- Machine type: e2-micro
- OS: Ubuntu 24.04.5 LTS
- Reverse proxy: NGINX 1.24.0
- Backend: Python HTTP server
- Backend address: 127.0.0.1:8000

## Incident Summary

NGINX returned HTTP 504 Gateway Time-out because the
backend application took longer to respond than the
configured upstream read timeout.

This was a controlled production-style incident simulation.

## Architecture

Client -> NGINX (Port 80) -> Python Backend (Port 8000)

## Healthy Baseline

Verified backend connectivity:

    curl -i http://127.0.0.1:8000/

Result:

    HTTP/1.0 200 OK

Verified the reverse proxy:

    curl -i http://127.0.0.1/

Result:

    HTTP/1.1 200 OK

Response:

    Backend Application Healthy - GCP Day 13

## Failure Simulation

The Python backend contained an endpoint that deliberately
delayed its response by 15 seconds:

    if self.path == "/slow":
        time.sleep(15)

NGINX was configured with:

    proxy_connect_timeout 5s;
    proxy_read_timeout 5s;
    proxy_send_timeout 5s;

The 15-second backend delay exceeded the 5-second
upstream read timeout.

## Incident Detection

Command:

    curl -i -w '\nTotal response time: %{time_total}s\n' \
      http://127.0.0.1/slow

Observed:

    HTTP/1.1 504 Gateway Time-out
    Total response time: 5.006294s

## Investigation

Inspected NGINX error logs:

    sudo tail -n 15 /var/log/nginx/error.log

Observed:

    upstream timed out (110: Connection timed out)
    while reading response header from upstream

Affected upstream:

    http://127.0.0.1:8000/slow

## Root Cause

The backend accepted the upstream connection but did not
send its response headers within NGINX's configured
5-second proxy_read_timeout.

The backend's deliberate 15-second delay caused NGINX
to return HTTP 504.

## Resolution

Removed the artificial backend delay:

    time.sleep(15)

Changed to:

    time.sleep(0)

Restarted the Python backend.

The NGINX timeout configuration was left unchanged.

## Recovery Verification

Command:

    curl -s -o /dev/null \
      -w 'HTTP %{http_code} | Response time: %{time_total}s\n' \
      http://127.0.0.1/slow

Observed:

    HTTP 200 | Response time: 0.003626s

## Key Lessons

1. HTTP 504 indicates that a gateway did not receive a
   timely response from its upstream.
2. A running backend can still be too slow to serve requests.
3. NGINX error logs identify upstream timeout failures.
4. proxy_read_timeout controls the idle time between
   successive reads from the upstream.
5. Increasing timeout values does not reduce backend latency.
6. Investigate application logs, database queries,
   resource utilization, and downstream dependencies.
7. Timeout configuration should reflect application
   behavior and latency requirements.
8. Verify recovery using both HTTP status and response time.
9. Evaluate production fixes against latency and
   availability service-level objectives.
10. Avoid masking performance problems with arbitrary
    timeout increases.

## HTTP 502 vs HTTP 504

HTTP 502:
The gateway receives an invalid upstream response or
encounters certain upstream connection failures.

HTTP 504:
The gateway does not receive a timely upstream response.

## Cleanup

Deleted the temporary Compute Engine VM:

    devops-504-lab-01

## Incident Classification

Controlled training simulation.
No production systems affected.
