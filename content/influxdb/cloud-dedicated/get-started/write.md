---
title: Get started writing data
seotitle: Write data | Get started with InfluxDB Cloud Dedicated
list_title: Write data
description: >
  Get started writing data to InfluxDB by learning about line protocol
  and using tools like Telegraf, client libraries, and the InfluxDB API.
menu:
  influxdb_cloud_dedicated:
    name: Write data
    parent: Get started
    identifier: get-started-write-data
weight: 101
metadata: [2 / 3]
related:
  - /influxdb/cloud-dedicated/write-data/
  - /influxdb/cloud-dedicated/write-data/best-practices/
  - /influxdb/cloud-dedicated/reference/syntax/line-protocol/
  - /{{< latest "telegraf" >}}/
---

This tutorial walks you through the fundamental of creating **line protocol** data and writing it to InfluxDB.

InfluxDB provides many different options for ingesting or writing data, including the following:

- InfluxDB HTTP API (v1 and v2)
- Telegraf
- `influx3` data CLI
- InfluxDB client libraries

If using tools like Telegraf or InfluxDB client libraries, they can
build the line protocol for you, but it's good to understand how line protocol works.

## Line protocol

All data written to InfluxDB is written using **line protocol**, a text-based
format that lets you provide the necessary information to write a data point to InfluxDB.
_This tutorial covers the basics of line protocol, but for detailed information,
see the [Line protocol reference](/influxdb/cloud-dedicated/reference/syntax/line-protocol/)._

### Line protocol elements

Each line of line protocol contains the following elements:

{{< req type="key" >}}

