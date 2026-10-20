---
title: Payload
summary: TryHackMe supply-chain incident room investigating a backdoored ML model, outbound beaconing, and a staged candidate model containing the second half of the campaign ID.
date: 2026-07-01
tags: [TryHackMe, Supply Chain, Machine Learning, Pickle, H5, Incident Response, Forensics]
difficulty: medium
os: Linux
url: https://tryhackme.com/room/payload
---

# CTF Room: Payload
- [Link to room](https://tryhackme.com/room/payload)
- **Difficulty:** Medium
- **Category:** Supply Chain, ML Model Analysis, Incident Response
- **OS:** Linux

## 1. Brief
Payload is a TryHackMe incident response room focused on a compromised ML inference server.

The incident materials are located at:

```bash
/opt/supply-chain/incident/
```

The investigation starts with the logs, then moves into the production model, and finally checks the staged replacement model before it can be deployed.

## 2. Lab Access
If using SSH instead of the split-screen VM, the provided credentials are:

### SSH
```bash
ssh analyst@MACHINE_IP
```

### Credentials
```text
Username: analyst
Password: ||analyst123||
```

## 3. Incident Timeline

#### Read the deployment log. The replacement model came from a different organisation than the original. What is the name of that organisation?
I started with the deployment log.

### Command
```bash
cat /opt/supply-chain/incident/logs/deployment.log
```

The replacement model came from a different organisation than the original model.

### Answer
```text
trustworthy-ai-lab
```

#### How many days passed between the replacement model being deployed and the SOC alert firing?
The deployment occurred on `2024-01-26` and the SOC alert fired on `2024-02-16`.

### Calculation
```text
2024-01-26 -> 2024-02-16 = 21 days
```

### Answer
```text
21
```

## 4. Production Model Analysis

#### Decompile the production model. What Python function does the payload use to execute the shell command?
I used Fickling to decompile the production model and inspect the embedded payload.

### Command
```bash
fickling /opt/supply-chain/incident/models/production_model.pkl
```

The payload used Python's `system` function to execute a shell command.

### Answer
```text
system
```

#### What shell command does the payload use to capture the host's identity?
The command executed by the payload captured the host identity.

### Answer
```text
hostname
```

## 5. Beacon Capture

#### The beacon capture log shows the HTTP method used in the outbound request. What is it?
The outbound beacon was recorded in `beacon_capture.log`.

### Command
```bash
cat /opt/supply-chain/incident/logs/beacon_capture.log
```

### Evidence
```text
[2024-02-16 03:13:47] SESSION beacon-4821 ESTABLISHED  src=10.0.1.50  dst=attacker.com:443
[2024-02-16 03:13:47] REQUEST POST /beacon HTTP/1.1
[2024-02-16 03:13:47] HOST attacker.com
[2024-02-16 03:13:47] PAYLOAD host=ml-server-prod-01&id=||****************||
[2024-02-16 03:13:48] SESSION beacon-4821 BLOCKED  bytes_captured=51  reason=SOC_RULE_4821
```

### Answer
```text
POST
```

## 6. Candidate Model Analysis

#### The engineering team staged `candidate_model.h5` as a replacement but have not yet deployed it. What is the name of the suspicious layer it contains?
I inspected the candidate H5 model with the provided helper script.

### Command
```bash
inspect_h5_model.py /opt/supply-chain/incident/models/candidate_model.h5
```

The suspicious layer was named `manipulate_output`.

### Answer
```text
manipulate_output
```

#### The attacker split the campaign ID across two artefacts to avoid full exposure in any single capture. Examine `beacon_capture.log` and the candidate model to recover the complete flag.
The first half of the campaign ID appeared in `beacon_capture.log`. The second half was embedded in the staged candidate model and appeared when inspecting `candidate_model.h5`.

Combining both artefacts recovered the complete flag.

### Answer
```text
||THM{b4ckd00r_1**************||
```

## 7. Summary
The deployment log showed that the replacement model came from `trustworthy-ai-lab`, and the SOC alert fired 21 days later.

The production model contained a payload that used `system` to run `hostname` and beacon the host identity over a `POST` request. The staged H5 replacement also contained a suspicious `manipulate_output` layer, which held the missing portion of the campaign ID.
