---
title: Grok input data format
description: Use the grok data format to parse line-delimited data using a regular expression-like language.
menu:
  telegraf_1_22_ref:

    name: Grok
    weight: 40
    parent: Input data formats
---

The grok data format parses line-delimited data using a regular expression-like
language.

For an introduction to grok patterns, see [Grok Basics](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html#_grok_basics)
in the Logstash documentation. The grok parser uses a slightly modified version of logstash "grok"
patterns, using the format:

```
%{<capture_syntax>[:<semantic_name>][:<modifier>]}
```

The `capture_syntax` defines the grok pattern that is used to parse the input
line and the `semantic_name` is used to name the field or tag.  The extension
`modifier` controls the data type that the parsed item is converted to or
other special handling.

By default, all named captures are converted into string fields.
Timestamp modifiers can be used to convert captures to the timestamp of the
parsed metric.  If no timestamp is parsed the metric will be created using the
current time.

You must capture at least one field per line.

- Available modifiers:
  - string   (default if nothing is specified)
  - int
  - float
  - duration (ie, 5.23ms gets converted to int nanoseconds)
  - tag      (converts the field into a tag)
  - drop     (drops the field completely)
  - measurement (use the matched text as the measurement name)
- Timestamp modifiers:
  - ts               (This will auto-learn the timestamp format)
  - ts-ansic         ("Mon Jan _2 15:04:05 2006")
  - ts-unix          ("Mon Jan _2 15:04:05 MST 2006")
  - ts-ruby          ("Mon Jan 02 15:04:05 -0700 2006")
  - ts-rfc822        ("02 Jan 06 15:04 MST")
  - ts-rfc822z       ("02 Jan 06 15:04 -0700")
  - ts-rfc850        ("Monday, 02-Jan-06 15:04:05 MST")
  - ts-rfc1123       ("Mon, 02 Jan 2006 15:04:05 MST")
  - ts-rfc1123z      ("Mon, 02 Jan 2006 15:04:05 -0700")
  - ts-rfc3339       ("2006-01-02T15:04:05Z07:00")
  - ts-rfc3339nano   ("2006-01-02T15:04:05.999999999Z07:00")
  - ts-httpd         ("02/Jan/2006:15:04:05 -0700")
  - ts-epoch         (seconds since unix epoch, may contain decimal)
  - ts-epochnano     (nanoseconds since unix epoch)
  - ts-syslog        ("Jan 02 15:04:05", parsed time is set to the current year)
  - ts-"CUSTOM"

CUSTOM time layouts must be within quotes and be the representation of the
"reference time", which is `Mon Jan 2 15:04:05 -0700 MST 2006`.
To match a comma decimal point you can use a period.  For example `%{TIMESTAMP:timestamp:ts-"2006-01-02 15:04:05.000"}` can be used to match `"2018-01-02 15:04:05,000"`
To match a comma decimal point you can use a period in the pattern string.
See https://golang.org/pkg/time/#Parse for more details.

Telegraf has many of its own [built-in patterns](https://github.com/influxdata/telegraf/blob/master/plugins/parsers/grok/influx_patterns.go),
as well as support for most of
[Logstash's core patterns](https://github.com/logstash-plugins/logstash-patterns-core/blob/main/patterns/ecs-v1/grok-patterns).
_Golang regular expressions do not support lookahead or lookbehind.
Logstash patterns that depend on these are not supported._

For help building and testing patterns, see [tips for creating patterns](#tips-for-creating-patterns).

## Configuration

```toml
[[inputs.file]]
  ## Files to parse each interval.
  ## These accept standard unix glob matching rules, but with the addition of
  ## ** as a "super asterisk". ie:
  ##   /var/log/**.log     -> recursively find all .log files in /var/log
  ##   /var/log/*/*.log    -> find all .log files with a parent dir in /var/log
  ##   /var/log/apache.log -> only tail the apache log file
  files = ["/var/log/apache/access.log"]

  ## The dataformat to be read from files
  ## Each data format has its own unique set of configuration options, read
  ## more about them here:
  ## https://github.com/influxdata/telegraf/blob/master/docs/DATA_FORMATS_INPUT.md
  data_format = "grok"

  ## This is a list of patterns to check the given log file(s) for.
  ## Note that adding patterns here increases processing time. The most
  ## efficient configuration is to have one pattern.
  ## Other common built-in patterns are:
  ##   %{COMMON_LOG_FORMAT}   (plain apache & nginx access logs)
  ##   %{COMBINED_LOG_FORMAT} (access logs + referrer & agent)
  grok_patterns = ["%{COMBINED_LOG_FORMAT}"]

  ## Full path(s) to custom pattern files.
  grok_custom_pattern_files = []

  ## Custom patterns can also be defined here. Put one pattern per line.
  grok_custom_patterns = '''
  '''

  ## Timezone allows you to provide an override for timestamps that
  ## don't already include an offset
  ## e.g. 04/06/2016 12:41:45 data one two 5.43µs
  ##
  ## Default: "" which renders UTC
  ## Options are as follows:
  ##   1. Local             -- interpret based on machine localtime
  ##   2. "Canada/Eastern"  -- Unix TZ values like those found in https://en.wikipedia.org/wiki/List_of_tz_database_time_zones
  ##   3. UTC               -- or blank/unspecified, will return timestamp in UTC
  grok_timezone = "Canada/Eastern"
```

### Timestamp examples

This example input and config parses a file using a custom timestamp conversion:

```
2017-02-21 13:10:34 value=42
```

```toml
[[inputs.file]]
  grok_patterns = ['%{TIMESTAMP_ISO8601:timestamp:ts-"2006-01-02 15:04:05"} value=%{NUMBER:value:int}']
```

This example input and config parses a file using a timestamp in unix time:

```
1466004605 value=42
1466004605.123456789 value=42
```

```toml
[[inputs.file]]
  grok_patterns = ['%{NUMBER:timestamp:ts-epoch} value=%{NUMBER:value:int}']
```

This example parses a file using a built-in conversion and a custom pattern:

```
Wed Apr 12 13:10:34 PST 2017 value=42
```

```toml
[[inputs.file]]
  grok_patterns = ["%{TS_UNIX:timestamp:ts-unix} value=%{NUMBER:value:int}"]
  grok_custom_patterns = '''
    TS_UNIX %{DAY} %{MONTH} %{MONTHDAY} %{HOUR}:%{MINUTE}:%{SECOND} %{TZ} %{YEAR}
  '''
```

For cases where the timestamp itself is without offset, the `timezone` config var is available
to denote an offset. By default (with `timezone` either omit, blank or set to `"UTC"`), the times
are processed as if in the UTC timezone. If specified as `timezone = "Local"`, the timestamp
will be processed based on the current machine timezone configuration. Lastly, if using a
timezone from the list of Unix [timezones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones),
grok will offset the timestamp accordingly.

### TOML escaping

When saving patterns to the configuration file, keep in mind the different TOML
[string](https://github.com/toml-lang/toml#string) types and the escaping
rules for each.  These escaping rules must be applied in addition to the
escaping required by the grok syntax.  Using the TOML multi-line literal
syntax (`'''`) may be useful.

The following config examples will parse this input file:

```
|42|\uD83D\uDC2F|'telegraf'|
```

Since `|` is a special character in the grok language, we must escape it to
get a literal `|`.  With a basic TOML string, special characters such as
backslash must be escaped, requiring us to escape the backslash a second time.

```toml
[[inputs.file]]
  grok_patterns = ["\\|%{NUMBER:value:int}\\|%{UNICODE_ESCAPE:escape}\\|'%{WORD:name}'\\|"]
  grok_custom_patterns = "UNICODE_ESCAPE (?:\\\\u[0-9A-F]{4})+"
```

We cannot use a literal TOML string for the pattern, because we cannot match a
`'` within it.  However, it works well for the custom pattern.
```toml
[[inputs.file]]
  grok_patterns = ["\\|%{NUMBER:value:int}\\|%{UNICODE_ESCAPE:escape}\\|'%{WORD:name}'\\|"]
  grok_custom_patterns = 'UNICODE_ESCAPE (?:\\u[0-9A-F]{4})+'
```

A multi-line literal string allows us to encode the pattern:
```toml
[[inputs.file]]
  grok_patterns = ['''
    \|%{NUMBER:value:int}\|%{UNICODE_ESCAPE:escape}\|'%{WORD:name}'\|
  ''']
  grok_custom_patterns = 'UNICODE_ESCAPE (?:\\u[0-9A-F]{4})+'
```

### Tips for creating patterns
Complex patterns can be difficult to read and write.
For help building and debugging grok patterns, see the following tools:
- [Grok Constructor](https://grokconstructor.appspot.com/)
- [Grok Debugger](https://grokdebugger.com/)

We recommend the following steps for building and testing a new pattern with Telegraf and your data:

1. In your Telegraf configuration, do the following to help you isolate and view the captured metrics:
    - Configure a file output that writes to stdout:

      ```toml
      [[outputs.file]]
        files = ["stdout"]
      ```

    - Disable other outputs while testing.

    *Keep in mind that the file output will only print once per `flush_interval`.*

2. For the input, start with a sample file that contains a single line of your data,
   and then remove all but the first token or piece of the line.
3. In your Telegraf configuration, add the section of your pattern that matches the piece of data from the previous step.
4. Run Telegraf and verify that the metric is parsed successfully.
5. If successful, add the next token to the data file, update the pattern configuration in Telegraf, and then retest.
6. Continue one token at a time until the entire line is successfully parsed.

