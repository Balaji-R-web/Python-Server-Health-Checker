# Project 9: Python Server Health Checker

## 1. Project Overview

This project uses Python to monitor basic Linux server health metrics, including CPU usage, memory utilization, disk space, uptime, and running processes.

The script collects system information, checks configurable warning thresholds, and displays a health report in the terminal. It can also save reports to a log file.

## 2. Objectives

- Learn Python system monitoring.
- Check CPU and memory utilization.
- Monitor disk space.
- Display server uptime.
- Identify high resource utilization.
- Generate readable health reports.
- Log monitoring results for troubleshooting.
- Build a foundation for Linux and cloud monitoring automation.

## 3. Environment

- Operating system: Amazon Linux 2023
- Programming language: Python 3
- Library: `psutil`
- Tools: Linux terminal, pip, Git, GitHub

## 4. Step 1: Check Python Installation

Check the Python version:

```bash
python3 --version
```

Example output:

```text
Python 3.x.x
```

Check pip:

```bash
python3 -m pip --version
```

If Python is not installed on Amazon Linux, install it:

```bash
sudo dnf install python3 python3-pip -y
```

Verify again:

```bash
python3 --version
```

## 5. Step 2: Create the Project Directory

Create and enter the project directory:

```bash
mkdir python-server-health-checker
cd python-server-health-checker
```

Create a Python virtual environment:

```bash
python3 -m venv .venv
```

Activate it:

```bash
source .venv/bin/activate
```

Your terminal may now display `(.venv)` at the beginning of the prompt.

Upgrade pip:

```bash
python -m pip install --upgrade pip
```

## 6. Step 3: Install the Required Library

Install `psutil`:

```bash
python -m pip install psutil
```

Verify the installation:

```bash
python -c "import psutil; print(psutil.__version__)"
```

Example output:

```text
7.x.x
```

The version shown depends on the installed release.

**Explanation:** `psutil` provides information about system utilization, memory, disks, uptime-related data, and running processes.

## 7. Step 4: Create the Python Script

Create a file named `server_health.py`:

```bash
vi server_health.py
```

Paste this complete script:

```python
#!/usr/bin/env python3

import os
import time
import platform
import logging
from datetime import datetime

import psutil

# Configurable warning thresholds
CPU_WARNING = 80
MEMORY_WARNING = 80
DISK_WARNING = 80

LOG_FILE = "server_health.log"


logging.basicConfig(
    filename=LOG_FILE,
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
)


def get_cpu_usage():
    """Return CPU usage percentage."""
    return psutil.cpu_percent(interval=1)


def get_memory_usage():
    """Return virtual memory utilization percentage."""
    return psutil.virtual_memory().percent


def get_disk_usage():
    """Return disk usage for the root filesystem."""
    return psutil.disk_usage("/").percent


def get_uptime():
    """Return system uptime in hours."""
    uptime_seconds = time.time() - psutil.boot_time()
    return uptime_seconds / 3600


def get_top_processes(limit=5):
    """Return the processes using the most CPU."""
    processes = []

    for process in psutil.process_iter(
        ["pid", "name", "cpu_percent", "memory_percent"]
    ):
        try:
            info = process.info
            processes.append(info)
        except (psutil.NoSuchProcess, psutil.AccessDenied):
            continue

    return sorted(
        processes,
        key=lambda item: item.get("cpu_percent") or 0,
        reverse=True,
    )[:limit]


def check_threshold(name, value, threshold):
    """Display and log a warning when a threshold is exceeded."""
    if value >= threshold:
        message = (
            f"WARNING: {name} usage is {value:.1f}% "
            f"(threshold: {threshold}%)"
        )
        print(message)
        logging.warning(message)
    else:
        print(f"OK: {name} usage is {value:.1f}%")


def main():
    print("=" * 50)
    print("        PYTHON SERVER HEALTH CHECKER")
    print("=" * 50)

    hostname = platform.node()
    operating_system = platform.system()
    timestamp = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

    cpu = get_cpu_usage()
    memory = get_memory_usage()
    disk = get_disk_usage()
    uptime = get_uptime()

    print(f"Timestamp : {timestamp}")
    print(f"Hostname  : {hostname}")
    print(f"OS        : {operating_system}")
    print(f"Uptime    : {uptime:.2f} hours")
    print("-" * 50)

    print(f"CPU Usage    : {cpu:.1f}%")
    print(f"Memory Usage : {memory:.1f}%")
    print(f"Disk Usage   : {disk:.1f}%")
    print("-" * 50)

    check_threshold("CPU", cpu, CPU_WARNING)
    check_threshold("Memory", memory, MEMORY_WARNING)
    check_threshold("Disk", disk, DISK_WARNING)

    print("-" * 50)
    print("Top Processes by CPU:")

    for process in get_top_processes():
        print(
            f"PID: {process['pid']:<7} "
            f"Name: {str(process['name']):<20} "
            f"CPU: {process.get('cpu_percent') or 0:.1f}% "
            f"Memory: {process.get('memory_percent') or 0:.1f}%"
        )

    print("=" * 50)

    logging.info(
        "Health check completed: host=%s cpu=%.1f%% "
        "memory=%.1f%% disk=%.1f%% uptime=%.2f hours",
        hostname,
        cpu,
        memory,
        disk,
        uptime,
    )

    print(f"Log saved to: {os.path.abspath(LOG_FILE)}")


if __name__ == "__main__":
    main()
```

