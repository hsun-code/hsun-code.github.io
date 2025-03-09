# Check CPU Usage with Sar and Visualize the results with SARChart

**Motivation**: When analyzing one workload, we may want to figure out the CPU
usage during the execution and collect the profiling information **only** for the
period with high or 100% CPU usage load.

There are many options to check CPU usage, e.g., [How to Check CPU Utilization in Linux with Command Line].
In this post we use [Sar] utility to analyze one Java workload [Renaissance] and
visualize the results with [SARChart].

## Sar

**Installation**:

```bash
# install the package
$ sudo apt-get install -y sysstat
# start the service. on host
$ systemctl start sysstat.service
# check the version
$ sar -V
sysstat version 12.5.2
(C) Sebastien Godard (sysstat <at> orange.fr)
```

**Basic usage**:

```bash
# Report CPU details for a total 60 times with the interval fo 1 second
sar -u 1 60 > /tmp/sar-res.txt
#
sar -P ALL 1 1200 > /tmp/aaa.txt # 20 min. each cpu per line
```

## Java Renaissance workload

## SARChart

<!-- Links -->
[How to Check CPU Utilization in Linux with Command Line]: https://phoenixnap.com/kb/check-cpu-usage-load-linux
[Sar]: https://www.geeksforgeeks.org/sar-command-linux-monitor-system-performance/
[SARChart]: https://sarchart.dotsuresh.com/
[Renaissance]: https://renaissance.dev/
