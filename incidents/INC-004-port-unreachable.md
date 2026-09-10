# INC-004 — Port Unreachable

## Incident Summary

Customers were unable to reach the web server through HTTP. External connectivity testing showed that requests to the server's HTTP endpoint were failing.

The investigation confirmed that Nginx was running and port 80 was listening correctly on the EC2 instance. The root cause was an AWS Security Group configuration that did not contain an inbound HTTP rule for TCP port 80.

## Impact

Customers were unable to access the web application through HTTP.

If this occurred in a real production environment, users could experience:

* Connection timeouts
* Application unavailability
* Failed HTTP requests
* Loss of access to a customer-facing service

## Symptoms

An external connectivity test was performed:

```bash
curl -i http://<EC2-PUBLIC-IP>
```

The request failed with a connection error.

## Investigation

### 1. Verify external connectivity

The external request failed, confirming that the server could not be reached through HTTP from the client side.

### 2. Check the Nginx service

The Nginx service was checked using:

```bash
sudo systemctl status nginx --no-pager
```

Result:

```text
Active: active (running)
```

This confirmed that the web server application itself was running.

### 3. Check whether port 80 was listening

The listening ports were checked using:

```bash
sudo ss -lntp | grep :80
```

Port 80 was found to be listening:

```text
0.0.0.0:80
[::]:80
```

This confirmed that Nginx was accepting connections on the expected HTTP port.

### 4. Check the AWS Security Group

Since the application was running and port 80 was listening, the next layer investigated was the AWS network security configuration.

The EC2 instance's Security Group was checked.

The investigation found that there was no inbound rule allowing HTTP traffic on TCP port 80.

## Root Cause

The AWS Security Group `linux-production-sg` did not have an inbound HTTP rule for TCP port 80.

Therefore:

```text
Client
  ↓
Internet
  ↓
AWS Security Group
  ↓
HTTP/80 blocked ❌
  ↓
EC2 / Nginx
```

Nginx itself was healthy, but the network security layer prevented external HTTP traffic from reaching the instance.

## Resolution

An inbound HTTP rule was added to the Security Group:

```text
Type:     HTTP
Protocol: TCP
Port:     80
Source:   0.0.0.0/0
```

The existing SSH rule on port 22 was left unchanged.

## Verification

After restoring the HTTP rule, external connectivity was tested again:

```bash
curl -i http://3.236.165.179
```

The request successfully returned:

```text
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
```

The Nginx welcome page was also returned successfully.

This confirmed that external HTTP connectivity had been restored.

## Preventive Actions

For a real production environment:

1. Use infrastructure-as-code to manage Security Group rules consistently.
2. Review Security Group changes through a controlled change-management process.
3. Monitor HTTP endpoint availability from outside the infrastructure.
4. Configure alerts for failed health checks.
5. Maintain documented inbound and outbound network requirements.
6. Restrict Security Group sources to only the required networks where possible instead of unnecessarily allowing `0.0.0.0/0`.
7. Audit Security Group changes to identify unexpected configuration changes.

## Troubleshooting Approach

The incident followed this sequence:

```text
Customer Symptom
      ↓
External Connectivity Test
      ↓
Application/Service Check
      ↓
Port/Listener Check
      ↓
AWS Security Group Check
      ↓
Identify Missing HTTP Rule
      ↓
Restore HTTP/80
      ↓
External Verification
```

## Lessons Learned

* A failed external connection does not automatically mean the application is down.
* Always verify the application/service before restarting it.
* `ss -lntp` helps determine whether the expected port is actually listening.
* AWS Security Groups control inbound network access to EC2 instances.
* Troubleshooting should move systematically through the application, operating system, and network layers.
* Always perform an external verification after making a network configuration change.

## Incident Conclusion

The incident was caused by a missing AWS Security Group inbound rule for HTTP port 80.

Nginx and the Linux server were healthy. Restoring the Security Group rule resolved the connectivity issue and restored successful external HTTP access.
