# Lambda metric alarm

When you create a lambda metric alarm you really don't want to have to think about it. Here's the values (in terraform, but you can use the same ones in cloudformation):

```terraform
resource "aws_cloudwatch_metric_alarm" "mylambda_error_alarm" {
  alarm_name          = "mylambda_error_alarm"
  alarm_description   = "MyLambda erorred out !!"
  comparison_operator = "GreaterThanOrEqualToThreshold"
  evaluation_periods  = "1"
  metric_name         = "Errors"
  namespace           = "AWS/Lambda"
  dimensions = {
    FunctionName = aws_lambda_function.mylambda.function_name
  }
  threshold                 = "1"
  period                    = "60"
  statistic                 = "Sum"
  treat_missing_data        = "missing"
  insufficient_data_actions = []
  alarm_actions             = [var.destination_sns_topic]
}
```

## IMPORTANT! You want to treat missing data as `missing`!

Why? Even though it's no fun to have a bunch of INSUFFICIENTs in CloudWatch, the rationale for keeping it as INSUFFICIENT instead of OK is that if a lambda runs every 6 hours and fails every time, in between executions there is insufficient information to know whether it's running correctly or not; you don't want it to show OK. **You never want something to show up as OK when you can't make that determination for sure!**

You also don't want to use `ignore` since in that case the alarm will only fire once for multiple consecutive failures: `ignore` needs a successful execution before it can alarm again. `missing` alarms every time.

## Frequent execution caveat

If you have a lambda that executes more often than the alarm period, then the alarm will only fire once and never have time to reset to `INSUFFICIENT`. If that's important you need to either reduce the period (extra cost) or figure some other way to not drop the ball on the alarm.
