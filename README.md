# Distributed Security Log Analyzer (DSLA)

## Project overview
The Distributed Security Log Analyzer is a Java-based Parallel and Distributed Computing (PDC) lab project. It reads authentication/security logs, identifies suspicious login activity, and produces a summarized security report. The project is designed to demonstrate and compare sequential, multithreaded, Fork/Join, socket-based, MPI, and CUDA-based processing approaches as these phases are implemented.

## Team
- Muhammad Arqam
- Mahnoor Fatima

## Current input format
The sample dataset is `datasets/sample/auth_sample.csv`.

It is a CSV file with exactly these columns:

| Column | Meaning | Example |
|---|---|---|
| `timestamp` | Event time in `YYYY-MM-DD HH:MM:SS` | `2026-11-20 10:00:08` |
| `ip` | Source IP address | `203.0.113.50` |
| `username` | Account involved in the login attempt | `admin` |
| `status` | Authentication result: `SUCCESS` or `FAILED` | `FAILED` |

Each row represents one login event. The sample contains ordinary activity and deliberately included failed-login patterns for testing. IP ranges such as `192.0.2.0/24`, `198.51.100.0/24`, and `203.0.113.0/24` are documentation ranges and are not intended to identify real attackers.

## Initial detection rules
1. **Repeated failures by source IP:** flag an IP when it produces more than four failed login attempts.
2. **Repeated failures against an account:** flag a username when it receives more than four failed login attempts.
3. **Time-window brute-force target:** flag an IP/username pair when it records at least five failed attempts within 60 seconds. This is the planned time-aware rule; implementation details should be kept consistent across processing modes.
4. **Successful login after repeated failures (future enhancement):** report a successful login that follows a suspicious failure burst, but do not treat this as implemented until the code supports it.

The initial threshold rules are intended for a lab demonstration, not as a production security policy. A real deployment would need configurable thresholds, trusted-source handling, and false-positive review.

## Repository layout

```text
.
├── README.md
├── datasets/
│   └── sample/
│       └── auth_sample.csv
├── docs/
│   ├── architecture.md
│   └── objectives-and-detection-rules.md
├── java/
│   └── src/main/java/com/dsla/
├── tests/
├── mpi/
├── cuda/
├── scripts/
└── results/
```

Some directories are planned locations and may be added as their implementation phases begin.

## High-level processing workflow
1. Read the CSV input.
2. Validate and parse each row into a normalized login-event record.
3. Analyze events using the selected processing approach.
4. Detect suspicious patterns using the shared detection rules.
5. Aggregate findings from workers/processes.
6. Generate a report and record execution time for performance comparison.

## Planned PDC approaches
- Sequential Java baseline
- Java multithreading and synchronization
- Fork/Join framework
- Java socket-based coordinator/worker processing
- MPI-based distributed processing
- CUDA-based acceleration for suitable scoring/analysis work

The project should report only the approaches that have actually been implemented and tested. Do not claim MPI or CUDA execution from the Java application until the integration is complete.

## Running the sample
The dataset is documentation/input material at this stage. Once the Java entry point and build instructions are finalized, add the exact compile/run commands here. Do not use placeholder commands as if they were tested.

## Team contribution workflow
Each member must work on their own branch and push their assigned commits. Mahnoor's assigned foundation branch is:

```bash
git checkout -b mahnoor/phase-1-foundation
```

After committing and pushing, open a Pull Request targeting `main`. Do not commit directly to the protected `main` branch.

## Dataset notes
- Header row is included.
- Timestamps use a consistent 24-hour format.
- Status values are uppercase `SUCCESS` or `FAILED`.
- The dataset is synthetic and safe for coursework.
- Keep generated/large datasets separate from the small sample dataset.
