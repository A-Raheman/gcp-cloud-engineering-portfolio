# Incident 008: NGINX HTTP 502 Bad Gateway

## Environment

- Cloud: Google Cloud Platform
- VM: devops-502-lab-01
- Region: asia-south1
- Zone: asia-south1-a
- OS: Ubuntu 24.04.5 LTS
- Web server: NGINX
- Backend: Python HTTP server
- Backend port: 8000

## Incident Summary

NGINX returned HTTP 502 Bad Gateway when its upstream
backend application became unavailable.

This was a controlled production-style incident simulation.

## Architecture

Client -> NGINX (Port 80) -> Python Backend (Port 8000)

## Healthy Baseline

The backend responded successfully:

    curl -i http://127.0.0.1:8000

Result:

    HTTP/1.0 200 OK

The NGINX reverse proxy also returned the expected
backend application content.

## Failure Simulation

Stopped the Python backend process:

    kill 2061

NGINX remained running, but its upstream became unavailable.

## Incident Detection

Command:

    curl -i http://127.0.0.1/

Result:

    HTTP/1.1 502 Bad Gateway

## Investigation

Inspected NGINX error logs:

    sudo tail -n 10 /var/log/nginx/error.log

Observed:

    connect() failed (111: Connection refused) while connecting to upstream

Upstream:

    http://127.0.0.1:8000/

Earlier socket inspection confirmed that Python was
listening on 127.0.0.1:8000.

## Root Cause

The Python backend process was terminated.

NGINX attempted to connect to port 8000, but no backend
application was listening.

The operating system refused the connection, causing
NGINX to return HTTP 502.

## Resolution

Restarted the backend:

    cd ~/backend-app
    nohup python3 -m http.server 8000 --bind 127.0.0.1 > ~/backend.log 2>&1 &

New backend PID:

    2496

## Recovery Verification

Command:

    curl -s -o /dev/null -w 'HTTP %{http_code}\n' http://127.0.0.1/

Result:

    HTTP 200

Application availability was restored.

## Lessons Learned

1. A running NGINX service does not guarantee application health.
2. HTTP 502 can indicate an unavailable upstream application.
3. Check NGINX error logs before restarting services.
4. Connection refused usually means no process is accepting connections at the target address and port.
5. Verify listening ports using ss -lntp.
6. Test backend connectivity directly with curl.
7. Restore the failed backend rather than unnecessarily restarting NGINX.
8. Verify recovery through the reverse proxy, not just the backend.
9. A successful HTTP response should also contain the expected application content.
10. Use process supervision and health monitoring for production backends.

## Additional Troubleshooting Observation

During initial setup, the backend was started on port 8000
while a test request targeted port 8080.

This demonstrated how a port mismatch can cause connection
failures even when the application is running.

## Cleanup

The temporary GCP VM was deleted successfully.
