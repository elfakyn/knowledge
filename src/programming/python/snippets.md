## Timezone madness

```python
import datetime
```

### Timestamp milliseconds to zulu time

```python
datetime.datetime.fromtimestamp(unix_time_milliseconds // 1000, tz=datetime.timezone.utc).isoformat().replace('+00:00', 'Z')
```

Input: `1582154420000`
Output: `'2020-02-19T23:20:20Z'`

### Zulu time to timestamp milliseconds

```python
int(datetime.datetime.strptime(formatted_timestamp, '%Y-%m-%dT%H:%M:%SZ').replace(tzinfo=datetime.timezone.utc).timestamp() * 1000)
```

Input: `'2020-02-19T23:20:20Z'`
Output: `1582154420000`
