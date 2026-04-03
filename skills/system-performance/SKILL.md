---
name: system-performance
description: Use when analyzing system performance, debugging performance issues, investigating bottlenecks, or when the user mentions "performance analysis", "USE method", "flame graphs", "Brendan Gregg", or "system performance".
---

# System Performance Analysis

## Overview

System performance analysis is a methodology for investigating and solving performance problems in computing systems. This skill provides structured approaches based on Brendan Gregg's performance analysis methodologies, including the USE Method, RED Method, flame graphs, and other systematic debugging techniques.

## When to Use

Use this skill when:

- Investigating performance bottlenecks
- Debugging slow systems or applications
- Analyzing CPU, memory, disk, or network performance
- The user mentions "performance analysis", "USE method", "flame graphs", or "Brendan Gregg"
- Conducting performance testing or benchmarking
- Optimizing system or application performance

## Core Methodologies

### 1. USE Method (Utilization, Saturation, Errors)

**For every resource, check utilization, saturation, and errors.**

The USE Method is most effective for finding resource bottlenecks early in a performance investigation.

#### Resource Types

**Physical Resources:**

- CPUs: sockets, cores, hardware threads (virtual CPUs)
- Memory: capacity
- Network interfaces
- Storage devices: I/O, capacity
- Controllers: storage, network cards
- Interconnects: CPUs, memory, I/O

**Software Resources:**

- Mutex locks
- Thread pools
- Process/thread capacity
- File descriptor capacity

#### Metrics

| Resource         | Utilization                              | Saturation                            | Errors                           |
| ---------------- | ---------------------------------------- | ------------------------------------- | -------------------------------- |
| CPU              | CPU utilization (per-CPU or system-wide) | Run-queue length or scheduler latency | Correctable CPU cache ECC events |
| Memory           | Available free memory (system-wide)      | Anonymous paging or thread swapping   | Failed malloc()s                 |
| Network          | RX/TX throughput / max bandwidth         | Dropped packets, overruns             | Interface errors                 |
| Storage I/O      | Device busy percent                      | Wait queue length                     | Device errors                    |
| Storage Capacity | Used vs total capacity                   | Allocation failures                   | Filesystem errors                |

#### Interpretation Guidelines

**Utilization:**

- 100% utilization is usually a sign of a bottleneck
- High utilization (>70%) can begin to be a problem:
  - Long time periods can hide short bursts of 100%
  - Some resources (disks) cannot be interrupted during operations

**Saturation:**

- Any degree of saturation can be a problem (non-zero)
- Measured as queue length or time spent waiting

**Errors:**

- Non-zero error counters are worth investigating
- Especially if they are still increasing while performance is poor

#### USE Method Flowchart

```
Start
  ↓
For each resource:
  ↓
Check errors → If errors found → Investigate
  ↓
Check utilization → If high utilization → Check saturation
  ↓                    ↓
Check saturation → If saturation → Investigate
  ↓
Next resource
```

### 2. RED Method (Rate, Errors, Duration)

**For every service or microservice, check request rate, errors, and duration.**

The RED Method is designed for monitoring services and microservices, focusing on request-driven metrics.

#### Metrics

| Metric   | Description                  | Measurement               |
| -------- | ---------------------------- | ------------------------- |
| Rate     | Requests per second          | Counter over time window  |
| Errors   | Failed requests per second   | Error count / time        |
| Duration | Request latency distribution | Histogram (p50, p95, p99) |

#### Application

For each service:

1. **Rate**: Measure requests per second
2. **Errors**: Count failed requests
3. **Duration**: Track latency percentiles

### 3. TSA Method (Thread State Analysis)

**For each thread of interest, measure time in operating system thread states.**

#### Thread States

- **Executing**: Running on-CPU
- **Runnable**: Ready to run, waiting for CPU
- **Swapping**: Waiting for memory pages
- **Sleeping**: Waiting for an event
- **Lock**: Waiting for a lock
- **Idle**: Thread is idle

#### Process

1. Identify threads of interest
2. Measure time in each state
3. Investigate states from most to least frequent
4. Use appropriate tools for each state

### 4. Flame Graphs

**Visualize CPU usage with stack traces.**

Flame graphs show which code paths are consuming CPU time, making it easy to identify hot spots.

#### Types

- **CPU Flame Graphs**: On-CPU time
- **Off-CPU Flame Graphs**: Off-CPU time (waiting)
- **Memory Flame Graphs**: Memory allocations
- **I/O Flame Graphs**: I/O operations

#### Reading Flame Graphs

- **X-axis**: Stack profile population (not time)
- **Y-axis**: Stack depth
- **Width**: Proportion of samples in that stack
- **Color**: Can indicate different dimensions (process, thread, etc.)

