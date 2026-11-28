# Initial System Architecture

## 1. Purpose
This document describes the planned architecture of the Distributed Security Log Analyzer (DSLA). It is an initial design: components marked as planned must not be represented as completed until implemented and tested.

## 2. High-level architecture

```text
                +----------------------+
                |  CSV Security Logs   |
                | auth_sample.csv      |
                +----------+-----------+
                           |
                           v
                +----------------------+
                | Log Reader / Loader  |
                +----------+-----------+
                           |
                           v
                +----------------------+
                | Parser & Validator   |
                | timestamp, IP, user, |
                | status               |
                +----------+-----------+
                           |
                           v
                +----------------------+
                | Analysis Coordinator |
                | chooses execution    |
                | strategy              |
                +----------+-----------+
                           |
          +----------------+-------------------+
          |                |                   |
          v                v                   v
   +-------------+  +-------------+    +----------------+
   | Sequential  |  | Java CPU    |    | Distributed    |
   | baseline    |  | parallelism |    | execution      |
   +------+------+  +------+------+    +-------+--------+
          |                |                   |
          |        Threads / ForkJoin    Sockets / MPI
          |                                    |
          +----------------+-------------------+
                           |
                           v
                +----------------------+
                | Detection Engine     |
                | shared rule logic    |
                +----------+-----------+
                           |
                           v
                +----------------------+
                | Result Aggregator    |
                | merge worker results|
                +----------+-----------+
                           |
                           v
                +----------------------+
                | Report & Metrics     |
                | findings, counts,    |
                | execution time       |
                +----------------------+

       CUDA acceleration: planned optional compute component
       for suitable scoring/analysis operations.
```

## 3. Main components

| Component | Responsibility |
|---|---|
| CSV Log Input | Supplies authentication events in the agreed four-column format. |
| Log Reader | Opens the input file and reads records safely. |
| Parser and Validator | Converts CSV rows into event objects and rejects malformed records. |
| Analysis Coordinator | Selects the execution mode and divides work where required. |
| Sequential Analyzer | Provides the baseline result and execution time. |
| Parallel Workers | Analyze independent chunks using Java threads or Fork/Join. |
| Socket Coordinator/Workers | Exchange work and results between Java processes over sockets. |
| MPI Engine | Planned process-level distributed analysis using MPI operations. |
| CUDA Component | Planned GPU acceleration for suitable computational work. |
| Detection Engine | Applies common suspicious-login rules consistently. |
| Result Aggregator | Combines worker/process findings without losing or double-counting events. |
| Report and Metrics | Produces a readable summary and performance measurements. |

## 4. Data model
Each input row represents one authentication event:

- `timestamp`: date and time of the event (`yyyy-MM-dd HH:mm:ss`)
- `ip`: source IP address
- `username`: account name used in the attempt
- `status`: `SUCCESS` or `FAILED`

The parser should normalize and validate these fields before analysis. Invalid timestamps, missing fields, empty IP/user values, and unknown status values should be counted or reported as invalid input rather than silently treated as valid events.

## 5. Processing flow
1. Load the input CSV and skip its header.
2. Parse and validate every record.
3. Select the requested analysis mode.
4. Process events sequentially or divide them into chunks for parallel/distributed execution.
5. Apply the same detection criteria in every mode.
6. Merge findings at the coordinator/aggregator.
7. Generate a report containing totals, suspicious IPs/accounts, and processing time.
8. Compare modes using the same dataset and detection configuration.

## 6. Correctness and concurrency considerations
- All modes must use the same input and detection thresholds for a fair comparison.
- Chunk boundaries must not cause time-window events to be missed. The 60-second rule needs boundary overlap or a final aggregation strategy.
- Shared counters/maps must use thread-safe updates or local aggregation followed by merging.
- Aggregation must avoid duplicate alerts when the same event is observed by overlapping chunks.
- Record processing time consistently; exclude unrelated setup time only if the measurement policy is documented.
- Handle malformed input and worker failures without silently producing incomplete results.

## 7. Planned technology mapping
- Java: application, parsing, sequential baseline, threads, Fork/Join, sockets, reporting.
- MPI: process-level distributed computation, as supported by the selected MPI environment.
- CUDA: GPU-side acceleration for a clearly defined suitable operation; data transfer overhead must be included in performance evaluation.
- GitHub: version control, individual branches, pull requests, and contribution evidence.

## 8. Current status
This is the initial architecture for the project. It documents the intended design, not proof that every component is already implemented. Update this document when interfaces, module names, data flow, or technology choices change.
