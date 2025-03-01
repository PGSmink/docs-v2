---
title: Use the influx query command
description: Query InfluxDB using the influx CLI. Discover how to query data in InfluxDB 2.1 using `influx query`.
weight: 204
menu:
  influxdb_2_7:
    name: Use the influx CLI
    parent: Execute queries
influxdb/v2.7/tags: [query]
related:
  - /influxdb/v2.7/reference/cli/influx/query/
---

Use the [`influx query` command](/influxdb/v2.7/reference/cli/influx/query) to query data in InfluxDB using Flux.
Pass Flux queries to the command as either a file or via stdin.

###### Run a query from a file

```bash
influx query --file /path/to/query.flux
```

###### Pass raw Flux via stdin pipe

```bash
influx query - # Return to open the pipe

data = from(bucket: "example-bucket") |> range(start: -10m) # ...
# Linux & macOS: <ctrl-d> to close the pipe and submit the command
# Windows: <enter>, then <ctrl-d>, then <enter> to close the pipe and submit the command
```

{{% note %}}
#### Remove unnecessary columns in large datasets
When using the `influx query` command to query and download large datasets,
drop columns such as `_start` and `_stop` to optimize the download file size.

```js
// ...
    |> drop(columns: ["_start", "_stop"])
```
{{% /note %}}
