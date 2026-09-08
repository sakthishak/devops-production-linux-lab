# INC-002 — Disk Full

## Incident Summary

A disk-space issue was simulated on the Linux production server by creating a large temporary file. The investigation identified the file consuming approximately 4.9 GiB of disk space. The file was safely removed and disk utilization returned to the original baseline.

## Impact

If this occurred in a real production environment, insufficient disk space could cause:

* Applications to fail when writing files
* Logs to stop being written
* Database or application errors
* Services to become unstable
* System operations to fail when the filesystem reaches 100%

## Symptoms

Initial disk usage:

```text
/dev/root    19G    2.7G    16G    15%
```

After the disk-pressure simulation:

```text
/dev/root    19G    7.6G    11G    42%
```

The root filesystem usage increased significantly.

## Investigation

### 1. Check filesystem usage

Command:

```bash
df -h
```

The root filesystem was found to be at 42% utilization.

### 2. Identify directories consuming disk space

Command:

```bash
sudo du -sh /* 2>/dev/null | sort -h
```

This helped identify where disk space was being consumed.

### 3. Investigate `/tmp`

Command:

```bash
sudo du -sh /tmp/* 2>/dev/null | sort -h
```

The investigation identified:

```text
4.9G    /tmp/disk-test.img
```

This was the file responsible for the additional disk usage.

## Root Cause

The simulated disk-space issue was caused by the temporary file:

```text
/tmp/disk-test.img
```

The file occupied approximately 4.9 GiB on the root filesystem.

## Resolution

The identified temporary file was safely removed:

```bash
sudo rm /tmp/disk-test.img
```

## Verification

After removing the file, disk usage was checked again:

```bash
df -h
```

The root filesystem returned to:

```text
/dev/root    19G    2.7G    16G    15%
```

This confirmed that the disk space had been successfully recovered.

## Preventive Actions

For a real production environment:

1. Monitor filesystem utilization continuously.
2. Configure alerts before the filesystem reaches critical levels.
3. Investigate large files and directories instead of blindly deleting files.
4. Implement log rotation and retention policies.
5. Monitor application-generated temporary files.
6. Regularly review disk-growth trends.
7. Use monitoring tools such as Prometheus and Grafana for visibility and alerting.

## Troubleshooting Approach

The incident followed this sequence:

```text
Detect
  ↓
Measure
  ↓
Locate
  ↓
Identify
  ↓
Recover
  ↓
Verify
```

The key commands were:

```bash
df -h
sudo du -sh /* 2>/dev/null | sort -h
sudo du -sh /tmp/* 2>/dev/null | sort -h
sudo rm /tmp/disk-test.img
df -h
```

## Lessons Learned

* `df -h` shows filesystem-level disk usage.
* `du -sh` helps identify which files or directories are consuming disk space.
* Disk incidents should be investigated using evidence before deleting anything.
* Cleanup should target the confirmed cause rather than blindly deleting files.
* Always verify disk utilization after remediation.
* Monitoring and alerting are important for preventing real disk-full incidents.
