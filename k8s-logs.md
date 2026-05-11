---
model: claude-sonnet-4-6
effort: medium
---

Kubernetes deployment health check for `$ARGUMENTS`.

Usage: `/k8s-logs <deployment-shortname> [env]`

Examples:
- `/k8s-logs col-casey qa` — check `col-casey-bot` in QA
- `/k8s-logs mlf prod` — check `mlf-bot` in PROD
- `/k8s-logs apaibackend` — env not supplied, ask the user

---

## Environment → kubectl context

| Env keyword (case-insensitive) | kubectl context |
|---|---|
| `qa`, `staging`, `stg` | `tranzact-k8s` |
| `prod`, `production`, `prd` | `tranzact-prod-k8s` |

---

## Step 0. Parse arguments

`$ARGUMENTS` is a free-form string. Split into tokens.

- **Token 1** = deployment shortname (required). Examples: `col-casey`, `mlf`, `apaibackend`. This is a substring used to grep pods — exact deployment names not required.
- **Token 2** = env keyword (optional). Examples: `qa`, `prod`.

**If env was NOT supplied**, ask the user with `AskUserQuestion`:
- Question: "Which environment should I check `<shortname>` in?"
- Options: `QA (tranzact-k8s)` and `PROD (tranzact-prod-k8s)`.

**If env WAS supplied**, do NOT ask for confirmation. Proceed directly. (If env resolves to PROD, prefix your response with "⚠️ Running against PROD" once so the user is aware.)

## Step 1. Switch context

```bash
kubectl config use-context <resolved-context>
kubectl config current-context
```

## Step 2. Find the pod(s)

Use the Bash tool — `grep` is available in Git Bash on Windows and natively on macOS/Linux. Do **not** use `findstr` (its `/I` flag is misread as a path under Git Bash).

```bash
kubectl get pods -A | grep <shortname>
```

Capture: namespace, full pod name(s), READY column, restart count, age. If multiple pods match unrelated deployments, list them and ask. If multiple replicas of the same deployment match, pick the newest unless the user asked otherwise.

## Step 3. Gather data in parallel

Run these in a single message with multiple Bash tool calls:

```bash
# Deployment & HPA
kubectl get deployment -n <ns> <deployment-name>
kubectl get hpa -n <ns>

# Pod details — narrow describe output to the rows that matter
kubectl describe pod <pod> -n <ns> | grep -E "Status:|Restart Count|Ready:|Image:|Started:|Reason:|Last State|QoS"

# Resource usage
kubectl top pod -n <ns> <pod>

# Recent log tail
kubectl logs -n <ns> <pod> --tail=80

# Error/warning scan over wider window
kubectl logs -n <ns> <pod> --tail=500 2>&1 | grep -iE "error|warn|hangup|hung|disconnect|failed|exception|timeout|crash|capacity|oom|killed" | tail -40

# Recent events
kubectl get events -n <ns> --sort-by='.lastTimestamp' | tail -20
```

## Step 4. Render the standard summary

ALWAYS use this exact format. Fill every row from real output — never guess. If a value is unavailable, write `n/a` and explain why in Notes.

```markdown
## <DeploymentName> Health Check — <ENV> (`<context>`)

**Pod:** `<full-pod-name>` (namespace `<ns>`)

### Status: <verdict>

| Check | Result |
|---|---|
| Phase | `<Running/Pending/CrashLoopBackOff>`, `Ready: <True/False>` |
| Restart count | <n> |
| Image | `<image:tag>` |
| Started | <timestamp> (~<age>) |
| App registration | <✅/❌ + key startup line, e.g. LiveKit `registered worker`, server `listening on :8080`> |
| Errors / warnings in last 500 log lines | <None / count + brief categorization> |
| Resource usage | CPU <m>, Memory <Mi> |
| Deployment / HPA | <ready>/<desired> Available · HPA `<name>`: min <x> / max <y> / current <z>, target <metric> (currently <val>) |

### Notes
- <recent rollouts, transient warnings explained, or "no notable events">
- <suggested next step or "no action needed">
```

**Verdict rules:**
- ✅ Healthy — pod Ready, 0 recent restarts, no error/warn matches, HPA in steady state
- ⚠️ Degraded — pod Ready but with warnings (recent restarts, log errors, HPA flapping, high memory)
- ❌ Failing — pod not Ready, CrashLoopBackOff, image pull errors, OOMKilled, or worker not registered

## Step 5. If degraded or failing, add a diagnosis block

Don't just dump errors. For each distinct issue, present this block. Aim for actionable — the reader could be a junior dev seeing this for the first time:

````markdown
### 🔍 Issue: <one-line title>

**What I saw:**
```
<3–10 line log/event excerpt — exact text, no paraphrase>
```

**Likely root cause:** <plain-language explanation. Reference the specific signal that points to this cause. If multiple causes are plausible, list top 2 with the deciding signal.>

**Suggested next steps:**
1. <concrete action — kubectl command, code area to inspect, or external system to check>
2. <next action if step 1 doesn't resolve it>

**Dig deeper:** <command for more context, e.g. `kubectl logs ... --previous`, `kubectl describe`, dashboard URL>
````

Common signal → diagnosis (starter set; expand based on what you actually see):

| Signal in logs/events | Likely root cause | First fix to try |
|---|---|---|
| `CrashLoopBackOff` + `OOMKilled` in `Last State` | Memory limit too low or memory leak | Bump `resources.limits.memory`; if leak, look at recent commits touching long-lived caches/coroutines |
| `ImagePullBackOff` / `ErrImagePull` | Tag doesn't exist or registry auth failed | Verify tag in GHCR; check `imagePullSecrets` |
| `worker is at full capacity` (LiveKit agents) | Replicas can't keep up with concurrent calls | Raise HPA `maxReplicas` or increase per-pod concurrency |
| `process did not exit in time` | Async cleanup hanging on shutdown | Look for awaits without timeout in teardown / `aclose()` paths |
| `LLM node interception failed` / 429s | Upstream LLM rate limit or quota | Check Azure OpenAI dashboard; review retry/backoff config |
| `FailedGetResourceMetric` *only right after rollout* | metrics-server hasn't scraped new pod yet | Transient — ignore unless it persists >2 min |
| `Liveness probe failed` repeating | App not responding to health endpoint | Check probe path/port matches app; check app startup time vs `initialDelaySeconds` |
| Phone number / request ID absent from logs | Traffic never reached this pod | Look upstream: SIP trunk, LiveKit dispatch rule, ingress |
| Pod just rolled but user didn't deploy | Someone else deployed, or HPA scaled, or node pressure evicted it | Check `kubectl describe` for eviction reason; check CI/CD history |

## Step 6. Safety rules

- **Never run `kubectl scale`, `delete`, `rollout restart`, or any write operation on PROD without explicit user approval in the current turn.** On QA, only do it if the user asked for it — don't auto-scale just because you saw a capacity warning, surface it as a suggestion.
- **Never run `kubectl exec`** unless the user asked. It can mutate pod state.
- If the deployment shortname matches multiple unrelated deployments, list them and ask before picking one.

## Step 7. End with one follow-up offer

One short line offering the next likely action — e.g. "Want me to tail logs live, search for a specific phone number, or check PROD for comparison?" Keep it to one line.
