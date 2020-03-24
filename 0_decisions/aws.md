# AWS Decisions

## You have a CloudWatch metric alarm on a lambda infrequently run. What to treat missing data as?

Answer: treat missing data as `missing` (shows INSUFFICIENT in console). The rationale for keeping it as INSUFFICIENT instead of OK is that if a lambda runs every 6 hours and fails every time, in between executions there is insufficient information to know whether it's running correctly or not; you don't want it to show OK.
