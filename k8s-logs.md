Kubernetes log troubleshooting for $ARGUMENTS (use: pod name or keyword, e.g. "my-service").

Follow these steps in order:

## 0. Confirm the active cluster context (ALWAYS do this first)

```bash
kubectl config current-context
kubectl config view --minify | grep server
```

Show the output to the user and **explicitly ask**: "You are connected to `<context-name>` (`<server-url>`). Is this QA or PROD? Do you want to continue?"

Do NOT proceed until the user confirms.

## 1. Find the pod

```bash
kubectl get pods -A | grep $ARGUMENTS
```

Note the **namespace** and **full pod name** from the output.

## 2. Pull recent logs

```bash
kubectl logs -n <namespace> <pod-name> --tail=200
```

## 3. Filter for errors and warnings

```bash
kubectl logs -n <namespace> <pod-name> --tail=500 2>&1 | grep -iE "error|warn|hangup|hung|disconnect|failed|exception|timeout|crash"
```

## 4. Check worker capacity issues

```bash
kubectl logs -n <namespace> <pod-name> --tail=500 2>&1 | grep -iE "capacity|busy|queue|dispatch|worker|accept|spawn"
```

## 5. Check replica count and scaling

```bash
kubectl get pods -n <namespace> | grep <pod-keyword>
kubectl get deployment <deployment-name> -n <namespace>
kubectl get hpa -n <namespace>
```

If only 1 replica is running and capacity warnings appear, scale up:

```bash
kubectl scale deployment <deployment-name> -n <namespace> --replicas=3
```

To revert:

```bash
kubectl scale deployment <deployment-name> -n <namespace> --replicas=1
```

## 6. Check Kubernetes events for the namespace

```bash
kubectl get events -n <namespace> --sort-by='.lastTimestamp' | tail -30
```

## 7. Search logs for a specific phone number or caller ID

```bash
kubectl logs -n <namespace> <pod-name> 2>&1 | grep -i "<phone-number>"
```

If the number does NOT appear in any pod logs, the call is not reaching the bot — the problem is upstream (SIP trunk, LiveKit dispatch rule, or telephony routing).

## 8. Check startup and registration

```bash
kubectl logs -n <namespace> <pod-name> 2>&1 | head -80
```

Look for:
- `registered worker` — confirms the bot connected to LiveKit successfully
- `agent_name` — confirms which agent name is registered
- `url` — confirms which LiveKit cloud URL it's registered to

## Common findings and their meaning

| Finding | Likely cause |
|---|---|
| `worker is at full capacity` | Not enough replicas — scale up |
| Phone number not in any log | Call never reached the bot — check LiveKit SIP dispatch rules |
| `process did not exit in time` | Hanging coroutine on teardown — check cleanup code |
| `speech not done in time after interruption` | Race condition during warm transfer |
| `memory_usage_mb` consistently above warn threshold | Possible memory leak — monitor over time |
| `LLM node interception failed` | LLM error mid-call — check Azure OpenAI quotas/errors |
