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
```
```sql
DROP TABLE IF EXISTS wsi_log_test;
CREATE TABLE wsi_log_test (
  `time` TIMESTAMP(3),
  `remote_addr` STRING,
  `method` STRING,
  `path` STRING,
  `status_code` STRING,
  WATERMARK FOR `time` AS `time` - INTERVAL '1' MINUTES
)
WITH (
  'connector' = 'kinesis',
  'stream' = 'wsi-app-log',
  'aws.region' = 'ap-northeast-2',
  'scan.stream.initpos' = 'LATEST',
  'format' = 'json'
);
```
```sql
%flink.ssql

SELECT window_start, window_end, path, `method`, COUNT(*) as request_count, count(CASE WHEN status_code BETWEEN 400 AND 499 then 1 end) as request_error_count, count(CASE WHEN status_code BETWEEN 500 AND 599 then 1 end) as server_error_count
FROM TABLE(HOP(TABLE wsi_log_test, DESCRIPTOR(`time`), INTERVAL '1' MINUTES, INTERVAL '3' MINUTES))
GROUP BY window_start, window_end, path, `method`;
```
```js
console.log('Loading function');

exports.handler = async (event, context) => {
    /* Process the list of records and transform them */
    const output = event.records.map((record) => {
        const data = JSON.parse(Buffer.from(record.data, 'base64').toString())
        const date = new Date(data.time)
        
        const partitionKeys = {
            year: date.getFullYear(),
            month: date.getMonth() + 1,
            day: date.getDate(),
            hour: date.getHours()
        }
        
        return {
            recordId: record.recordId,
            result: 'Ok',
            data: record.data,
            metadata: {
                partitionKeys
            }
        }
    });
    console.log(`Processing completed.  Successful records ${output.length}.`);
    return { records: output };
};
```
```
logs/year=!{partitionKeyFromLambda:year}/month=!{partitionKeyFromLambda:month}/day=!{partitionKeyFromLambda:day}/hour=!{partitionKeyFromLambda:hour}/
```
```sh
#!/bin/bash
while true; do
        RAND=$(($RANDOM % 4))
        case $RAND in
                0)
                        curl localhost:8080/v1/color/red
                        echo
                        ;;

                1)
                        curl localhost:8080/v1/color/orange
                        echo
                        ;;

                2)
                        curl localhost:8080/v1/color/melon
                        echo
                        ;;

                *)
                        curl localhost:8080/v1/color/oops -s > /dev/null
                        echo oops
                        ;;
        esac
done
```

![image](https://github.com/cloudshit/fluentbit-proba/assets/39158228/bb2c0704-1d09-4707-b6d2-a7b1170a2325)

