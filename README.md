# metric-stream

A streaming metric producer. Allows producing counters, gauges, time series in a way that is independent of your metrics system so that you can produce metrics and let consumers decide how to consume them. Additionally, you can pipe together different metrics streams before finally consuming them all in a single location.

## Usage

The client is intended to be used in the following way:

### Step 1.

Instantiate a new client

```js
const client = new MetricStream();
```

### Step 2.

Use the client for instrumentation

```js
client.metric({
    name: 'unique_metic_name',
    description: 'Description of metric being collected',
    value: 10,
});
```

### Step 3.

Pipe collected metrics to a collector

```js
client.pipe(consumer);
```

The client supports 2 types of metric creation use cases. 1. timers (via `client.timer`) and 2. Non timers (via `client.metric`)

-   Histograms are supported by using the timer functionality.
-   Gauges are supported via setting the `value` property.
-   Counters are implicit. ie. Each metric added can be counted by a consumer.
-   Timestamps are always generated by default for all metrics.
-   By default the client will only buffer up 100 metrics items before it starts dropping the oldest metric from the stream for each new metric it adds. This should prevent any memory leaks associated with not consuming the stream.

## Examples

_A metric to be used in a counter_

```js
client.metric({
    name: 'my_counter',
    description: 'Counter description',
});
```

_Setting a value to be used in a gauge_

```js
client.metric({
    name: 'my_gauge',
    description: 'Gauge description',
    value: 123,
});
```

_A timer to be used in a histogram_

```js
const end = client.timer({
    name: 'my_histogram',
    description: 'Histogram description'
});
await something();
end();
```

_A timer to be used in a histogram and a gauge_

```js
const end = client.timer({
    name: 'my_histogram_gauge',
    description: 'Histogram gauge example'
});
await something();
end({ value: 123 });
```

## API

### new MetricStream(options)

Creates a new instance of the metrics client.

**options**

| name        | description                                                         | type     | default |
| ----------- | ------------------------------------------------------------------- | -------- | ------- |
| `maxBuffer` | Max number of metrics to retain if not consumed. Keeps most recent. | `number` | 100     |

_Example_

```js
const client = new MetricStream(options);
```

**return**: `Duplex Stream`

### instance methods

#### .metric(options)

Collects a metric. As a minimum, a name and description for the metric must be provided.

**options**

| name          | description                                      | type            | default | required |
| ------------- | ------------------------------------------------ | --------------- | ------- | -------- |
| `name`        | Metric name. valid characters: a-z,A-Z,0-9,\_    | `string`        | null    | `true`   |
| `description` | Metric description                               | `string`        | null    | `true`   |
| `value`       | Arbitrary value for the metric (used for gauges) | `string|number` | null    | `false`  |
| `meta`        | Available to be used to hold any misc data.      | `object`        | null    | `false`  |

**n.b.** In practice, `meta` can be used as a way to label metrics. Use each key of the meta object as the label name and the value as the label value

**return**: `void`

```js
client.metric({
    name: '',
    description: '',
});
```

#### .timer(options)

Starts a metric timer and returns and end function to be called when the measurement should be considered finished.

**options**

| name          | description                                      | type            | default | required |
| ------------- | ------------------------------------------------ | --------------- | ------- | -------- |
| `name`        | Metric name. valid characters: a-z,A-Z,0-9,\_    | `string`        | null    | `true`   |
| `description` | Metric description                               | `string`        | null    | `true`   |
| `value`       | Arbitrary value for the metric (used for gauges) | `string|number` | null    | `false`  |
| `meta`        | Available to be used to hold any misc data       | `object`        | null    | `false`  |

**n.b.** In practice, `meta` can be used as a way to label metrics. Use each key of the meta object as the label name and the value as the label value

**return**: `function` Returns an end function (see below) to be used to indicate that the timer measurement is finished.

_Example_

```js
const end = client.timer(options);
```

##### .end(options)

Stops a previously started timer, merges timers `options` with end `options` and and sets the measured `time` value.

**options**

| name          | description                                      | type            | default | required |
| ------------- | ------------------------------------------------ | --------------- | ------- | -------- |
| `name`        | Metric name. valid characters: a-z,A-Z,0-9,\_    | `string`        | null    | `true`   |
| `description` | Metric description                               | `string`        | null    | `true`   |
| `value`       | Arbitrary value for the metric (used for gauges) | `string|number` | null    | `false`  |
| `meta`        | Available to be used to hold any misc data       | `object`        | null    | `false`  |

**n.b.** In practice, `meta` can be used as a way to label metrics. Use each key of the meta object as the label name and the value as the label value

**return**: `void`

_Example_

```js
const end = client.timer(options);
// ... thing to be measured
end(options);
```