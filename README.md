# fluentbit-proba
```ini
[SERVICE]
    Parsers_File parsers.conf

[INPUT]
    Name tail
    Path /home/ec2-user/app/app.log
    Parser app

[OUTPUT]
    name  stdout
    match *

[OUTPUT]
    Name  kinesis_streams
    Match *
    region ap-northeast-2
    stream wsi-app-log
    time_key time
    time_key_format %Y-%m-%d %H:%M:%S.%3N
```
```ini
[PARSER]
    Name app
    Format regex
    Regex ^\[(?<time>[^\]]*)\] (?<remote_addr>[^ ]*) - - (?<method>\S+) (?<path>\S+) HTTP/1.1 (?<status_code>[^ ]*)?$
    Types status_code:integer
    Time_Key time
    Time_Format %Y-%m-%d %H:%M:%S,%L
    Time_Offset +0900
```

![image](https://github.com/cloudshit/fluentbit-proba/assets/39158228/bb2c0704-1d09-4707-b6d2-a7b1170a2325)

