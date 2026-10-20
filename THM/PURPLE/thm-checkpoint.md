---
title: Checkpoint
summary: TryHackMe AI security assessment room reviewing four deployment candidates, identifying Candidate A's supply-chain failures, and making the production deployment call.
date: 2026-07-02
tags: [TryHackMe, AI Security, Supply Chain, Guardrails, Model Security, Purple Team]
difficulty: easy
os: N/A
url: https://tryhackme.com/room/checkpoint
---

# CTF Room: Checkpoint
- [Link to room](https://tryhackme.com/room/checkpoint)
- **Difficulty:** Easy
- **Category:** AI Security, Supply Chain, Deployment Review, Purple Team
- **OS:** N/A

## 1. Brief
Checkpoint is a TryHackMe AI security room built around a production decision. There are four candidate systems, and the job is to decide which one is safe enough to ship.

Candidate A is the problem child here. Its load session shows suspicious file access, one of its guardrails is disabled, and its agent points back to a risky review template.

## 2. Candidate A - File Access

#### Candidate A's load session shows a suspicious file access event. What file did it attempt to read?
I started with Candidate A's load session. The file access event jumped out because a model artefact or agent runtime should not be reaching for local system account files during a normal review workflow.

### Answer
```text
||/etc/passwd||
```

## 3. Candidate A - Guardrail State

#### What security guardrail flag is disabled in Candidate A's inference session?
Next I checked Candidate A's inference session. One security review flag was disabled, which explained why the candidate was allowed to behave outside the expected review boundary.

### Answer
```text
||security_review_flag||
```

## 4. Candidate A - Policy Template

#### Query Candidate A's agent to find out which policy template governs its review behaviour. What is the policy template?
The room then points you at the agent itself. Querying Candidate A gave the review policy template controlling the behaviour.

### Answer
```text
||CommunityReview||
```

## 5. Linking The Failures

#### Candidate A's two supply chain failures are not independent. Find what links them and use it to retrieve the flag.
The file access and disabled guardrail were not two random issues. They linked back through Candidate A's review behaviour and the same policy template path.

That common link was enough to retrieve the room flag.

### Answer
```text
||THM{supp1y_c***********||
```

## 6. Production Decision

#### Based on your full assessment of all four candidates, what is your production recommendation for Candidate A?
Candidate A has multiple supply-chain red flags:

- It attempted to read a sensitive local file.
- It had a security guardrail disabled.
- Its review behaviour tied back to a suspicious policy template.

That is not something I would approve for production.

### Answer
```text
||Reject||
```

#### Which candidate would you approve for production deployment?
After assessing all four candidates, Candidate B was the production-safe option.

### Answer
```text
||B||
```

## 7. Summary
Checkpoint is a production-readiness call, not just a hunt for a single flag. Candidate A fails because the technical findings reinforce each other: suspicious file access, weakened guardrails, and a risky policy template all point to the same supply-chain problem.

The clean production decision is to reject Candidate A and approve Candidate B.
