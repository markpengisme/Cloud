# Google Cloud Solutions I: Scaling Your Infrastructure

[toc]

> The stable Helm chart repositories deprecated on 2020/11/13, use <https://artifacthub.io> to search new chart

## Autoscaling an Instance Group with Custom Cloud Monitoring Metrics

- In this lab you will create a [Compute Engine](https://cloud.google.com/compute/docs/) managed instance group that autoscales based on the value of a custom [Cloud Monitoring](https://cloud.google.com/monitoring/docs/) metric.

- The application includes:

  1. **Compute Engine instance template**: used to create `(5)` in `(4)`
  2. **Cloud Storage**: Store `(3)` and other script files
  3. **Compute Engine startup script**: It is installs and starts code automatically when an instance starts, and write values to `(6)`
  4. **Compute Engine instance group**: collection of `(5)`, autoscales based on `(6)`
  5. **Compute Engine instances**: Using `(3)` which is stored in `(2)` to startup.
  6. **Custom Cloud Monitoring metric**: A custom monitoring metric used to autoscale.

- ![App arch](https://cdn.qwiklabs.com/peFD70aHkYcyJdJHw7VfhmNAaG1jgPYJlIbcqMjcB9A%3D)

- Creatring the bucket(2)

  - **Navigation menu** > **Cloud Storage** > **Create bucket**

  - Name: PROJECT_ID

  - Create

  - Copy Compute Engine startup script(3) to bucket

    ```SH
    gsutil cp -r gs://spls/gsp087/* gs://${YOUR_BUCKET}
    ```

  - Refresh Bucket details page

    - `Startup.sh`: installs the necessary
    - `writeToCustomMetric.js`: Creates a custom monitoring metric whose value triggers scaling
    - `writeToCustomMetric.sh`: Continuously runs the `writeToCustomMetric.js`
    - `Config.json`:  Specifies the values for the custom monitoring metric

- Creating an instance template(1)

  - Create a template for the instances that are created in the instance group

  - **Navigation menu** > **Compute Engine** > **Instance templates** >  **Create Instance Template**

  - Name: autoscaling-instance01

  - Mangement > Metadata

    | Key                | Value                           |
    | :----------------- | :------------------------------ |
    | startup-script-url | `gs://[YOUR_BUCKET]/startup.sh` |
    | gcs-bucket         | `gs://[YOUR_BUCKET]`            |

  - Create

- Creating the instance group(4)

  - **Navigation menu** > **Compute Engine** > **Instance groups** > **Create instance group**
  - Name: autoscaling-instance-group-1
  - Instance template: autoscaling-instance01
  - Autoscaling mode: Don't autoscaling
  - Create

- Verifying that the instance group has been created

  - Wait to see the **green check** mark next to the new instance group you just created.

- Verifying that the Node.js script is running

  - **autoscaling-instance-group-1** > **instance name** > **Cloud Logging**
  - **Query preview** has `resource.type` & `resource.labels.instance_id`, now add `"nodeapp"` as line 3
  - Click **Run Query**
  - Check `Finished writing time series data` appear in the logs

- Configure autoscaling for the instance group based on the value of the custom metric.

  - **Compute Engine** > **Instance groups** > **autoscaling-instance-group-1** > **Configure**
  - Set **Autoscaling mode** to **Autoscale**
  - **Add new metric**
    - Metric Type: Stackdriver Monitoring metric
    - Metric export scope: Time series per instance
    - Metric identifier: custom.googleapis.com/appdemo_queue_depth_01
    - Utilization target: 150
    - Utilization target type: Gauge
      - The autoscaler should compute the average value of the data collected over the last few minutes and compare it to the target value.
    - Minimum number of instances: 1
    - Maximum number of instances: 3
    - Save

- Watching the instance group perform autoscaling

  - \> target, add comput engine instance; \< target, remove comput engine instance
  - **Compute Engine** > **Instance groups** > **builtin-igm** > **Monitoring** > Enable **Auto Refresh**

  > Since this group had a head start, you can see the autoscaling details about the instance group in the autoscaling graph. The autoscaler will take about five minutes to correctly recognize the custom metric and it can take up to ten minutes for the script to generate sufficient data to trigger the autoscaling behavior.

- [Autoscaling example](https://www.qwiklabs.com/focuses/611?parent=catalog#step12)
