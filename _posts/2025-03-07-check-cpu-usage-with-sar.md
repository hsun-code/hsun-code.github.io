---
title: "Check CPU Usage with Sar and Visualize the results with SARChart"
tags: Performance
---

**Motivation**: When analyzing one workload, we may want to figure out the CPU
usage during the execution and collect the profiling information **only** for the
period with high or 100% CPU usage load.

There are many options to check CPU usage, e.g., [How to Check CPU Utilization in Linux with Command Line].
In this post we use [Sar] utility to analyze one Java workload [Renaissance] and
visualize the results with [SARChart].

## Sar

**Installation**:

it's a cpp

```cpp
#include <stdio.h>
int main(int a, int b) {
  return a + b;
}
```

it's a java

```java
class Test {
    public static void main(String args[]) {
        System.out.println("Hello Java");  
    }
}
```

it's python

```python
x = 1
if x == 1:
  # indented four spaces
  print("x is 1.")
```

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

```shell
# Report CPU details for a total 10 times with the interval fo 1 second
# Save the report records into a file
$ sar -u 1 10 > /tmp/sar-res.txt

# Check the output
$ cat /tmp/sar-res.txt

12:43:17 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
12:43:18 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
12:43:19 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
12:43:20 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
12:43:21 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
12:43:22 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
12:43:23 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
12:43:24 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
12:43:25 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
12:43:26 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
12:43:27 AM     all      0.00      0.00      0.00      0.00      0.00    100.00
Average:        all      0.00      0.00      0.00      0.00      0.00    100.00

# Each report record contains one line for each CPU
$ sar -P ALL 1 10 > /tmp/sar-res.txt
```

## Java Renaissance workload

**Environment**: We evaluate [Renaissance] workload, the JMH version 0.16, with
OpenJDK 21 version. We select `AkkaUct` benchmark as an example and run it with
arguments `-f 3 -i 10 -wi 5`, that is, using 3 forks with 5 warmup iterations
and 10 measurement iterations. The benchmark takes ~1m20s to finish.

**Evaluation**:

Open one terminal and run `AkkaUct`.

```bash
$ java -jar renaissance-jmh-0.16.0.jar -f 3 -i 10 -wi 5 AkkaUct

...
...

Benchmark       Mode  Cnt     Score    Error  Units
JmhAkkaUct.run    ss   30  1701.498 ± 48.076  ms/op
```

At the same time, we run `sar` in one separate terminal and
the CPU usage information wil be recorded in file `/tmp/sar-res-akkauct.txt`.

```bash
# One trick: we may 'ctrl-c' to termine sar if we saw the benchmark has finished.
$ sar -u 1 90 > /tmp/sar-res-akkauct.txt

# Here shows the first 10 lines of the sar result
$ cat /tmp/sar-res-akkauct.txt | head -10

01:13:37 AM     CPU     %user     %nice   %system   %iowait    %steal     %idle
01:13:38 AM     all     84.57      0.00      0.67      0.00      0.00     14.76
01:13:39 AM     all     88.59      0.00      0.86      0.00      0.00     10.55
01:13:40 AM     all     97.75      0.00      0.39      0.00      0.00      1.86
01:13:41 AM     all     82.95      0.00      0.41      0.00      0.00     16.63
01:13:42 AM     all     66.51      0.00      0.39      0.00      0.00     33.10
01:13:43 AM     all     98.15      0.00      0.44      0.00      0.00      1.40
01:13:44 AM     all     73.23      0.00      0.71      0.00      0.00     26.06
```

## SARChart

We upload the `Sar` records to [SARChart] and get the following chart to display
the CPU utilization at the **user level(application)**.

![Sar-CPU-AkkaUct-3](/images/sar-cpu.svg)

### Full result

Full result can be found in [Renaissance-Sar result](/files/202503-renaissace-sar-result)

<!-- Links -->
[How to Check CPU Utilization in Linux with Command Line]: https://phoenixnap.com/kb/check-cpu-usage-load-linux
[Sar]: https://www.geeksforgeeks.org/sar-command-linux-monitor-system-performance/
[SARChart]: https://sarchart.dotsuresh.com/
[Renaissance]: https://renaissance.dev/