- {{< req "\*" >}} **measurement**:  String that identifies the [measurement](/influxdb/cloud-dedicated/reference/glossary/#measurement) to store the data in.
- **tag set**: Comma-delimited list of key value pairs, each representing a tag.
  Tag keys and values are unquoted strings. _Spaces, commas, and equal characters must be escaped._
- {{< req "\*" >}} **field set**: Comma-delimited list of key value pairs, each representing a field.
  Field keys are unquoted strings. _Spaces and commas must be escaped._
  Field values can be [strings](/influxdb/cloud-dedicated/reference/syntax/line-protocol/#string) (quoted),
  [floats](/influxdb/cloud-dedicated/reference/syntax/line-protocol/#float),
  [integers](/influxdb/cloud-dedicated/reference/syntax/line-protocol/#integer),
  [unsigned integers](/influxdb/cloud-dedicated/reference/syntax/line-protocol/#uinteger),
  or [booleans](/influxdb/cloud-dedicated/reference/syntax/line-protocol/#boolean).
- **timestamp**: [Unix timestamp](/influxdb/cloud-dedicated/reference/syntax/line-protocol/#unix-timestamp)
  associated with the data. InfluxDB supports up to nanosecond precision.
  _If the precision of the timestamp is not in nanoseconds, you must specify the
  precision when writing the data to InfluxDB._

#### Line protocol element parsing

- **measurement**: Everything before the _first unescaped comma before the first whitespace_.
- **tag set**: Key-value pairs between the _first unescaped comma_ and the _first unescaped whitespace_.
- **field set**: Key-value pairs between the _first and second unescaped whitespaces_.
- **timestamp**: Integer value after the _second unescaped whitespace_.
- Lines are separated by the newline character (`\n`).
  Line protocol is whitespace sensitive.

---

{{< influxdb/line-protocol >}}

---

_For schema design recommendations, see [InfluxDB schema design](/influxdb/cloud-dedicated/write-data/best-practices/schema-design/)._

## Construct line protocol

With a basic understanding of line protocol, you can now construct line protocol
and write data to InfluxDB.
Consider a use case where you collect data from sensors in your home.
Each sensor collects temperature, humidity, and carbon monoxide readings.
To collect this data, use the following schema:

- **measurement**: `home` 
  - **tags**
    - `room`: Living Room or Kitchen
  - **fields**
    - `temp`: temperature in °C (float)
    - `hum`: percent humidity (float)
    - `co`: carbon monoxide in parts per million (integer)
  - **timestamp**: Unix timestamp in _second_ precision

Data is collected hourly beginning at 
{{% influxdb/custom-timestamps-span %}}**2022-01-01T08:00:00Z (UTC)** until **2022-01-01T20:00:00Z (UTC)**{{% /influxdb/custom-timestamps-span %}}.
The resulting line protocol would look something like the following:

{{% influxdb/custom-timestamps %}}

##### Home sensor data line protocol

```sh
home,room=Living\ Room temp=21.1,hum=35.9,co=0i 1641024000
home,room=Kitchen temp=21.0,hum=35.9,co=0i 1641024000
home,room=Living\ Room temp=21.4,hum=35.9,co=0i 1641027600
home,room=Kitchen temp=23.0,hum=36.2,co=0i 1641027600
home,room=Living\ Room temp=21.8,hum=36.0,co=0i 1641031200
home,room=Kitchen temp=22.7,hum=36.1,co=0i 1641031200
home,room=Living\ Room temp=22.2,hum=36.0,co=0i 1641034800
home,room=Kitchen temp=22.4,hum=36.0,co=0i 1641034800
home,room=Living\ Room temp=22.2,hum=35.9,co=0i 1641038400
home,room=Kitchen temp=22.5,hum=36.0,co=0i 1641038400
home,room=Living\ Room temp=22.4,hum=36.0,co=0i 1641042000
home,room=Kitchen temp=22.8,hum=36.5,co=1i 1641042000
home,room=Living\ Room temp=22.3,hum=36.1,co=0i 1641045600
home,room=Kitchen temp=22.8,hum=36.3,co=1i 1641045600
home,room=Living\ Room temp=22.3,hum=36.1,co=1i 1641049200
home,room=Kitchen temp=22.7,hum=36.2,co=3i 1641049200
home,room=Living\ Room temp=22.4,hum=36.0,co=4i 1641052800
home,room=Kitchen temp=22.4,hum=36.0,co=7i 1641052800
home,room=Living\ Room temp=22.6,hum=35.9,co=5i 1641056400
home,room=Kitchen temp=22.7,hum=36.0,co=9i 1641056400
home,room=Living\ Room temp=22.8,hum=36.2,co=9i 1641060000
home,room=Kitchen temp=23.3,hum=36.9,co=18i 1641060000
home,room=Living\ Room temp=22.5,hum=36.3,co=14i 1641063600
home,room=Kitchen temp=23.1,hum=36.6,co=22i 1641063600
home,room=Living\ Room temp=22.2,hum=36.4,co=17i 1641067200
home,room=Kitchen temp=22.7,hum=36.5,co=26i 1641067200
```

{{% /influxdb/custom-timestamps %}}

## Write line protocol to InfluxDB

The following examples show how to write the 
[sample data](#home-sensor-data-line-protocol), already in line protocol format,
to an {{% cloud-name %}} database.

To learn more about available tools and options, see [Write data](/influxdb/cloud-dedicated/write-data/).

{{% note %}}
Some examples in this getting started tutorial assume your InfluxDB
credentials (**url**, **organization**, and **token**) are provided by
[environment variables](/influxdb/cloud-dedicated/get-started/setup/?t=InfluxDB+API#configure-authentication-credentials).
{{% /note %}}

{{< tabs-wrapper >}}
{{% tabs %}}
[Telegraf](#)
[cURL](#)
[Python](#)
[Go](#)
[Node.js](#)
[C#](#)
{{% /tabs %}}
{{% tab-content %}}
<!------------------------------- BEGIN TELEGRAF CONTENT ------------------------------>
Use [Telegraf](/{{< latest "telegraf" >}}/) to consume line protocol,
and then write it to {{< cloud-name >}}.

1.  If you haven't already, follow the instructions to [download and install Telegraf](/{{< latest "telegraf" >}}/install/).

2.  Copy and save the [home sensor data sample](#home-sensor-data-line-protocol) to a file on your local system--for example, `home.lp`.

3.  Run the following command to generate a Telegraf configuration file (`./telegraf.conf`) that enables the `inputs.file` and `outputs.influxdb_v2` plugins:

    ```sh
    telegraf --sample-config \
      --input-filter file \
      --output-filter influxdb_v2 \
      > telegraf.conf
    ```

4.  In your editor, open `./telegraf.conf` and configure the following:

    - **`file` input plugin**: In the `[[inputs.file]].files` list, replace `"/tmp/metrics.out"` with your sample data filename.
      If Telegraf can't find a file when started, it stops processing and exits.

      ```toml
      [[inputs.file]]
        ## Files to parse each interval.  Accept standard unix glob matching rules,
        ## as well as ** to match recursive files and directories.
        files = ["home.lp"]
      ```

    - **`output-influxdb_v2` output plugin**: In the `[[outputs.influxdb_v2]]` section, replace the default values with the following configuration for your InfluxDB Cloud Dedicated cluster:

      ```toml
      [[outputs.influxdb_v2]]
        # InfluxDB Cloud Dedicated cluster URL
        urls = ["${INFLUX_URL}"]

        # INFLUX_TOKEN is an environment variable you assigned to your database token
        token = "${INFLUX_TOKEN}"

        # An empty string (InfluxDB ignores this parameter)
        organization = ""

        # Database name
        bucket = "get-started"
      ```

      The example configuration uses the following InfluxDB credentials:

      - **`urls`**: an array containing your **`INFLUX_URL`** environment variable
      - **`token`**: your **`INFLUX_TOKEN`** environment variable
      - **`organization`**: an empty string (InfluxDB ignores this parameter)
      - **`bucket`**: the name of the database to write to

5.  To write the data, start the `telegraf` daemon with the following options:

    - `--config`: Specifies the filepath of the configuration file.
    - `--once`: Runs a single Telegraf collection cycle for the configured inputs and outputs, and then exits.

    Enter the following command in your terminal:

    ```sh
    telegraf --once --config ./telegraf.conf
    ```

    If the write is successful, the output is similar to the following:

    ```plaintext
    2023-05-31T20:09:08Z D! [agent] Starting service inputs
    2023-05-31T20:09:19Z D! [outputs.influxdb_v2] Wrote batch of 52 metrics in 348.008167ms
    2023-05-31T20:09:19Z D! [outputs.influxdb_v2] Buffer fullness: 0 / 10000 metrics
    ```

Telegraf and its plugins provide many options for reading and writing data.
To learn more, see how to [use Telegraf to write data](/influxdb/cloud-dedicated/write-data/use-telegraf/).

<!------------------------------- END TELEGRAF CONTENT ------------------------------>
{{% /tab-content %}}
{{% tab-content %}}
<!----------------------------- BEGIN cURL CONTENT ----------------------------->

To write data to InfluxDB using the InfluxDB v2 HTTP API, send a
request to the InfluxDB API `/api/v2/write` endpoint using the `POST` request method.

{{< api-endpoint endpoint="https://cluster-id.influxdb.io/api/v2/write" method="post" api-ref="/influxdb/cloud-iox/api/#operation/PostWrite" >}}

Include the following with your request:

- **Headers**:
  - **Authorization**: Bearer <INFLUX_TOKEN>
  - **Content-Type**: text/plain; charset=utf-8
  - **Accept**: application/json
- **Query parameters**:
  - **bucket**: InfluxDB database name
  - **precision**: Timestamp precision (default is `ns`)
- **Request body**: Line protocol as plain text

{{% note %}}
With the {{% cloud-name %}} v2 API `/api/v2/write` endpoint, `Authorization: Bearer` and `Authorization: Token` are equivalent and you can use either scheme to pass a database token in your request. For more information about HTTP API token schemes, see how to [authenticate API requests](/influxdb/cloud-dedicated/primers/api/v2/#authenticate-api-requests).
{{% /note %}}

The following example uses cURL and the InfluxDB v2 API to write line protocol
to InfluxDB:

{{% code-placeholders "DATABASE_TOKEN"%}}
{{% influxdb/custom-timestamps %}}
```sh
curl --request POST \
"https://cluster-id.influxdb.io/api/v2/write?bucket=get-started&precision=s" \
  --header "Authorization: Bearer DATABASE_TOKEN" \
  --header "Content-Type: text/plain; charset=utf-8" \
  --header "Accept: application/json" \
  --data-binary "
home,room=Living\ Room temp=21.1,hum=35.9,co=0i 1641024000
home,room=Kitchen temp=21.0,hum=35.9,co=0i 1641024000
home,room=Living\ Room temp=21.4,hum=35.9,co=0i 1641027600
home,room=Kitchen temp=23.0,hum=36.2,co=0i 1641027600
home,room=Living\ Room temp=21.8,hum=36.0,co=0i 1641031200
home,room=Kitchen temp=22.7,hum=36.1,co=0i 1641031200
home,room=Living\ Room temp=22.2,hum=36.0,co=0i 1641034800
home,room=Kitchen temp=22.4,hum=36.0,co=0i 1641034800
home,room=Living\ Room temp=22.2,hum=35.9,co=0i 1641038400
home,room=Kitchen temp=22.5,hum=36.0,co=0i 1641038400
home,room=Living\ Room temp=22.4,hum=36.0,co=0i 1641042000
home,room=Kitchen temp=22.8,hum=36.5,co=1i 1641042000
home,room=Living\ Room temp=22.3,hum=36.1,co=0i 1641045600
home,room=Kitchen temp=22.8,hum=36.3,co=1i 1641045600
home,room=Living\ Room temp=22.3,hum=36.1,co=1i 1641049200
home,room=Kitchen temp=22.7,hum=36.2,co=3i 1641049200
home,room=Living\ Room temp=22.4,hum=36.0,co=4i 1641052800
home,room=Kitchen temp=22.4,hum=36.0,co=7i 1641052800
home,room=Living\ Room temp=22.6,hum=35.9,co=5i 1641056400
home,room=Kitchen temp=22.7,hum=36.0,co=9i 1641056400
home,room=Living\ Room temp=22.8,hum=36.2,co=9i 1641060000
home,room=Kitchen temp=23.3,hum=36.9,co=18i 1641060000
home,room=Living\ Room temp=22.5,hum=36.3,co=14i 1641063600
home,room=Kitchen temp=23.1,hum=36.6,co=22i 1641063600
home,room=Living\ Room temp=22.2,hum=36.4,co=17i 1641067200
home,room=Kitchen temp=22.7,hum=36.5,co=26i 1641067200
"
```
{{% /influxdb/custom-timestamps %}}
{{% /code-placeholders %}}
<!------------------------------ END cURL CONTENT ------------------------------>
{{% /tab-content %}}
{{% tab-content %}}
<!---------------------------- BEGIN PYTHON CONTENT --------------------------->
{{% influxdb/custom-timestamps %}}

To write data to {{% cloud-name %}} using Python, use the
[`influxdb_client_3` module](https://github.com/InfluxCommunity/influxdb3-python).
The following steps include setting up a Python virtual environment to scope
dependencies to your current project.

1. Create a new module directory and navigate into it--for example:

    ```sh
    mkdir influxdb_py_client && cd $_
    ```

2.  Setup your Python virtual environment.
    Inside of your module directory:

    ```sh
    python -m venv envs/virtual-env
    ```

3. Activate the virtual environment.

    ```sh
    source ./envs/virtual-env/bin/activate
    ```

4.  Install the client library package:

    ```sh
    pip install influxdb3-python
    ```

    The `influxdb3-python` package provides the `influxdb_client_3` module and also installs the [`pyarrow` package](https://arrow.apache.org/docs/python/index.html) for working with Arrow data returned from queries.

5.  In your terminal or editor, create a new file for your code--for example: `write.py`.

    ```sh
    touch write.py
    ```

6.  Inside of `write.py`, enter the following sample code:

      ```py
      from influxdb_client_3 import InfluxDBClient3
      import os

      # INFLUX_TOKEN is an environment variable you assigned to your
      # database token value.
      token = os.getenv('INFLUX_TOKEN')

      # host is the URL without protocol or trailing slash
      client = InfluxDBClient3(
          host='cluster-id.influxdb.io',
          org='',
          token=token,
          database='get-started'
      )

      lines = [
          "home,room=Living\ Room temp=21.1,hum=35.9,co=0i 1641024000",
          "home,room=Kitchen temp=21.0,hum=35.9,co=0i 1641024000",
          "home,room=Living\ Room temp=21.4,hum=35.9,co=0i 1641027600",
          "home,room=Kitchen temp=23.0,hum=36.2,co=0i 1641027600",
          "home,room=Living\ Room temp=21.8,hum=36.0,co=0i 1641031200",
          "home,room=Kitchen temp=22.7,hum=36.1,co=0i 1641031200",
          "home,room=Living\ Room temp=22.2,hum=36.0,co=0i 1641034800",
          "home,room=Kitchen temp=22.4,hum=36.0,co=0i 1641034800",
          "home,room=Living\ Room temp=22.2,hum=35.9,co=0i 1641038400",
          "home,room=Kitchen temp=22.5,hum=36.0,co=0i 1641038400",
          "home,room=Living\ Room temp=22.4,hum=36.0,co=0i 1641042000",
          "home,room=Kitchen temp=22.8,hum=36.5,co=1i 1641042000",
          "home,room=Living\ Room temp=22.3,hum=36.1,co=0i 1641045600",
          "home,room=Kitchen temp=22.8,hum=36.3,co=1i 1641045600",
          "home,room=Living\ Room temp=22.3,hum=36.1,co=1i 1641049200",
          "home,room=Kitchen temp=22.7,hum=36.2,co=3i 1641049200",
          "home,room=Living\ Room temp=22.4,hum=36.0,co=4i 1641052800",
          "home,room=Kitchen temp=22.4,hum=36.0,co=7i 1641052800",
          "home,room=Living\ Room temp=22.6,hum=35.9,co=5i 1641056400",
          "home,room=Kitchen temp=22.7,hum=36.0,co=9i 1641056400",
          "home,room=Living\ Room temp=22.8,hum=36.2,co=9i 1641060000",
          "home,room=Kitchen temp=23.3,hum=36.9,co=18i 1641060000",
          "home,room=Living\ Room temp=22.5,hum=36.3,co=14i 1641063600",
          "home,room=Kitchen temp=23.1,hum=36.6,co=22i 1641063600",
          "home,room=Living\ Room temp=22.2,hum=36.4,co=17i 1641067200",
          "home,room=Kitchen temp=22.7,hum=36.5,co=26i 1641067200"
      ]

      client.write(lines,write_precision='s')
      ```

    The sample does the following:

    1.  Imports the `InfluxDBClient3` object from the `influxdb_client_3` module.
    2.  Calls the `InfluxDBClient3()` constructor to instantiate an InfluxDB client
        configured with the following credentials:

        - **host**: {{% cloud-name %}} cluster hostname (URL without protocol or trailing slash)
        - **org**: an empty or arbitrary string (InfluxDB ignores this parameter)
        - **token**: an InfluxDB [database token](/influxdb/cloud-dedicated/admin/tokens/) with write access to the target database
        - **database**: the name of the {{% cloud-name %}} database to write to
    
    3.  Defines a list of line protocol strings where each string represents a data record.
    4.  Calls the `client.write()` method with the line protocol record list and write options.

        **Because the timestamps in the sample line protocol are in second
        precision, the example passes the `write_precision='s'` option
        to set the timestamp precision to seconds.**

6.  To execute the module and write line protocol to your {{% cloud-name %}}
    database, enter the following command in your terminal:
    
      ```sh
      python write.py
      ```

{{% /influxdb/custom-timestamps %}}

<!----------------------------- END PYTHON CONTENT ---------------------------->
{{% /tab-content %}}
{{% tab-content %}}
<!----------------------------- BEGIN GO CONTENT ------------------------------>
{{% influxdb/custom-timestamps %}}

To write data to {{% cloud-name %}} using Go, use the
InfluxDB v3 [influxdb3-go client library package](https://github.com/InfluxCommunity/influxdb3-go).

1.  Inside of your project directory, create a new module directory and navigate into it.

    ```sh
    mkdir influxdb_go_client && cd $_
    ```

2.  Initialize a new Go module in the directory.

    ```sh
    go mod init influxdb_go_client
    ```

3.  In your terminal or editor, create a new file for your code--for example: `write.go`.

    ```sh
    touch write.go
    ```

4.  Inside of `write.go`, enter the following sample code:

    ```go
    package main

    import (
      "context"
      "os"
      "fmt"
      "log"

      "github.com/InfluxCommunity/influxdb3-go/influx"
    )

    // Write line protocol data to InfluxDB
    func WriteLineProtocol() error {
      url := "https://cluster-id.influxdb.io"
      // INFLUX_TOKEN is an environment variable you assigned to your
      // database token value.
      token := os.Getenv("INFLUX_TOKEN")
      database := os.Getenv("INFLUX_DATABASE")
      
      // Initialize a client with URL and token,
      // and set the timestamp precision for writes.
      client, err := influx.New(influx.Configs{
        HostURL: url,
        AuthToken: token,
		    WriteParams: influx.WriteParams{Precision: lineprotocol.Second},
      })

      // Close the client when the function returns.
      defer func (client *influx.Client)  {
        err := client.Close()
        if err != nil {
            panic(err)
        }
      }(client)

      // Define line protocol records to write.
      // Use a raw string literal (denoted by backticks)
      // to preserve backslashes and prevent interpretation
      // of escape sequences--for example, escaped spaces in tag values.
      lines := [...]string{
        `home,room=Living\ Room temp=21.1,hum=35.9,co=0i 1641124000`,
        `home,room=Kitchen temp=21.0,hum=35.9,co=0i 1641124000`,
        `home,room=Living\ Room temp=21.4,hum=35.9,co=0i 1641127600`,
        `home,room=Kitchen temp=23.0,hum=36.2,co=0i 1641127600`,
        `home,room=Living\ Room temp=21.8,hum=36.0,co=0i 1641131200`,
        `home,room=Kitchen temp=22.7,hum=36.1,co=0i 1641131200`,
        `home,room=Living\ Room temp=22.2,hum=36.0,co=0i 1641134800`,
        `home,room=Kitchen temp=22.4,hum=36.0,co=0i 1641134800`,
        `home,room=Living\ Room temp=22.2,hum=35.9,co=0i 1641138400`,
        `home,room=Kitchen temp=22.5,hum=36.0,co=0i 1641138400`,
        `home,room=Living\ Room temp=22.4,hum=36.0,co=0i 1641142000`,
        `home,room=Kitchen temp=22.8,hum=36.5,co=1i 1641142000`,
        `home,room=Living\ Room temp=22.3,hum=36.1,co=0i 1641145600`,
        `home,room=Kitchen temp=22.8,hum=36.3,co=1i 1641145600`,
        `home,room=Living\ Room temp=22.3,hum=36.1,co=1i 1641149200`,
        `home,room=Kitchen temp=22.7,hum=36.2,co=3i 1641149200`,
        `home,room=Living\ Room temp=22.4,hum=36.0,co=4i 1641152800`,
        `home,room=Kitchen temp=22.4,hum=36.0,co=7i 1641152800`,
        `home,room=Living\ Room temp=22.6,hum=35.9,co=5i 1641156400`,
        `home,room=Kitchen temp=22.7,hum=36.0,co=9i 1641156400`,
        `home,room=Living\ Room temp=22.8,hum=36.2,co=9i 1641160000`,
        `home,room=Kitchen temp=23.3,hum=36.9,co=18i 1641160000`,
        `home,room=Living\ Room temp=22.5,hum=36.3,co=14i 1641163600`,
        `home,room=Kitchen temp=23.1,hum=36.6,co=22i 1641163600`,
        `home,room=Living\ Room temp=22.2,hum=36.4,co=17i 1641167200`,
        `home,room=Kitchen temp=22.7,hum=36.5,co=26i 1641167200`,
      }
      
      // Iterate over the lines array and write each line
      // separately to InfluxDB
      for _, record := range lines {
        err = client.Write(context.Background(), database, []byte(record))
        if err != nil {
          log.Fatalf("Error writing line protocol: %v", err)
        }
      }

      if err != nil {
        panic(err)
      }
      
      fmt.Println("Data has been written successfully.")
      return nil
    }
    ```

    The sample does the following:

    1.  Imports required packages.
      
    2.  Defines a `WriteLineProtocol()` function that does the following:
        
        1.  To instantiate the client, calls the `influx.New(influx.Configs)` function and passes the InfluxDB URL,
            database token, and [timestamp precision](/influxdb/cloud-dedicated/reference/glossary/#timestamp-precision) for writing data to {{% cloud-name %}}.

        2.  Defines a deferred function that closes the client when the function returns.
    
        3.  Defines an array of line protocol strings where each string
            represents a data record.
    
        4.  Iterates through the array of line protocol and calls the
            write client's `Write()` method
            to write each line of line protocol separately to InfluxDB.

4.  In your editor, create a `main.go` file and enter the following sample code that calls the `WriteLineProtocol()` function:

    ```go
    package main

    // Module main function
    func main() {	
      WriteLineProtocol()
    }
    ```

5.  In your terminal, enter the following command to install the packages listed in `imports`, build the `influxdb_go_client` module, and execute the `main()` function:

    ```sh
    go mod tidy && go build && go run influxdb_go_client
    ```

    The program writes the line protocol to your {{% cloud-name %}} database.

{{% /influxdb/custom-timestamps %}}
<!------------------------------- END GO CONTENT ------------------------------>
{{% /tab-content %}}
{{% tab-content %}}
<!------------------------------- BEGIN NODE.JS CONTENT ----------------------->
{{% influxdb/custom-timestamps %}}

To write data to {{% cloud-name %}} using Node.js, use the
[influxdb-client-js package](https://github.com/influxdata/influxdb-client-js).

1. Inside of your project directory, create an NPM or Yarn package and install 
    the `@influxdata/influxdb-client` InfluxDB v2 JavaScript client library.

    ```sh
    npm init -y && npm install --save @influxdata/influxdb-client
    ```

2.  In your terminal or editor, create a new file for your code--for example: `write.js`.

    ```sh
    touch write.js
    ```

3. Inside of `write.js`, enter the following sample code:

    ```js
    'use strict'
    /** @module write
    * Writes line protocol strings to InfluxDB using the JavaScript client
    * library with Node.js.
    **/
    import {InfluxDB} from '@influxdata/influxdb-client';

    /** Get credentials from environment variables. **/
    const url = process.env.INFLUX_URL;
    const token = process.env.INFLUX_TOKEN;
    const org = process.env.INFLUX_ORG;

    /**
    * Instantiate the InfluxDB client with a configuration object.
    **/
    const influxDB = new InfluxDB({url, token});

    /** 
    * Define an array of line protocol strings to write.
    * Include an additional backslash to preserve backslashes
    * and prevent interpretation of escape sequences---for example,
    * escaped spaces in tag values.
    */
    const lines = [
      'home,room=Living\\ Room temp=21.1,hum=35.9,co=0i 1641024000',
      'home,room=Kitchen temp=21.0,hum=35.9,co=0i 1641024000',
      'home,room=Living\\ Room temp=21.4,hum=35.9,co=0i 1641027600',
      'home,room=Kitchen temp=23.0,hum=36.2,co=0i 1641027600',
      'home,room=Living\\ Room temp=21.8,hum=36.0,co=0i 1641031200',
      'home,room=Kitchen temp=22.7,hum=36.1,co=0i 1641031200',
      'home,room=Living\\ Room temp=22.2,hum=36.0,co=0i 1641034800',
      'home,room=Kitchen temp=22.4,hum=36.0,co=0i 1641034800',
      'home,room=Living\\ Room temp=22.2,hum=35.9,co=0i 1641038400',
      'home,room=Kitchen temp=22.5,hum=36.0,co=0i 1641038400',
      'home,room=Living\\ Room temp=22.4,hum=36.0,co=0i 1641042000',
      'home,room=Kitchen temp=22.8,hum=36.5,co=1i 1641042000',
      'home,room=Living\\ Room temp=22.3,hum=36.1,co=0i 1641045600',
      'home,room=Kitchen temp=22.8,hum=36.3,co=1i 1641045600',
      'home,room=Living\\ Room temp=22.3,hum=36.1,co=1i 1641049200',
      'home,room=Kitchen temp=22.7,hum=36.2,co=3i 1641049200',
      'home,room=Living\\ Room temp=22.4,hum=36.0,co=4i 1641052800',
      'home,room=Kitchen temp=22.4,hum=36.0,co=7i 1641052800',
      'home,room=Living\\ Room temp=22.6,hum=35.9,co=5i 1641056400',
      'home,room=Kitchen temp=22.7,hum=36.0,co=9i 1641056400',
      'home,room=Living\\ Room temp=22.8,hum=36.2,co=9i 1641060000',
      'home,room=Kitchen temp=23.3,hum=36.9,co=18i 1641060000',
      'home,room=Living\\ Room temp=22.5,hum=36.3,co=14i 1641063600',
      'home,room=Kitchen temp=23.1,hum=36.6,co=22i 1641063600',
      'home,room=Living\\ Room temp=22.2,hum=36.4,co=17i 1641067200',
      'home,room=Kitchen temp=22.7,hum=36.5,co=26i 1641067200',
    ];

    /**
    * Create a write client from the getWriteApi method.
    * Provide your org and database.
    **/
    const writeApi = influxDB.getWriteApi(org, 'get-started', 's');

    /**
    * Write line protocol to the batch
    */
    writeApi.writeRecords(lines);

    /**
    * Flush pending writes from the buffer and close the write client.
    **/
    writeApi.close().then(
      () => {
        console.log('Data has been written successfully.');
      },
      (err) => {
        console.log('Error writing line protocol');
      }
    );
    ```

    The sample does the following:

      1.  Calls the `new InfluxDB()` constructor to instantiate a client configured
           with the InfluxDB URL and database token.

      2.  Defines an array of line protocol strings where each string represents a data record.

      3.  Calls the client's `getWriteApi()` method to create a
          **write client** configured to write to the database
          (the method requires an `org` argument, but InfluxDB ignores it).

          **Because the timestamps in the sample line protocol are in second
            precision, the example passes `'s'` for the `precision` option in order
            to set the timestamp precision to seconds**.

      6.  Calls the write client's `writeRecords()` method with the line protocol array
          to write the records in batches to InfluxDB.

      7.  Calls the write client's `close()` method with callback functions for success and error.
          The `close()` method sends any records remaining in the buffer,
          executes callbacks, and releases resources.

4. To execute the file and write the line protocol to your {{% cloud-name %}} database,
    enter the following command in your terminal:
   
    ```sh
    node write.js
    ```
{{% /influxdb/custom-timestamps %}}
<!------------------------------- END NODE.JS CONTENT ------------------------------>
{{% /tab-content %}}
{{% tab-content %}}
<!---------------------------- BEGIN C# CONTENT --------------------------->
{{% influxdb/custom-timestamps %}}
1.  If you haven't already, follow the [Microsoft.com download instructions](https://dotnet.microsoft.com/en-us/download) to install .NET and the `dotnet` CLI.
2. In your terminal, create an executable C# project using the .NET **console** template.

    ```sh
    dotnet new console --name influxdb_csharp_client
    ```

3. Change into the generated `influxdb_csharp_client` directory.
    
    ```sh
    cd influxdb_csharp_client
    ```

4. Run the following command to install the latest version of the InfluxDB v3 C# client library.

    ```sh
    dotnet add package InfluxDB3.Client
    ```

5.  In your editor, create a `Write.cs` file and enter the following sample code:

    ```c#
    // Write.cs

    using System;
    using System.Threading.Tasks;
    using InfluxDB3.Client;
    using InfluxDB3.Client.Query;

    namespace InfluxDBv3;

    public class Write
    {
      /**
        * Writes line protocol to InfluxDB using the C# .NET client
        * library.
        */ 
      public static async Task WriteLines()
      {
        // Set InfluxDB credentials
        const string hostUrl = "https://cloud2.influxdata.com";
        string? database = "get-started";

        /**
          * INFLUX_TOKEN is an environment variable you assigned to your
          * database token value.
          */
        string? authToken = System.Environment
            .GetEnvironmentVariable("INFLUX_TOKEN");

        // Instantiate the InfluxDB client with credentials.
        using var client = new InfluxDBClient(
            hostUrl, authToken: authToken, database: database);

        /** 
          * Define an array of line protocol strings to write.
          * Include an additional backslash to preserve backslashes
          * and prevent interpretation of escape sequences---for example,
          * escaped spaces in tag values.
          */
        string[] lines = new string[] {
              "home,room=Living\\ Room temp=21.1,hum=35.9,co=0i 1641024000",
              "home,room=Kitchen temp=21.0,hum=35.9,co=0i 1641024000",
              "home,room=Living\\ Room temp=21.4,hum=35.9,co=0i 1641027600",
              "home,room=Kitchen temp=23.0,hum=36.2,co=0i 1641027600",
              "home,room=Living\\ Room temp=21.8,hum=36.0,co=0i 1641031200",
              "home,room=Kitchen temp=22.7,hum=36.1,co=0i 1641031200",
              "home,room=Living\\ Room temp=22.2,hum=36.0,co=0i 1641034800",
              "home,room=Kitchen temp=22.4,hum=36.0,co=0i 1641034800",
              "home,room=Living\\ Room temp=22.2,hum=35.9,co=0i 1641038400",
              "home,room=Kitchen temp=22.5,hum=36.0,co=0i 1641038400",
              "home,room=Living\\ Room temp=22.4,hum=36.0,co=0i 1641042000",
              "home,room=Kitchen temp=22.8,hum=36.5,co=1i 1641042000",
              "home,room=Living\\ Room temp=22.3,hum=36.1,co=0i 1641045600",
              "home,room=Kitchen temp=22.8,hum=36.3,co=1i 1641045600",
              "home,room=Living\\ Room temp=22.3,hum=36.1,co=1i 1641049200",
              "home,room=Kitchen temp=22.7,hum=36.2,co=3i 1641049200",
              "home,room=Living\\ Room temp=22.4,hum=36.0,co=4i 1641052800",
              "home,room=Kitchen temp=22.4,hum=36.0,co=7i 1641052800",
              "home,room=Living\\ Room temp=22.6,hum=35.9,co=5i 1641056400",
              "home,room=Kitchen temp=22.7,hum=36.0,co=9i 1641056400",
              "home,room=Living\\ Room temp=22.8,hum=36.2,co=9i 1641060000",
              "home,room=Kitchen temp=23.3,hum=36.9,co=18i 1641060000",
              "home,room=Living\\ Room temp=22.5,hum=36.3,co=14i 1641063600",
              "home,room=Kitchen temp=23.1,hum=36.6,co=22i 1641063600",
              "home,room=Living\\ Room temp=22.2,hum=36.4,co=17i 1641067200",
              "home,room=Kitchen temp=22.7,hum=36.5,co=26i 1641067200"
        };

        // Write each record separately.
        foreach (string line in lines)
        {
          // Write the record to InfluxDB with timestamp precision in seconds.
          await client.WriteRecordAsync(
              record: line, precision: WritePrecision.S);
          Console.WriteLine(
              "Data has been written successfully: {0,-30}", line);
        }
      }
    }
    ```

    The sample does the following:

      1.  Calls the `new InfluxDBClient()` constructor to instantiate a client configured
           with InfluxDB credentials.

          - **hostUrl**: your {{% cloud-name %}} cluster URL
          - **database**: the name of the {{% cloud-name %}} database to write to
          - **authToken**: an [database token](/influxdb/cloud-dedicated/admin/tokens/) with _write_ access to the specified bucket.
          _For security reasons, we recommend setting this as an environment variable rather than including the raw token string._

          _Instantiating the client with the `using` statement ensures that the client is disposed of when it's no longer needed._

      2.  Defines an array of line protocol strings where each string represents a data record.
      3.  Calls the client's `WriteRecordAsync()` method to write each line protocol record to InfluxDB.

          **Because the timestamps in the sample line protocol are in second
          precision, the example passes the [`WritePrecision.S` enum value](https://github.com/InfluxCommunity/influxdb3-csharp/blob/main/Client/Write/WritePrecision.cs)
          to the `precision:` option in order to set the timestamp precision to seconds.**

6.  In your editor, open the `Program.cs` file and replace its contents with the following:

    ```c#
    // Program.cs

    using System;
    using System.Threading.Tasks;

    namespace InfluxDBv3;

    public class Program
    {
      public static async Task Main()
      {
        await Write.WriteLineProtocol();
      }
    }
    ```

    The `Program` class shares the same `InfluxDBv3` namespace as the `Write` class you defined in the preceding step
    and defines a `Main()` function that calls `Write.WriteLineProtocol()`.
    The `dotnet` CLI recognizes `Program.Main()` as the entry point for your program.

7.  To build and execute the program and write the line protocol to your {{% cloud-name %}} database, enter the following commands in your terminal:

    ```sh
    dotnet build
    ```

    ```sh
    dotnet run
    ```
<!---------------------------- END C# CONTENT --------------------------->
{{% /influxdb/custom-timestamps %}}
{{% /tab-content %}}
{{< /tabs-wrapper >}}

If successful, the output is the success message; otherwise, error details and the failure message.

{{< expand-wrapper >}}
{{% expand "View the written data" %}}

{{% influxdb/custom-timestamps %}}
| time                 | room        |  co |  hum | temp |
| :------------------- | :---------- | --: | ---: | ---: |
| 2022-01-01T08:00:00Z | Kitchen     |   0 | 35.9 |   21 |
| 2022-01-01T09:00:00Z | Kitchen     |   0 | 36.2 |   23 |
| 2022-01-01T10:00:00Z | Kitchen     |   0 | 36.1 | 22.7 |
| 2022-01-01T11:00:00Z | Kitchen     |   0 |   36 | 22.4 |
| 2022-01-01T12:00:00Z | Kitchen     |   0 |   36 | 22.5 |
| 2022-01-01T13:00:00Z | Kitchen     |   1 | 36.5 | 22.8 |
| 2022-01-01T14:00:00Z | Kitchen     |   1 | 36.3 | 22.8 |
| 2022-01-01T15:00:00Z | Kitchen     |   3 | 36.2 | 22.7 |
| 2022-01-01T16:00:00Z | Kitchen     |   7 |   36 | 22.4 |
| 2022-01-01T17:00:00Z | Kitchen     |   9 |   36 | 22.7 |
| 2022-01-01T18:00:00Z | Kitchen     |  18 | 36.9 | 23.3 |
| 2022-01-01T19:00:00Z | Kitchen     |  22 | 36.6 | 23.1 |
| 2022-01-01T20:00:00Z | Kitchen     |  26 | 36.5 | 22.7 |
| 2022-01-01T08:00:00Z | Living Room |   0 | 35.9 | 21.1 |
| 2022-01-01T09:00:00Z | Living Room |   0 | 35.9 | 21.4 |
| 2022-01-01T10:00:00Z | Living Room |   0 |   36 | 21.8 |
| 2022-01-01T11:00:00Z | Living Room |   0 |   36 | 22.2 |
| 2022-01-01T12:00:00Z | Living Room |   0 | 35.9 | 22.2 |
| 2022-01-01T13:00:00Z | Living Room |   0 |   36 | 22.4 |
| 2022-01-01T14:00:00Z | Living Room |   0 | 36.1 | 22.3 |
| 2022-01-01T15:00:00Z | Living Room |   1 | 36.1 | 22.3 |
| 2022-01-01T16:00:00Z | Living Room |   4 |   36 | 22.4 |
| 2022-01-01T17:00:00Z | Living Room |   5 | 35.9 | 22.6 |
| 2022-01-01T18:00:00Z | Living Room |   9 | 36.2 | 22.8 |
| 2022-01-01T19:00:00Z | Living Room |  14 | 36.3 | 22.5 |
| 2022-01-01T20:00:00Z | Living Room |  17 | 36.4 | 22.2 |
{{% /influxdb/custom-timestamps %}}

{{% /expand %}}
{{< /expand-wrapper >}}

**Congratulations!** You have written data to InfluxDB.
With data now stored in InfluxDB, let's query it.

{{< page-nav prev="/influxdb/cloud-dedicated/get-started/setup/" next="/influxdb/cloud-dedicated/get-started/query/" keepTab=true >}}
