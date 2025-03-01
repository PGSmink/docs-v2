---
title: Use Python to query data with InfluxQL
description: >
  Use Python and the `influxdb3-python` library to query data stored in InfluxDB
  with InfluxQL.
weight: 401
menu:
  influxdb_cloud_serverless:
    parent: influxql-execute-queries
    name: Use Python
    identifier: query-with-python-influxql
influxdb/cloud-serverless/tags: [query, influxql, python]
related:
    - /influxdb/cloud-serverless/process-data/tools/pandas/
    - /influxdb/cloud-serverless/process-data/tools/pyarrow/
    - /influxdb/cloud-serverless/reference/influxql/
list_code_example: |
    ```py
    from influxdb_client_3 import InfluxDBClient3

    # Instantiate an InfluxDB client
    client = InfluxDBClient3(
        host='cloud2.influxdata.com',
        org='ORG_NAME',
        token='API_TOKEN',
        database='DATABASE_NAME'
    )

    # Execute the query and return an Arrow table
    table = client.query(
        query="SELECT * FROM home",
        language="influxql"
    )

    # Return query results as a markdown table
    print(table.to_pandas().to_markdown())
    ```
---

Use the `influxdb3-python` client library to query data stored in InfluxDB with InfluxQL.
The `influxdb3-client` uses InfluxDB v3's Flight RPC protocol to query data from InfluxDB and return
results in Apache Arrow Table format.