Save and exit the editor.

## 8. Step 5: Run the Script

Execute the health checker:

```bash
python server_health.py
```

Example output:

```text
==================================================
        PYTHON SERVER HEALTH CHECKER
==================================================
Timestamp : 2026-10-10 14:00:00
Hostname  : ip-172-31-xx-xx
OS        : Linux
Uptime    : 120.50 hours
--------------------------------------------------
CPU Usage    : 12.0%
Memory Usage : 45.0%
Disk Usage   : 35.0%
--------------------------------------------------
OK: CPU usage is 12.0%
OK: Memory usage is 45.0%
OK: Disk usage is 35.0%
--------------------------------------------------
Top Processes by CPU:
PID: 1234    Name: python               CPU: 1.0% Memory: 2.0%
==================================================
Log saved to: /path/to/python-server-health-checker/server_health.log
```

**Note:** All values above are examples. Your actual hostname, uptime, resource usage, and processes will differ.

## 9. Step 6: Understand the Health Metrics

| Metric | What it measures |
|---|---|
| CPU usage | Percentage of CPU utilization during the sampling interval |
| Memory usage | Percentage of virtual memory currently used |
| Disk usage | Percentage of space used on the root filesystem |
| Uptime | Approximate hours since the system booted |
| Top processes | Processes with the highest reported CPU usage |

The script displays a warning when CPU, memory, or disk utilization reaches its configured threshold.

## 10. Step 7: View the Log File

Check whether the log file exists:

```bash
ls -l server_health.log
```

View the latest entries:

```bash
tail -n 10 server_health.log
```

Example output:

```text
2026-10-10 14:00:00 - INFO - Health check completed: host=server1 cpu=12.0% memory=45.0% disk=35.0% uptime=120.50 hours
```

Your actual log will contain measurements from your system.

## 11. Step 8: Change Warning Thresholds

Open the script:

```bash
vi server_health.py
```

At the beginning of the file, locate:

```python
CPU_WARNING = 80
MEMORY_WARNING = 80
DISK_WARNING = 80
```

For a test, change a threshold to a lower value. For example:

```python
CPU_WARNING = 5
```

Run the script again:

```bash
python server_health.py
```

If CPU usage is at least 5%, the script will display a CPU warning.

Restore the threshold to the desired production value after testing.

**Explanation:** Configurable thresholds help adapt the monitoring script to different workloads. A warning does not automatically mean the server is unhealthy; investigate trends and workload requirements.

## 12. Step 9: Run Python Syntax Checks

Compile the script to check for syntax errors:

```bash
python -m py_compile server_health.py
```

If successful, the command normally produces no output.

Check the script's permissions:

```bash
chmod +x server_health.py
ls -l server_health.py
```

Run it directly:

```bash
./server_health.py
```

The shebang at the top tells Linux to use Python 3.

## 13. Step 10: Create a Requirements File

Record the dependency:

```bash
python -m pip freeze > requirements.txt
```

Example content:

```text
psutil==<installed-version>
```

The actual version will be written into the file by the command.

Another computer can install the dependencies using:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

## 14. Step 11: Prepare the Project for GitHub

Create a `.gitignore` file:

```bash
cat > .gitignore <<'EOF'
.venv/
__pycache__/
*.pyc
server_health.log
.env
EOF
```

Check the project files:

```bash
ls -la
```

Expected files include:

```text
server_health.py
requirements.txt
README.md
.gitignore
```

The `.venv` directory and log file should not be committed for this basic project.

If you want to publish the project, add your README to the repository and commit the source files.

## 15. Step 12: Troubleshooting

### Problem A: ModuleNotFoundError: No module named 'psutil'

Activate your virtual environment and install the dependency:

```bash
source .venv/bin/activate
python -m pip install -r requirements.txt
```

### Problem B: Permission denied

Check that you are running the script as an authorized user and that the project directory is writable.

```bash
ls -l server_health.py
chmod +x server_health.py
```

### Problem C: Disk usage is high

Check filesystem utilization:

```bash
df -h
```

Identify large directories carefully:

```bash
du -sh /var/log 2>/dev/null
du -sh "$HOME" 2>/dev/null
```

Do not delete files until you understand their purpose and retention requirements.

### Problem D: A process cannot be inspected

Some processes belong to other users or require elevated permissions for detailed information. The script skips processes it cannot inspect rather than stopping the entire health check.

## 16. Limitations

- This is a basic, manually executed monitoring tool.
- CPU usage is sampled over a short interval.
- Disk usage covers the root filesystem only.
- A single measurement does not show long-term trends.
- It does not send email, SMS, or incident alerts.
- Production monitoring should include access controls, log rotation, persistent metrics, and suitable alerting.

## 17. Project Outcome

After completing this project, I can:

- Write and run a Python monitoring script on Linux.
- Collect CPU, memory, disk, uptime, and process information.
- Configure utilization thresholds.
- Generate health reports and logs.
- Manage Python dependencies with a virtual environment.
- Troubleshoot basic monitoring-script errors.
- Prepare a Python project for GitHub.

## 18. Tools and Technologies

- Amazon Linux 2023
- Python 3
- psutil
- Bash
- Git
- GitHub

## 19. Conclusion

The Python Server Health Checker demonstrates basic system monitoring and automation using Python. It provides a foundation for building more advanced monitoring tools for Linux servers and cloud environments.
