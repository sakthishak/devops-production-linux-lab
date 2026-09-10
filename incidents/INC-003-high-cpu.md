# INC-003 — High CPU Usage

## Incident Summary

The Linux production server experienced abnormally high CPU utilization caused by two processes consuming almost 100% CPU each.

## Initial Condition

The server was initially checked using:

```bash
top
ps aux
```

The system was operating normally before the CPU spike was introduced.

## Symptoms

CPU utilization suddenly increased to approximately 100%.

The `top` command showed two processes consuming almost 100% CPU each, resulting in the available CPU capacity being fully utilized.

## Investigation

To identify the processes consuming the most CPU, I used:

```bash
ps aux --sort=-%cpu | head
```

This sorted the running processes by CPU utilization and displayed the highest CPU-consuming processes at the top.

The investigation identified the two processes responsible for the CPU spike.

## Root Cause

The high CPU utilization was caused by two continuously running `yes` processes that had been intentionally started as part of the incident simulation.

On this 2-vCPU server, each process consumed approximately one full CPU core, resulting in almost complete CPU saturation.

## Resolution

The identified processes were terminated using:

```bash
kill <PID1> <PID2>
```

After terminating the processes, CPU utilization returned to normal.

## Verification

The server was verified using:

```bash
top
```

and:

```bash
ps aux --sort=-%cpu | head
```

The CPU returned to a healthy state, with approximately 99% CPU idle.

## Preventive Actions

In a production environment, the following measures could help prevent or detect similar incidents:

1. Monitor CPU utilization continuously using Amazon CloudWatch or Prometheus.
2. Configure alerts when CPU remains above an agreed threshold, such as 80–90%, for several minutes.
3. Investigate the top CPU-consuming process before terminating it.
4. Review application and system logs to determine why CPU usage increased.
5. Configure appropriate resource limits for workloads where applicable.
6. Consider scaling the server if the workload legitimately requires additional CPU capacity.
7. Monitor CPU trends to identify recurring performance problems.

## Lessons Learned

The troubleshooting process followed a structured approach:

**Monitor → Detect → Identify → Investigate → Resolve → Verify → Prevent**

The key lesson is to use system evidence to identify the actual CPU-consuming process instead of restarting the entire server or blindly terminating processes.
