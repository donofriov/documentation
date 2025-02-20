---
title: Install Serverless Monitoring for AWS Step Functions
further_reading:
    - link: '/serverless/configuration/'
      tag: 'Documentation'
      text: 'Configure Serverless Monitoring'
    - link: "/integrations/amazon_lambda/"
      tag: "Documentation"
      text: "AWS Lambda Integration"
---

### Requirements
* The full Step Function execution length must be less than 6 hours for full traces.

### How it works
AWS Step Functions is a fully managed service, and the Datadog Agent cannot be directly installed on Step Functions. However, Datadog can monitor Step Functions through Cloudwatch metrics and logs.

Datadog collects Step Functions metrics from Cloudwatch through the [AWS Step Functions integration][9]. Datadog collects Step Functions logs from Cloudwatch through one of the following:

- [Datadog Forwarder][6]. For instructions, see the [Setup](#setup) section on this page.
- Amazon Data Firehose. For instructions, see [Send AWS service logs to the Datadog Amazon Data Firehose destination][7].

Datadog uses these ingested logs to generate [enhanced metrics][8] and traces for your Step Function executions.

{{< img src="serverless/step_functions/telemetry_ingestion.png" alt="A diagram explaining how Step Functions telemetry is ingested and used in Datadog" style="width:100%;" >}}

## Setup

Ensure that the [AWS Step Functions integration][9] is installed.

Then, to send your Step Functions logs to Datadog:

{{< tabs >}}
{{% tab "Serverless Framework" %}}

For developers using [Serverless Framework][4] to deploy serverless applications, use the Datadog Serverless Framework Plugin.

1. If you have not already, install the [Datadog Serverless Framework Plugin][1] v5.40.0+:

    ```shell
    serverless plugin install --name serverless-plugin-datadog
    ```

2. Ensure you have deployed the [Datadog Lambda Forwarder][2], a Lambda function that ships logs from AWS to Datadog, and that you are using v3.121.0+. You may need to [update your Forwarder][5].

   Take note of your Forwarder's ARN.

3. Add the following to your `serverless.yml`:

    ```yaml
    custom:
      datadog:
        site: <DATADOG_SITE>
        apiKeySecretArn: <DATADOG_API_KEY_SECRET_ARN>
        forwarderArn: <FORWARDER_ARN>
        enableStepFunctionsTracing: true
    ```
    - Replace `<DATADOG_SITE>` with {{< region-param key="dd_site" code="true" >}} (ensure the correct SITE is selected on the right).
    - Replace `<DATADOG_API_KEY_SECRET_ARN>` with the ARN of the AWS secret where your [Datadog API key][3] is securely stored. The key needs to be stored as a plaintext string (not a JSON blob). The `secretsmanager:GetSecretValue` permission is required. For quick testing, you can instead use `apiKey` and set the Datadog API key in plaintext.
    - Replace `<FORWARDER_ARN>` with the ARN of your Datadog Lambda Forwarder, as noted previously.

    For additional settings, see [Datadog Serverless Framework Plugin - Configuration parameters][7].

[1]: https://docs.datadoghq.com/serverless/libraries_integrations/plugin/
[2]: /logs/guide/forwarder/
[3]: https://app.datadoghq.com/organization-settings/api-keys
[4]: https://www.serverless.com/
[5]: /logs/guide/forwarder/?tab=cloudformation#upgrade-to-a-new-version
[6]: logs/guide/forwarder/?tab=cloudformation#installation
[7]: https://github.com/datadog/serverless-plugin-datadog?tab=readme-ov-file#configuration-parameters
{{% /tab %}}
{{% tab "Datadog CLI" %}}
1. If you have not already, install the [Datadog CLI][1] v2.18.0+.

   ```shell
   npm install -g @datadog/datadog-ci
   ```
2. Ensure you have deployed the [Datadog Lambda Forwarder][2], a Lambda function that ships logs from AWS to Datadog, and that you are using v3.121.0+. You may need to [update your Forwarder][3].

   Take note of your Forwarder's ARN.
3. Instrument your Step Function.

   ```shell
   datadog-ci stepfunctions instrument \
    --step-function <STEP_FUNCTION_ARN> \
    --forwarder <FORWARDER_ARN> \
    --env <ENVIRONMENT> 
   ```
   - Replace `<STEP_FUNCTION_ARN>` with the ARN of your Step Function. Repeat the `--step-function` flag for each Step Function you wish to instrument.
   - Replace `<FORWARDER_ARN>` with the ARN of your Datadog Lambda Forwarder, as noted previously.
   - Replace `<ENVIRONMENT>` with the environment tag you would like to apply to your Step Functions.

   For more information about the `datadog-ci stepfunctions` command, see the [Datadog CLI documentation][5].


[1]: /serverless/libraries_integrations/cli/
[2]: /logs/guide/forwarder/
[3]: /logs/guide/forwarder/?tab=cloudformation#upgrade-to-a-new-version
[4]: logs/guide/forwarder/?tab=cloudformation#installation
[5]: https://github.com/DataDog/datadog-ci/blob/master/src/commands/stepfunctions/README.md
{{% /tab %}}
{{% tab "Custom" %}}

1. Enable all logging for your Step Function. In your AWS console, open your state machine. Click *Edit* and find the Logging section. There, set *Log level* to `ALL` and enable the *Include execution data* checkbox.
   {{< img src="serverless/step_functions/aws_log.png" alt="AWS UI, Logging section, showing log level set to ALL." style="width:100%;" >}}

2. Ensure you have deployed the [Datadog Lambda Forwarder][1], a Lambda function that ships logs from AWS to Datadog, and that you are using v3.121.0+. You may need to [update your Forwarder][2]. When deploying the Forwarder on v3.121.0+, you can also set the `DdStepFunctionsTraceEnabled` parameter in CloudFormation to enable tracing for all your Step Functions at the Forwarder-level.

   Take note of your Forwarder's ARN.

3. Subscribe CloudWatch logs to the Datadog Lambda Forwarder. To do this, you have two options:
   - **Datadog-AWS integration** (recommended)
     1. Ensure that you have set up the [Datadog-AWS integration][4].
     2. In Datadog, open the [AWS integration tile][5], and view the *Configuration* tab.
     3. On the left, select the AWS account where your Step Function is running. Open the *Log Collection* tab.
     4. In the *Log Autosubscription* section, under *Autosubscribe Forwarder Lambda Functions*, enter the ARN of your Datadog Lambda Forwarder, as noted previously. Click *Add*.
     5. Toggle on *Step Functions CloudWatch Logs*. Changes take 15 minutes to take effect.

     **Note**: Log Autosubscription requires your Lambda Forwarder and Step Function to be in the same region.

   - **Manual**
     1. Ensure that your log group name has the prefix `/aws/vendedlogs/states`. 
     2. Open your AWS console and go to your Datadog Lambda Forwarder. In the *Function overview* section, click on *Add trigger*.
     3. Under *Add trigger*, in the *Trigger configuration* section, use the *Select a source* dropdown to select `CloudWatch Logs`.
     4. Under *Log group*, select the log group for your state machine. For example, `/aws/vendedlogs/states/my-state-machine`.
     5. Enter a filter name. You can choose to name it "empty filter" and leave the *Filter pattern* box blank.

<div class="alert alert-warning"> If you are using a different instrumentation method, such as Serverless Framework or datadog-ci, enabling autosubscription may create duplicated logs. To avoid this behavior, choose only one configuration method.</div>

4. Set up tags. Open your AWS console and go to your Step Functions state machine. Open the *Tags* section and add `env:<ENV_NAME>`, `service:<SERVICE_NAME>`, and `version:<VERSION>` tags. The `env` tag is required to see traces in Datadog, and it defaults to `dev`. The `service` tag defaults to the state machine's name. The `version` tag defaults to `1.0`.


[1]: /logs/guide/forwarder/
[2]: /logs/guide/forwarder/?tab=cloudformation#upgrade-to-a-new-version
[4]: /getting_started/integrations/aws/
[5]: https://app.datadoghq.com/integrations/aws
{{% /tab %}}
{{< /tabs >}}

## Enable tracing

<div class="alert alert-info">To see Step Functions spans and Lambda spans connected in the same trace, see <a href="/serverless/step_functions/merge-step-functions-lambda">Merge Step Functions with your AWS Lambda traces</a>.</div>

Datadog generates traces from collected Cloudwatch logs. To enable this, add a `DD_TRACE_ENABLED` tag to each of your Step Functions and set the value to `true`. Alternatively, to enable tracing for **all** your Step Functions, add a `DD_STEP_FUNCTIONS_TRACE_ENABLED` environment variable to the Datadog Forwarder and set the value to `true`.

Enhanced metrics are automatically enabled if you enable tracing.

<div class="alert alert-info">If you enable enhanced metrics without enabling traces, you are only billed for Serverless Workload Monitoring. If you enable tracing (which automatically includes enhanced metrics), you are billed for both Serverless Workload Monitoring and Serverless APM. See <a href="https://www.datadoghq.com/pricing/?product=serverless-monitoring#products">Pricing</a>.</div>

## Merge Step Functions with your AWS Lambda traces

See [Merge Step Functions traces with Lambda traces][11]. Ensure that you have also [set up Serverless Monitoring for AWS Lambda][10].

## Sample traces

To manage the APM traced invocation sampling rate for serverless functions, set the `DD_TRACE_SAMPLE_RATE` environment variable on the function to a value between 0.000 (no tracing of Step Function invocations) and 1.000 (trace all Step Function invocations). 

The dropped traces are not ingested into Datadog. 

## Enable enhanced metrics (without tracing)

Datadog generates [enhanced metrics][8] from collected Cloudwatch logs. Enhanced metrics are automatically enabled if you enable traces.

To enable enhanced metrics without enabling tracing, add a `DD_ENHANCED_METRICS` tag to each of your Step Functions and set the value to `true`.

## See your Step Function metrics, logs, and traces in Datadog

After you have invoked your state machine, go to the [**Serverless app**][2] in Datadog. Search for `service:<YOUR_STATE_MACHINE_NAME>` to see the relevant metrics, logs, and traces associated with that state machine. If you set the `service` tag on your state machine to a custom value, search for `service:<CUSTOM_VALUE>`.

{{< img src="serverless/step_functions/overview1.png" alt="An AWS Step Function side panel view." style="width:100%;" >}}

If you cannot see your traces, see [Troubleshooting][5].

[2]: https://app.datadoghq.com/functions?search=&cloud=aws&entity_view=step_functions
[3]: /serverless/installation/#installation-instructions
[5]: /serverless/step_functions/troubleshooting
[6]: /logs/guide/forwarder
[7]: /logs/guide/send-aws-services-logs-with-the-datadog-kinesis-firehose-destination
[8]: /serverless/step_functions/enhanced-metrics
[9]: /integrations/amazon_step_functions
[10]: /serverless/aws_lambda/installation
[11]: /serverless/step_functions/merge-step-functions-lambda
