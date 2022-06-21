# : human-friendly OpenTelemetry CLI

Provides a CLI for crafting and sending telemetry data over OTLP (OpenTelemetry Line Protocol).

## Requirements

## Installation

There are several ways of running this CLI.

### Docker

```sh
docker pull afeoscyc-mw.cec.lab.emc.com/otel-cli-python:<version>
```

You can specify a version like `0.0.1` or use `latest` to get the most up-to-date version.

Run latest version of otel-cli in a container:

```sh
# set OTEL_EXPORTER_OTLP_ENDPOINT to your OTel collector instance
export OTEL_EXPORTER_OTLP_ENDPOINT=http://127.0.0.1:4317
docker run --rm -e OTEL_EXPORTER_OTLP_ENDPOINT afeoscyc-mw.cec.lab.emc.com/otel-cli-python:latest --help
```

Replace `--help` with any `otel-cli` command, without `otel-cli` itself.

### PyPI

TBD

## Usage

First, define `OTEL_EXPORTER_OTLP_ENDPOINT` in your shell and set it to the OTLP collector instance you want to use.
For a local collector, set this to `http://127.0.0.1:4317` like so:

```sh
export OTEL_EXPORTER_OTLP_ENDPOINT=http://127.0.0.1:4317
```

To send a span, run:

```sh
otel-cli span "span name"
```

To set a different service name, use the `--service` flag:

```sh
otel-cli span --service "My Service" "span name"
```

You can also pass custom start and end dates. These should be *nanoseconds* since the epoch:

```sh
SPAN_START_DATE=$(date --date "2 minutes ago" +%s%N)
SPAN_END_DATE=$(date +%s%N)
otel-cli span --start "$SPAN_START_DATE" --end "$SPAN_END_DATE" "span name"
```

By default, spans are reported with a status of `UNKNOWN`. To pass a different status, use the `--status` option:

```sh
otel-cli span --status OK "successful span"
otel-cli span --status ERROR "failed span"
```

To add attributes to spans, use the `--attribute|-a` option. It accepts attributes in a `key=value` format. Use multiple instances of this option to send multiple attributes:

```sh
otel-cli span -a "my.foo=bar" -a "my.bar=baz" "span name"
```

otel-cli will create a random trace ID and span ID. You can override those:

```sh
otel-cli span --trace-id "4d999706756fd1859345f8dc6d0af218" --span-id "ac2a3b2b19ac602d"
```

### Sending multiple spans in a trace

To create a single trace with one root span and multiple child spans, we first need to generate a trace ID for the entire trace and a span ID for the parent span. Use `otel-cli generate` to create those:

```sh
TRACE_ID=$(otel-cli generate trace_id)
PARENT_SPAN=$(otel-cli generate span_id)
```

Then, when creating children span, we pass this information in the format of a `TRACEPARENT`:

```sh
TRACEPARENT="00-${TRACE_ID}-${PARENT_SPAN}-01"
otel-cli span --traceparent "$TRACEPARENT" "Child A Name"
otel-cli span --traceparent "$TRACEPARENT" "Child B Name"
```

Finally, send the parent span using the pre-generated IDs:

```sh
otel-cli span --trace-id "$TRACE_ID" --span-id "$PARENT_SPAN" "Parent Span Name"
```
