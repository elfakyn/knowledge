## Get a fucking UTC ISO 8601 from timestamp.

```python
datetime.datetime.fromtimestamp(unix_time_milliseconds // 1000, tz=datetime.timezone(datetime.timedelta(0))).isoformat().replace('+00:00', 'Z')
```

Displays: `2020-02-19T23:20:20Z`