#### Generation

```bash
# Linux perf (CPU)
perf record -g -p <PID>
perf script | stackcollapse-perf.pl | flamegraph.pl > cpu.svg

# Off-CPU
perf record -e sched:sched_switch -g -p <PID>
perf script | stackcollapse-perf.pl | flamegraph.pl > offcpu.svg

# BPF (modern Linux)
bpftrace -e 'profile:hz:99 @[ustack] = count();' > out.stacks
cat out.stacks | flamegraph.pl > cpu.svg
```

## Performance Analysis Workflow

### Problem Statement Method

Start by asking:

1. What makes you think there is a performance problem?
2. Has this system ever performed well?
3. What has changed recently? (Software? Hardware? Load?)
4. Can the performance degradation be expressed in terms of latency or run time?
5. Does the problem affect other people or applications (or is it just you)?
6. What is the environment? What software and hardware is used? Versions? Configuration?

### Workload Characterization Method

1. **Who** is causing the load? (PID, UID, IP addr, customer ID, geographic region, ...)
2. **Why** is the load called? (code path, stack trace, flame graph)
3. **What** is the load? (IOPS, throughput, type, url)
4. **How** is the load changing over time? (time series line graph)

### Drill-Down Analysis Method

1. Start at highest level
2. Examine next-level details
3. Pick most interesting breakdown
4. If problem unsolved, go to 2

### Process of Elimination

1. Divide the target into components
2. Choose a test which:
   - Can exonerate many untested components (ideally, half of those remaining)
   - Is quick to perform
3. Perform test
4. Were the tested components exonerated?
   - Yes: go to 2
   - No: problem found?
     - Yes: done
     - No: how many components were tested?
       - one: target = tested component; go to 1
       - multiple: go to 2 and choose a different test

## Anti-Methodologies (What NOT to Do)

### Blame-Someone-Else Anti-Method

1. Find a system or environment component you are not responsible for
2. Hypothesize that the issue is with that component
3. Redirect the issue to the responsible team
4. When proven wrong, go to 1

### Streetlight Anti-Method

1. Pick observability tools that are:
   - familiar
   - found on the Internet
   - found at random
2. Run tools
3. Look for obvious issues

### Drunk Man Anti-Method

1. Change things at random until the problem goes away

### Random Change Anti-Method

1. Measure a performance baseline
2. Pick a random attribute to change (eg, a tunable)
3. Change it in one direction
4. Measure performance
5. Change it in the other direction
6. Measure performance
7. Were the step 4 or 6 results better than the baseline? If so, keep the change; if not, revert
8. Goto step 1

### Passive Benchmarking Anti-Method

1. Pick a benchmark tool
2. Run it with a variety of options
3. Make a slide deck of the results
4. Hand the slides to management

### Traffic Light Anti-Method

1. Open dashboard
2. All green? Assume everything is good.
3. Something red? Assume that's a problem.

## Linux Performance Tools

### System-Wide

```bash
# CPU utilization
top, htop, mpstat

# Memory
free, vmstat

# Disk I/O
iostat, iotop

# Network
netstat, ss, ip, ifconfig

# All-in-one
dstat, vmstat, mpstat
```

### Advanced Tools

```bash
# Tracing
strace, ltrace

# Performance counters
perf

# BPF tools
bpftrace, bcc tools

# eBPF-based
bpftrace, bcc
```

### CPU Analysis

```bash
# CPU utilization
mpstat 1

# Per-process CPU
top -P

# CPU flame graph
perf record -g -a sleep 60
perf script | stackcollapse-perf.pl | flamegraph.pl > cpu.svg

# CPU scheduler latency
perf sched record -- sleep 60
perf sched latency

# Off-CPU analysis
offcputime -p <PID>
```

### Memory Analysis

```bash
# Memory utilization
free -m
vmstat 1

# Page faults
ps -o pid,comm,maj_flt,min_flt -p <PID>

# Memory leaks
valgrind --leak-check=full <program>

# Working set size
wss <PID>

# Memory flame graph
perf record -e page-faults -g -p <PID>
```

### Disk I/O Analysis

```bash
# Disk utilization
iostat -x 1

# Per-process I/O
iotop

# I/O latency
biolatency

# I/O flame graph
filetop -p <PID>
```

### Network Analysis

```bash
# Network utilization
sar -n DEV 1

# TCP connections
ss -s
netstat -s

# Network latency
ping, traceroute, mtr

# Packet capture
tcpdump, wireshark

# Network flame graph
perf record -e net:* -g -a
```

## Performance Mantras

