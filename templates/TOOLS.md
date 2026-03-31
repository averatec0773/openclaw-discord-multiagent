# TOOLS.md

<!-- This file is for YOUR specifics — environment-specific details unique to your setup.
     Skills define *how* tools work. This file holds *your* accounts, IDs, and config. -->

## Google (gog)

- Default account: YOUR_EMAIL@gmail.com (set via GOG_ACCOUNT env var)
- No --account flag needed — gog picks it up automatically
- Gmail, Calendar, Drive enabled

## Discord

- Bot guild: YOUR_GUILD_NAME (GUILD_ID)
- Owner Discord user ID: YOUR_USER_ID
- Owner timezone: YOUR_TIMEZONE
  Always show local time, not UTC.

## Usage Analysis

Session JSONL files: `/home/node/.openclaw/agents/main/sessions/*.jsonl`
Each assistant turn has `message.usage` with `input`, `output`, `cacheRead`, `cacheWrite`, `cost.total`.

Run this Python snippet to get a daily cost breakdown:

```python
import json, glob
files = glob.glob('/home/node/.openclaw/agents/main/sessions/*.jsonl')
daily = {}
total_cost = 0
for f in sorted(files):
    with open(f) as fh:
        for line in fh:
            try:
                obj = json.loads(line)
                msg = obj.get('message', {})
                usage = msg.get('usage', {})
                ts = obj.get('timestamp', '')[:10]
                if not (usage and (usage.get('input', 0) or usage.get('cacheRead', 0))):
                    continue
                cost = usage.get('cost', {}).get('total', 0)
                total_cost += cost
                d = daily.setdefault(ts, {'in':0,'cr':0,'cost':0,'turns':0})
                d['in'] += usage.get('input', 0)
                d['cr'] += usage.get('cacheRead', 0)
                d['cost'] += cost
                d['turns'] += 1
            except:
                pass
for day in sorted(daily):
    d = daily[day]
    cpt = d['cost']/d['turns'] if d['turns'] else 0
    print('%s  input=%6.1fK  cacheRead=%6.1fK  cost=$%.3f  $/turn=$%.4f' % (
        day, d['in']/1000, d['cr']/1000, d['cost'], cpt))
print('TOTAL: $%.4f' % total_cost)
```

Signal to run `/compact`: `$/turn` rising above ~$0.05 on routine conversations.

<!-- Add other services below as needed:
## SSH / Servers
- alias: your-server → user@host

## Camera / Devices
- living-room: Reolink E1
-->
