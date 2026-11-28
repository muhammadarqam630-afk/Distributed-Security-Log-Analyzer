# Project Objectives and Detection Rules

## 1. Project objectives
The Distributed Security Log Analyzer (DSLA) aims to:

1. Read structured authentication logs from CSV files.
2. Parse and validate login events consistently.
3. Identify suspicious repeated failed-login activity.
4. Produce understandable summaries of suspicious IP addresses and targeted usernames.
5. Implement a sequential baseline for correctness and timing.
6. Apply parallel and distributed computing techniques to analyze larger datasets.
7. Compare execution time and results across sequential, multithreaded, Fork/Join, socket, MPI, and CUDA approaches as each is implemented.
8. Demonstrate correct synchronization, work distribution, aggregation, and error handling.
9. Maintain reproducible sample/generated datasets and document how each experiment was performed.

## 2. Input event schema

| Field | Required | Description |
|---|---|---|
| `timestamp` | Yes | Event timestamp in `yyyy-MM-dd HH:mm:ss` format |
| `ip` | Yes | Source IP address |
| `username` | Yes | Account name attempted |
| `status` | Yes | `SUCCESS` or `FAILED` |

Only `FAILED` events count toward failed-login thresholds. `SUCCESS` events are retained for reporting and possible future correlation.

## 3. Detection rules

### Rule 1 — Repeated failed attempts from one IP
- Group events by source IP.
- Count events whose status is `FAILED`.
- Flag the IP when the count is greater than 4 (5 or more).
- Suggested alert: `SUSPICIOUS_IP: repeated failed logins`.

### Rule 2 — Repeated failed attempts against one username
- Group events by username.
- Count events whose status is `FAILED`.
- Flag the username when the count is greater than 4 (5 or more).
- Suggested alert: `TARGETED_ACCOUNT: repeated failed logins`.

### Rule 3 — Brute-force burst within a time window
- Group failed events by the combination of IP and username.
- Sort each group by timestamp.
- Flag the pair when at least 5 failed events occur within any rolling 60-second interval.
- Suggested alert: `BRUTE_FORCE_BURST: 5+ failures within 60 seconds`.

Rule 3 is a time-aware target. It must be implemented using timestamps; a total count alone is not sufficient to prove that attempts occurred within 60 seconds.

### Rule 4 — Successful login after a suspicious burst (future enhancement)
- Correlate a successful event with a preceding suspicious failure burst for the same IP/account context.
- Report it as a separate correlation finding.
- This is a future enhancement and is not part of the initial required threshold rules unless implemented and tested.

## 4. Important interpretation
The simple total-count rules (Rules 1 and 2) can flag activity spread over a long period. Rule 3 specifically detects a concentrated burst. The report should identify which rule triggered, rather than displaying all alerts as if they meant the same thing.

These thresholds are educational defaults for a PDC lab demonstration. They are not universal indicators of malicious behavior. Real systems require environment-specific thresholds, allowlists, and analyst review.

## 5. Expected output
A basic report should include:
- Total rows read
- Valid events
- Invalid/skipped rows
- Total successful attempts
- Total failed attempts
- Suspicious IP addresses and their failed-attempt counts
- Targeted usernames and their failed-attempt counts
- IP/username pairs that triggered the 60-second burst rule
- Execution mode and processing time

## 6. Testing expectations
At minimum, verify:
1. Four failures for an IP do not trigger the `greater than 4` rule.
2. Five failures for an IP do trigger the rule.
3. Five failures for a username do trigger the account rule.
4. Five failures within 60 seconds for one IP/username pair trigger Rule 3.
5. Five failures spread over more than 60 seconds do not trigger Rule 3 solely on the basis of that time-window rule.
6. Successful events do not increase failed-attempt counts.
7. Invalid rows do not crash the full analysis.
8. Sequential and parallel modes produce equivalent findings for the same input.

## 7. Dataset design
The sample dataset includes ordinary successful/failed logins and synthetic suspicious patterns:
- A single source IP making repeated failed attempts against `admin`.
- Multiple source IPs attempting the `root` account.
- One source IP attempting multiple usernames.
- Ordinary activity from internal/private example addresses.

The dataset uses documentation-only public IP ranges for suspicious examples. It is synthetic and must not be interpreted as real threat intelligence.
