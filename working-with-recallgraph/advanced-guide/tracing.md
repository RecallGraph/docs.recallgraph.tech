---
description: Tracing and instrumentation based on OpenTracing.
---

# Tracing

RecallGraph supports distributed tracing in its HTTP API, based on the [OpenTracing](https://opentracing.io/) standard.

A suite of libraries, services and plugins has been created specifically to allow Foxx microservices to be traced using the OpenTracing specification. These include a [client library,](https://github.com/RecallGraph/foxx-tracer) a [trace collector](https://github.com/RecallGraph/foxx-tracer-collector) and a number of [reporter plugins](https://www.npmjs.com/search?q=foxx-tracer-reporter) to push traces from the collector to different destinations.

These projects are well documented, and should be sufficient to help you get started with tracing RecallGraph \(or any other trace-enabled foxx microservice\). To get started, you can begin with the [foxx-tracer documentation](https://github.com/RecallGraph/foxx-tracer#quickstart). Some of those instructions are repeated here for quick reference.

## Runtime Setup

1. Install the [collector service](https://github.com/RecallGraph/foxx-tracer-collector) and follow its setup instructions. **Add any reporter plugins that you need.**
2. In RecallGraph's settings, there is a param named `sampling-probability`. You can set this to a value between 0 and 1 \(both inclusive\) to tell the tracer how often to record a trace for incoming requests. For example, if `sampling-probability = 0.1`, then roughly 1 out of 10 requests will be traced.  Regardless of this param's value, a trace can be **force-recorded or force-suppressed** using the [`x-force-sample`](https://recallgraph.github.io/foxx-tracer/enums/_helpers_types_.trace_header_keys.html#force_sample) header parameter. See [Recording Traces](https://github.com/RecallGraph/foxx-tracer#recording-traces) for details.

## Recording Traces

For all HTTP endpoints, there are 4 trace-specific headers available that can be used for the following:

1. Propagate a running trace from the client to your application.
2. Force the application to record or suppress a trace for the request, regardless of the `sampling-probability` setting.

The headers \(all optional\) are as follows:

<table>
  <thead>
    <tr>
      <th style="text-align:left">Header</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>x-baggage</code>
      </td>
      <td style="text-align:left">A JSON object containing key-value pairs that will set as the <a href="https://opentracing.io/specification/#set-a-baggage-item">baggage</a> for
        all spans recorded for this request.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>x-force-sample</code>
      </td>
      <td style="text-align:left">An optional boolean that control whether the decision to record a trace
        should be forced, suppressed or be left to the application to decide. If <code>true</code> a
        sample is forced. If <code>false</code> no sample is taken. If left blank,
        the application decides based on the <code>sampling-probability</code> configuration
        parameter.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>x-parent-span-id</code>
      </td>
      <td style="text-align:left">A span ID (belonging to an ongoing trace) under which to create the top
        level span of the traced request. This header <b>must be accompanied</b> by
        a non-emtpy <a href="https://recallgraph.github.io/foxx-tracer/enums/_helpers_types_.trace_header_keys.html#trace_id">TRACE_ID</a> header.
        All spans generated with the application will now have this span ID as
        an ancestor.</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>x-trace-id</code>
      </td>
      <td style="text-align:left">
        <p>The trace ID under which to record all new spans. If unspecified, a new
          trace is started and is assigned a randomly generated <a href="https://recallgraph.github.io/foxx-tracer/modules/_helpers_utils_.html#generateuuid">UUID</a>.</p>
        <p>Note that if a new trace is started by <em>foxx-tracer</em>, the subsequent
          root span&apos;s span ID will <b>not</b> be same as the generated trace ID.</p>
      </td>
    </tr>
  </tbody>
</table>

