# INC-001 — Nginx Service Down

## Incident

The Nginx web server unexpectedly stopped running on the production Linux server, causing the website to become unavailable to customers.

## Impact

Customers were unable to connect to the website. External HTTP requests resulted in connection timeouts/failures because the server was not accepting connections on port 80.

## Symptoms

* Website was inaccessible from the Internet.
* External `curl` request failed with a connection error.
* Nginx service was found to be inactive.
* Port 80 was not listening for incoming connections.

## Investigation

### 1. Checked Nginx service status

```bash
sudo systemctl status nginx
```

Result:

```text
Active: inactive (dead)
```

This confirmed that the Nginx service was stopped.

### 2. Checked whether port 80 was listening

```bash
sudo ss -lntp | grep :80
```

No output was returned.

This confirmed that nothing was listening on TCP port 80.

### 3. Tested external connectivity

From the local workstation:

```bash
curl -i http://3.236.165.179
```

The request failed with:

```text
curl: (7) Failed to connect to 3.236.165.179 port 80
```

This confirmed that the customer-facing HTTP endpoint was unavailable.

## Root Cause

The Nginx service had stopped running.

Because Nginx was responsible for accepting HTTP connections on port 80, stopping the service caused port 80 to stop listening and made the website unreachable.

## Resolution

After confirming the root cause, Nginx was started:

```bash
sudo systemctl start nginx
```

The service was then checked:

```bash
sudo systemctl status nginx
```

Result:

```text
Active: active (running)
```

Port 80 was checked again:

```bash
sudo ss -lntp | grep :80
```

Nginx was listening on:

```text
0.0.0.0:80
[::]:80
```

## Verification

External connectivity was tested again:

```bash
curl -i http://3.236.165.179
```

Result:

```text
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
```

The Nginx welcome page was successfully returned, confirming that the service and customer-facing HTTP endpoint were restored.

## Preventive Actions

1. Configure monitoring for the Nginx service and generate an alert if it becomes inactive.
2. Monitor HTTP port 80 and perform regular health checks against the website.
3. Configure an appropriate systemd restart policy so Nginx can automatically recover from certain unexpected failures.
4. Monitor Nginx and system logs to identify recurring service failures.
5. Introduce centralized monitoring and alerting using Prometheus and Grafana in a later stage of the project.
6. Document the troubleshooting procedure so future incidents can be diagnosed and resolved quickly.

## Lessons Learned

The incident demonstrated the importance of investigating before applying a restart.

The troubleshooting process was:

```text
Customer symptom
       ↓
External connectivity test
       ↓
Service status
       ↓
Port/listener check
       ↓
Root cause identification
       ↓
Service recovery
       ↓
External verification
```

The key lesson is to collect evidence before making changes. This helps identify the actual failure and prevents unnecessary or blind restarts.
