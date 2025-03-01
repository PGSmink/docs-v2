---
title: Write data with the influx CLI
weight: 205
description: >
  Use the `influx write` command to write data to InfluxDB from the command line.
menu:
  influxdb_2_4:
    name: Influx CLI
    parent: Developer tools
related:
  - /influxdb/v2.4/write-data/developer-tools/csv/
---

To write data from the command line, use the [`influx write` command](/influxdb/v2.4/reference/cli/influx/write/).
Include the following in your command:

| Requirement          | Include by                                                                                         |
|:-----------          |:----------                                                                                         |
| Organization         | Use the `-o`,`--org`, or `--org-id` flags.                                                         |
| Bucket               | Use the `-b`, `--bucket`, or `--bucket-id` flags.                                                  |
| Precision            | Use the `-p`, `--precision` flag.                                                                  |
| API token | Set the `INFLUX_TOKEN` environment variable or use the `t`, `--token` flag.                        |
| Data                 | Write data using **line protocol** or **annotated CSV**. Pass a file with the `-f`, `--file` flag. |

_See [Line protocol](/influxdb/v2.4/reference/syntax/line-protocol/) and [Annotated CSV](/influxdb/v2.4/reference/syntax/annotated-csv)_

#### Example influx write commands

##### Write a single line of line protocol
```sh
influx write \
  -b bucketName \
  -o orgName \
  -p s \
  'myMeasurement,host=myHost testField="testData" 1556896326'
```

##### Write line protocol from a file
```sh
influx write \
  -b bucketName \
  -o orgName \
  -p s \
  --format=lp
  -f /path/to/line-protocol.txt
```

##### Write annotated CSV from a file
```sh
influx write \
  -b bucketName \
  -o orgName \
  -p s \
  --format=csv
  -f /path/to/data.csv
```
