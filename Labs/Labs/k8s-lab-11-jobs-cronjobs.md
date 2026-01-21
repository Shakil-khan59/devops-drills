# Kubernetes Lab 11: Jobs and CronJobs

This lab covers batch workloads, retries, and scheduling.

---

## Table of Contents
1. Prerequisites and Setup
2. Core Concepts
3. Lab 1: One-Time Job
4. Lab 2: Parallelism and Completions
5. Lab 3: CronJob Scheduling
6. Lab 4: TTL and Backoff Limits
7. Troubleshooting Scenarios
8. Interview Questions and Answers
9. Cleanup
10. Theory (Concise)

---

## 1. Prerequisites and Setup
```bash
kubectl create ns job-lab
kubectl config set-context --current --namespace=job-lab
```

---

## 2. Core Concepts
- **Job:** Runs pods to completion
- **CronJob:** Schedules Jobs on a cron expression
- **BackoffLimit:** Retry cap for failures

---

## 3. Lab 1: One-Time Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-job
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: app
        image: busybox:1.36
        command: ["/bin/sh", "-c", "echo hello; sleep 2"]
```

### Apply
```bash
kubectl apply -f - <<'YAML'
<PASTE THE YAML ABOVE>
YAML
kubectl get jobs
```

### Solution check
Job completes with `1/1` successful pods.

---

## 4. Lab 2: Parallelism and Completions

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: parallel-job
spec:
  completions: 6
  parallelism: 2
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: app
        image: busybox:1.36
        command: ["/bin/sh", "-c", "echo work; sleep 3"]
```

### Solution check
Two pods run concurrently until six complete.

---

## 5. Lab 3: CronJob Scheduling

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello-cron
spec:
  schedule: "*/2 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: app
            image: busybox:1.36
            command: ["/bin/sh", "-c", "date; echo cron run"]
```

### Solution check
A new Job appears every 2 minutes.

---

## 6. Lab 4: TTL and Backoff Limits

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: fail-fast
spec:
  backoffLimit: 1
  ttlSecondsAfterFinished: 60
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: app
        image: busybox:1.36
        command: ["/bin/sh", "-c", "exit 1"]
```

### Solution check
Job retries once, then stops and is cleaned up after 60s.

---

## 7. Troubleshooting Scenarios
- **Jobs stuck:** Check pod logs and `backoffLimit`.
- **CronJob not running:** Verify schedule and controller status.
- **Too many jobs:** Set `successfulJobsHistoryLimit`.

---

## 8. Interview Questions and Answers

### Q1: Job vs CronJob?
**Answer:** Job runs once; CronJob schedules Jobs repeatedly.

### Q2: What does backoffLimit do?
**Answer:** Limits retries for failed pods in a Job.

### Q3: How do you clean up old Jobs?
**Answer:** Use TTL or history limits.

---

## 9. Cleanup
```bash
kubectl delete ns job-lab
```

---

## 10. Theory (Concise)
- **Jobs guarantee completion**, not availability.
- **CronJobs schedule Jobs** using cron expressions.
- **Backoff and TTL** control retry and retention behavior.
