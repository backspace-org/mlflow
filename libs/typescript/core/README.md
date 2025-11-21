# MLflow Typescript SDK - Core

This is the core package of the [MLflow Typescript SDK](https://github.com/mlflow/mlflow/tree/main/libs/typescript). It is a skinny package that includes the core tracing functionality and manual instrumentation.

| Package              | NPM                                                                                                                           | Description                                                |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| [mlflow-tracing](./) | [![npm package](https://img.shields.io/npm/v/mlflow-tracing?style=flat-square)](https://www.npmjs.com/package/mlflow-tracing) | The core tracing functionality and manual instrumentation. |

## Installation

### Stable release (npm)

```bash
npm install mlflow-tracing
```

### Preview build (Git branch)

While the new shared-tracer-provider + OTLP work is incubating, you can install
directly from the Backspace fork:

```bash
npm install github:backspace-org/mlflow#ts-otlp-sdk-preview
```

The package ships TypeScript sources and a `prepare` script that compiles `dist/`
during `npm install`, so no additional build steps are required.

## Quickstart

Start MLflow Tracking Server. If you have a local Python environment, you can run the following command:

```bash
pip install mlflow
mlflow server --backend-store-uri sqlite:///mlruns.db --port 5000
```

If you don't have Python environment locally, MLflow also supports Docker deployment or managed services. See [Self-Hosting Guide](https://mlflow.org/docs/latest/self-hosting/index.html) for getting started.

Instantiate MLflow SDK in your application:

```typescript
import * as mlflow from 'mlflow-tracing';

mlflow.init({
  trackingUri: process.env.MLFLOW_TRACKING_URI,
  experimentId: process.env.MLFLOW_EXPERIMENT_ID
});
```

Create a trace:

```typescript
// Wrap a function with mlflow.trace to generate a span when the function is called.
// MLflow will automatically record the function name, arguments, return value,
// latency, and exception information to the span.
const getWeather = mlflow.trace(
  (city: string) => {
    return `The weather in ${city} is sunny`;
  },
  // Pass options to set span name. See https://mlflow.org/docs/latest/genai/tracing/app-instrumentation/typescript-sdk
  // for the full list of options.
  { name: 'get-weather' }
);
getWeather('San Francisco');

// Alternatively, start and end span manually
const span = mlflow.startSpan({ name: 'my-span' });
span.end();
```

## Environment configuration

At minimum set the following before your app process starts:

| Variable                  | Description                                                                                         |
|---------------------------|-----------------------------------------------------------------------------------------------------|
| `MLFLOW_TRACKING_URI`     | Base URL of your MLflow tracking server (e.g. `http://localhost:5001` or `databricks`).             |
| `MLFLOW_EXPERIMENT_ID`    | Experiment that should receive traces.                                                              |
| `OTEL_RESOURCE_ATTRIBUTES`| Optional resource metadata (comma-delimited `key=value`) stamped on every span/trace.               |
| `OTEL_SERVICE_NAME`       | Optional override for the OpenTelemetry resource service name.                                      |

### Optional: dual-export to OTLP

To mirror the Python SDK’s behavior and forward traces to an OTLP collector:

| Variable                              | Description                                                                                                       |
|---------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| `MLFLOW_ENABLE_OTLP_EXPORTER`         | Enable the OTLP exporter (`true`/`false`, defaults to `true`).                                                    |
| `MLFLOW_TRACE_ENABLE_OTLP_DUAL_EXPORT`| When `true`, keep sending traces to MLflow **and** OTLP. When `false`, OTLP replaces MLflow.                      |
| `OTEL_EXPORTER_OTLP_TRACES_ENDPOINT`  | OTLP traces endpoint (e.g. `http://localhost:4318/v1/traces`).                                                    |
| `OTEL_EXPORTER_OTLP_HEADERS`          | Optional comma-delimited headers such as `Authorization=Bearer <token>`.                                          |
| `OTEL_EXPORTER_OTLP_PROTOCOL` / `OTEL_EXPORTER_OTLP_TRACES_PROTOCOL` | Set to `http/protobuf` (default) or `grpc`.                                   |

When these are configured the provider automatically wires a `BatchSpanProcessor`
with `@opentelemetry/exporter-trace-otlp-proto` so every span is emitted as a
standard OTLP protobuf payload in addition to the MLflow REST export.

## Documentation 📘

Official documentation for MLflow Typescript SDK can be found [here](https://mlflow.org/docs/latest/genai/tracing/app-instrumentation/typescript-sdk).

## License

This project is licensed under the [Apache License 2.0](https://github.com/mlflow/mlflow/blob/master/LICENSE.txt).