1. **Don't do it** - The fastest code is no code
2. **Do it, but don't do it again** - Cache results
3. **Do it less** - Optimize algorithms
4. **Do it later** - Defer work to background
5. **Do it when they're not looking** - Batch operations
6. **Do it concurrently** - Parallelize work
7. **Do it cheaper** - Use cheaper resources

## Benchmarking Checklist

Before trusting benchmark results, ask:

1. **Why not double?** - If performance doesn't double with double resources, there's a bottleneck
2. **Did it break limits?** - Check for resource limits (CPU, memory, I/O)
3. **Did it error?** - Check for errors that may have been ignored
4. **Does it reproduce?** - Run multiple times to check consistency
5. **Does it matter?** - Is the difference statistically significant?
6. **Did it even happen?** - Verify the benchmark actually ran as expected

## Active Benchmarking

Instead of passive benchmarking (run and observe), use active benchmarking:

1. Configure the benchmark to run for a long duration
2. While running, analyze performance using other tools
3. Determine limiting factors
4. Understand why performance is what it is

## Common Performance Issues

### CPU Issues

**Symptoms:**

- High CPU utilization
- Slow response times
- Run queue saturation

**Investigation:**

```bash
# Check CPU utilization
mpstat 1

# Check run queue
vmstat 1

# Generate flame graph
perf record -g -a sleep 60
perf script | stackcollapse-perf.pl | flamegraph.pl > cpu.svg
```

**Common Causes:**

- Inefficient algorithms
- Excessive loops
- Unnecessary computations
- Lock contention

### Memory Issues

**Symptoms:**

- High memory utilization
- Page faults
- Out of memory errors
- Swap usage

**Investigation:**

```bash
# Check memory utilization
free -m
vmstat 1

# Check page faults
ps -o pid,comm,maj_flt,min_flt -p <PID>

# Check working set
wss <PID>
```

**Common Causes:**

- Memory leaks
- Large data structures
- Inefficient caching
- Memory fragmentation

### Disk I/O Issues

**Symptoms:**

- High disk utilization
- I/O wait times
- Slow file operations

**Investigation:**

```bash
# Check disk utilization
iostat -x 1

# Check I/O latency
biolatency

# Check per-process I/O
iotop
```

**Common Causes:**

- Random I/O patterns
- Small I/O sizes
- Synchronous I/O
- Disk fragmentation

### Network Issues

**Symptoms:**

- High network utilization
- Packet loss
- High latency
- Connection timeouts

**Investigation:**

```bash
# Check network utilization
sar -n DEV 1

# Check TCP statistics
ss -s
netstat -s

# Check latency
ping, traceroute
```

**Common Causes:**

- Network congestion
- Packet loss
- DNS issues
- Firewall rules

## Performance Analysis Steps

### Step 1: Problem Statement

Define the performance problem clearly:

- What is the symptom?
- When did it start?
- What changed?
- What is the impact?

### Step 2: USE Method

Apply USE Method to identify resource bottlenecks:

- Check CPU, memory, disk, network
- Look for high utilization, saturation, errors

### Step 3: Workload Characterization

Understand the workload:

- Who is causing the load?
- What type of load?
- How is it changing?

### Step 4: Drill-Down Analysis

Drill down into identified bottlenecks:

- Use appropriate tools
- Generate flame graphs
- Analyze stack traces

### Step 5: Root Cause Analysis

Identify the root cause:

- Use process of elimination
- Test hypotheses
- Verify fixes

## Quick Reference

| Method        | Use Case               | Key Metrics                     |
| ------------- | ---------------------- | ------------------------------- |
| USE Method    | Resource bottlenecks   | Utilization, Saturation, Errors |
| RED Method    | Services/microservices | Rate, Errors, Duration          |
| TSA Method    | Thread analysis        | Time in each state              |
| Flame Graphs  | CPU hot spots          | Stack traces                    |
| Workload Char | Understanding load     | Who, Why, What, How             |
| Drill-Down    | Deep analysis          | Hierarchical breakdown          |

## Tools Quick Reference

| Tool            | Purpose                |
| --------------- | ---------------------- |
| `top`, `htop`   | Process CPU/memory     |
| `mpstat`        | CPU statistics         |
| `vmstat`        | Virtual memory         |
| `iostat`        | Disk I/O               |
| `netstat`, `ss` | Network statistics     |
| `perf`          | Performance counters   |
| `bpftrace`      | BPF tracing            |
| `flamegraph.pl` | Flame graph generation |

## The Bottom Line

System performance analysis requires a systematic approach. Start with the USE Method to identify resource bottlenecks, use flame graphs to visualize CPU usage, apply workload characterization to understand the load, and drill down to find root causes. Avoid anti-methodologies like random changes or passive benchmarking. Always verify your findings and understand why performance is what it is.