- [Get started using Python to query InfluxDB](#get-started-using-python-to-query-influxdb)
- [Create a Python virtual environment](#create-a-python-virtual-environment)
  - [Install Python](#install-python)
  - [Create a project virtual environment](#venv-install)
  - [Install Anaconda](?t=Anaconda#conda-install)
- [Query InfluxDB](#query-influxdb)
  - [Install the influxdb3-python library](#install-the-influxdb3-python-library)
  - [Create an InfluxDB client](#create-an-influxdb-client)
  - [Execute a query](#execute-a-query)

{{% note %}}
#### Databases and retention policies map to InfluxDB buckets

InfluxQL **databases** and **retention policies** are used to route queries to
an InfluxDB **bucket** based on database and retention policy (DBRP) mappings.
For more information, see
[Map databases and retention policies to buckets](/influxdb/cloud-serverless/query-data/influxql/dbrp/).
{{% /note %}}

{{% warn %}}
#### InfluxQL feature support

InfluxQL is being rearchitected to work with the InfluxDB IOx storage engine.
This process is ongoing and some InfluxQL features are still being implemented.
For information about the current implementation status of InfluxQL features,
see [InfluxQL feature support](/influxdb/cloud-serverless/reference/influxql/feature-support/).
{{% /warn %}}

## Get started using Python to query InfluxDB

This guide follows the recommended practice of using Python _virtual environments_.
If you don't want to use virtual environments and you have Python installed,
continue to [Query InfluxDB](#query-influxdb).

## Create a Python virtual environment

Python [virtual environments](https://docs.python.org/3/library/venv.html) keep
the Python interpreter and dependencies for your project self-contained and isolated from other projects.

To install Python and create a virtual environment, choose one of the following options:

- [Python venv](?t=venv#venv-install): The [`venv` module](https://docs.python.org/3/library/venv.html) comes standard in Python as of version 3.5.
- [Anaconda® Distribution](?t=Anaconda#conda-install): A Python/R data science distribution that provides Python and the  **conda** package and environment manager. 

    {{< tabs-wrapper >}}
{{% tabs "small" %}}
[venv]()
[Anaconda]()
{{% /tabs %}}
{{% tab-content %}}
<!--------------------------------- Begin venv -------------------------------->

### Install Python

1. Follow the [Python installation instructions](https://wiki.python.org/moin/BeginnersGuide/Download)
to install a recent version of the Python programming language for your system.
2. Check that you can run `python` and `pip` commands.
`pip` is a package manager included in most Python distributions.

    In your terminal, enter the following commands:

    ```sh
    python --version
    ```

    ```sh
    pip --version
    ```

    Depending on your system, you may need to use version-specific commands--for example.

    ```sh
    python3 --version
    ```

    ```sh
    pip3 --version
    ```

    If neither `pip` nor `pip<PYTHON_VERSION>` works, follow one of the [Pypa.io Pip installation](https://pip.pypa.io/en/stable/installation/) methods for your system.

### Create a project virtual environment {#venv-install}

1. Create a directory for your Python project and change to the new directory--for example:

    ```sh
    mkdir ./PROJECT_DIRECTORY && cd $_
    ```

2. Use the Python `venv` module to create a virtual environment--for example:
    
    ```sh
    python -m venv envs/virtualenv-1
    ```

   `venv` creates the new virtual environment directory in your project.
   
3. To activate the new virtual environment in your terminal, run the `source` command and pass the file path of the virtual environment `activate` script:

    ```sh
    source envs/VIRTUAL_ENVIRONMENT_NAME/bin/activate
    ```

    For example:
    
    ```sh
    source envs/virtualenv-1/bin/activate
    ```
<!---------------------------------- End venv --------------------------------->
{{% /tab-content %}}
{{% tab-content %}}
<!-------------------------------- Begin conda -------------------------------->

### Install Anaconda {#conda-install}

1. Follow the [Anaconda installation instructions](https://docs.continuum.io/anaconda/install/) for your system.
2. Check that you can run the `conda` command:

    ```sh
    conda
    ```

3. Use `conda` to create a virtual environment--for example:

    ```sh
    conda create --prefix envs/virtualenv-1 
    ```

    `conda` creates a virtual environment in a directory named `./envs/virtualenv-1`.

4. To activate the new virtual environment, use the `conda activate` command and pass the directory path of the virtual environment:

    ```sh
    conda activate envs/VIRTUAL_ENVIRONMENT_NAME
    ```

    For example:

    ```sh
    conda activate ./envs/virtualenv-1
    ```

<!--------------------------------- END conda --------------------------------->
{{% /tab-content %}}
    {{< /tabs-wrapper >}}

When a virtual environment is activated, the name displays at the beginning of your terminal command line--for example:
  
{{% code-callout "(virtualenv-1)"%}}
```sh
(virtualenv-1) $ PROJECT_DIRECTORY
```
{{% /code-callout %}}

## Query InfluxDB

1. [Install the influxdb3-python library](#install-the-influxdb3-python-library)
2. [Create an InfluxDB client](#create-an-influxdb-client)
3. [Execute a query](#execute-a-query)

### Install the influxdb3-python library

The `influxdb_client_3` module provides a simple and convenient way to interact
with {{< cloud-name >}} using Python. This module supports both writing data to
InfluxDB and querying data using SQL or InfluxQL queries.

Installing `inflxudb3-python` also installs the
[`pyarrow`](https://arrow.apache.org/docs/python/index.html) library that you'll
use for working with Arrow data returned from queries.

In your terminal, use `pip` to install `influxdb3-python`:

```sh
pip install influxdb3-python
```

With `influxdb3-python` and `pyarrow` installed, you're ready to query and
analyze data stored in an InfluxDB database.

### Create an InfluxDB client

The following example shows how to use Python with `influxdb3-python`
and to instantiate a client configured for an InfluxDB database.

In your editor, copy and paste the following sample code to a new file--for
example, `query-example.py`:

{{% code-placeholders "(DATABASE|ORG|API)_(NAME|TOKEN)" %}}
```py
# query-example.py

from influxdb_client_3 import InfluxDBClient3

# Instantiate an InfluxDBClient3 client configured for your database
client = InfluxDBClient3(
    host='cloud2.influxdata.com',
    org='ORG_NAME',
    token='API_TOKEN',
    database='DATABASE_NAME'
)
```
{{% /code-placeholders %}}

Replace the following configuration values:

- {{% code-placeholder-key %}}`API_TOKEN`{{% /code-placeholder-key %}}:
  Your InfluxDB [token](/influxdb/cloud-serverless/admin/tokens/) with read permissions on the databases you want to query.
- {{% code-placeholder-key %}}`DATABASE_NAME`{{% /code-placeholder-key %}}:
  The name of your InfluxDB database.

### Execute a query

To execute an InfluxQL query, call the client's `query(query,language)` method
and specify the following arguments:

- **query**: InfluxQL query string to execute.
- **language**: `influxql`

#### Syntax {#execute-query-syntax}

```py
query(query: str, language: str)
```

#### Example {#execute-query-example}

{{% code-placeholders "(DATABASE|ORG|API)_(NAME|TOKEN)" %}}
```py
# query-example.py

from influxdb_client_3 import InfluxDBClient3

client = InfluxDBClient3(
    host='cloud2.influxdata.com',
    org='ORG_NAME',
    token='API_TOKEN',
    database='DATABASE_NAME'
)

# Execute the query and return an Arrow table
table = client.query(
    query="SELECT * FROM home",
    language="influxql"
)

# Return query results as a markdown table
print(table.to_pandas().to_markdown())
```
{{% /code-placeholders %}}

Next, learn how to use Python tools to work with time series data:

- [Use PyArrow](/influxdb/cloud-serverless/process-data/tools/pyarrow/)
- [Use pandas](/influxdb/cloud-serverless/process-data/tools/pandas/)
