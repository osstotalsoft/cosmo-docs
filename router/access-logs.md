---
description: Configuring Access Logs for Enhanced Traffic Monitoring.
icon: camera-cctv
---

# Access logs

{% hint style="info" %}
Available since Router [0.118.0](https://github.com/wundergraph/cosmo/releases/tag/router%400.118.0)
{% endhint %}

Access or request logs provide valuable insights into the traffic passing through your router instances. By default, logs are output to `stdout`, but logging to a file is also supported, which is recommended for high-load scenarios. However, only one logging mode—either `stdout` or file—can be enabled at any given time. The logs include a predefined set of fields that help identify HTTP requests. Additionally, custom fields can be configured to capture more detailed information from the GraphQL requests.

### Logging to stdout

The following configuration enables logging to `stdout`:

```yaml
access_logs:
  enabled: true
  output: 
    stdout:
      enabled: true
```

### Logging to file

For environments with high traffic, logging to a file is recommended. Below is an example configuration for logging to a file:

```yaml
access_logs:
  enabled: true
  output:
    file:
      enabled: true
      path: /var/log/gateway/access.log
```

### Default Log Fields

The default log fields provide essential details about each HTTP request. These fields include:

* **hostname**: The hostname of the machine handling the request.
* **pid**: The process ID of the router instance.
* **request\_id**: The request ID generated by the router instance.
* **trace\_id**: The Trace ID propagated or generated by the router instance.
* **status**: The HTTP status code of the request.
* **method**: The HTTP method used (e.g., `GET`, `POST`).
* **path**: The request path (e.g., `/graphql`).
* **query**: The query string included in the request URL (e.g., `/?param=1`).
* **ip**: The client’s IP address (redacted by default).
* **user\_agent**: The `User-Agent` header value provided by the client.
* **config\_version**: The version of the router configuration used to handle the request.
* **latency**: The time taken to process the request, in float seconds.
* **log\_type**: Identifies the type of log (useful for distinguishing access logs from other log types).

#### Example Log Entry

An example of a log entry is shown below:

```json
12:21:37 PM INFO /graphql {
  "hostname": "dustins-MacBook-Pro.local",
  "pid": 13616,
  "log_type": "request",
  "method": "POST",
  "path": "/graphql",
  "query": "",
  "ip": "[REDACTED]",
  "user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36",
  "config_version": "6c6b3f36-99b9-4632-84e1-ca24d95eee8d",
  "trace_id": "cc4bccf2e981fecc3f8f3b87257c1ce6",
  "latency": 0.019332208,
  "status": 200,
  "request_id": "dustins-MacBook-Pro.local/TBOXjJsAo5-000007"
}
```

### Custom Fields

In addition to the default fields, you can extend the log with custom fields. These can include values extracted from request headers or additional information generated after the GraphQL operation has been processed.

Let's take the following example:

```yaml
log_level: info
dev_mode: true
access_logs:
  enabled: true
  output:
    file:
      enabled: true
      path: "access.log"
  fields:
    - key: "service"
      value_from:
        request_header: "x-service"
        
    - key: "operationName"
      value_from:
        context_field: operation_name
        
    - key: "operationSha256"
      value_from:
        context_field: operation_sha256
        
    - key: "responseErrorMessage"
      value_from:
        context_field: response_error_message
        
    - key: "serviceNames"
      value_from:
        context_field: operation_service_names
        
    - key: "operationHash"
      value_from:
        context_field: operation_hash
        
    - key: "errorCodes"
      value_from:
        context_field: graphql_error_codes
        
    - key: "operationType"
      value_from:
        context_field: operation_type
        
    - key: "persistentOperationSha256"
      value_from:
        context_field: persisted_operation_sha256
        
    - key: "errorServiceNames"
      value_from:
        context_field: graphql_error_service_names
        
    - key: "operationParsingTime"
      value_from:
        context_field: operation_parsing_time
        
    - key: "operationPlanningTime"
      value_from:
        context_field: operation_planning_time
        
    - key: "operationNormalizationTime"
      value_from:
        context_field: operation_normalization_time
        
    - key: "operationValidationTime"
      value_from:
        context_field: operation_validation_time
```

Here’s an explanation of each field’s meaning:

* **service**: This field logs the name of the service making the request, extracted from the request header "x-service". It helps identify which specific service is making the API call.
* **operationName**: This field logs the name of the GraphQL operation being performed. It is extracted from the context of the request and indicates which query or mutation is being executed.
* **operationSha256**: This field logs the SHA-256 hash of the original GraphQL operation, serving as a unique identifier for the operation's structure. It helps to identify operations in a secure and consistent way.
* **responseErrorMessage**: This field logs any error message returned in the response. It captures detailed information about what went wrong during the execution of the operation.
* **serviceNames**: This field logs the names of all services involved in processing the operation. It can be useful for tracking dependencies or microservices that are part of the request's execution.
* **operationHash**: This field logs a unique hash generated for the normalized operation. It serves as an identifier for the specific instance of the operation being executed, useful for tracing or debugging.
* **errorCodes**: This field logs any GraphQL error codes returned in the response. These codes help diagnose issues within the GraphQL schema or logic.
* **operationType**: This field logs the type of GraphQL operation being executed, such as a query, mutation, or subscription. It helps classify the request.
* **persistentOperationSha256**: This field logs the SHA-256 hash of the persisted GraphQL operation. Persisted operations are pre-saved on the server for efficiency, and this hash allows tracking them.
* **errorServiceNames**: This field logs the names of services that were involved when the operation resulted in an error. It helps identify which microservices or external APIs contributed to the issue.
* **operationParsingTime**: This field logs the amount of time (float seconds) spent parsing the GraphQL operation. It measures how long it took to interpret the operation’s syntax.
* **operationPlanningTime**: This field logs the time (float seconds) taken for planning the execution of the GraphQL operation, including resolving dependencies or building execution strategies.
* **operationNormalizationTime**: This field logs the time (float seconds) spent normalizing the GraphQL operation, which involves standardizing the operation to ensure consistency in processing.
* **operationValidationTime**: This field logs the time (float seconds) spent validating the GraphQL operation, ensuring that the operation is correct and adheres to the GraphQL schema and rules before execution.

#### Error-Specific Log Fields

In addition to the standard and custom fields typically present in a log, certain fields are included only in error scenarios to capture details about issues that occurred during request processing. These fields are useful for diagnosing errors in GraphQL operations and understanding the root causes.

Here are the fields that are specifically added in error cases:

* **graphql\_error\_service\_names**:\
  Lists the names of the services involved in the GraphQL request where an error occurred. This helps in pinpointing which microservice(s) or backend service(s) contributed to the error.
* **graphql\_error\_codes**:\
  Provides error codes associated with the GraphQL operation. These codes may correspond to specific validation, execution, or schema-related errors that occurred during processing.
* **response\_error\_message**:\
  Contains the actual error message returned in the response. This message can be used to better understand what went wrong and why the request failed.

#### Default value

When extracting values from a header, it can be beneficial to ensure a default value is set. This can be achieved by defining the value as follows:

```yaml
access_logs:
  enabled: true
  fields:
    - key: "service"
      default: "gateway" # Here
      value_from:
        request_header: "x-service"
```

#### Example Log Entry

```json5
12:39:49 PM INFO /graphql {
  "hostname": "dustins-MacBook-Pro.local",
  "pid": 15142,
  "log_type": "request",
  "method": "POST",
  "path": "/graphql",
  "query": "",
  "ip": "[REDACTED]",
  "user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.0.0 Safari/537.36",
  "config_version": "6c6b3f36-99b9-4632-84e1-ca24d95eee8d",
  "trace_id": "ed9acd8851c666d53f1d7244015cf71a",
  "latency": 0.019447292,
  "status": 200,
  "request_id": "dustins-MacBook-Pro.local/XKchE7RlTu-000001",
  "operationSha256": "57fcea4f7ba8da669882d80960c08607fa13873af754642af6769d8d25f09b99",
  "serviceNames": ["employees"],
  "operationHash": "14226210703439426856",
  "operationType": "query",
  "operationParsingTime": 0.000066541,
  "operationPlanningTime": 0.001880208,
  "operationNormalizationTime": 0.000468625,
  "operationValidationTime": 0.000281958
}
```

### Structured log output

In development, log lines are formatted for readability using pretty printing. In production mode, all logs are output as newline-separated JSON values.

### Conclusion

By configuring access logs, you can gather vital data on the traffic processed by your router. This helps in monitoring performance, debugging issues, and understanding request patterns. Whether logging to `stdout` or to a file, the logs provide flexibility with both default and custom fields to suit your specific needs.
