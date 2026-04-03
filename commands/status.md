---
description: Check QkConvert API status, key validity, and account info
---

# /qkconvert:status

Check if the QkConvert API is reachable and your API key is valid.

## Steps

### 1. Check API key is set

Run in bash:
```bash
test -n "$QKCONVERT_API_KEY" && echo "set" || echo "not set"
```

If not set, tell the user:
> Your QkConvert API key is not set.
> 1. Sign up at https://qkconvert.dev/portal/signup
> 2. Create an API key in the dashboard
> 3. **Mac/Linux:** `export QKCONVERT_API_KEY=sk_live_...` in `~/.bashrc`
> 4. **Windows:** `setx QKCONVERT_API_KEY sk_live_...`
> 5. Restart Claude Code

### 2. Check API health

```bash
curl -s https://qkconvert.dev/api/health
```

Expected: `{"status":"ok"}`

### 3. Verify API key works

```bash
curl -s -o /dev/null -w "%{http_code}" -X POST https://qkconvert.dev/api/v1/image/metadata -H "Authorization: Bearer $QKCONVERT_API_KEY" -F "image=@/dev/null"
```

- **400** = key is valid (bad request because empty file, but auth passed)
- **401** = key is invalid or revoked
- **403** = quota exhausted
- **429** = rate limited

### 4. Report

Tell the user:
- API status (healthy/unreachable)
- Key status (valid/invalid/not set)
- If 403: credits exhausted, suggest upgrading at https://qkconvert.dev/portal/dashboard
- Link to dashboard: https://qkconvert.dev/portal/dashboard
- Link to sandbox: https://qkconvert.dev/sandbox
