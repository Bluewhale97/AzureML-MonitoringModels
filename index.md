## Introduction

After a machine learning model has been deployed into production, it's important to understand how it is being used by capturing and viewing telemetry. Now let's know how to use Azure Application Insights to monitor a deployed Azure Machine Learning model..

## 1. Enabling application insights

To log telemetry in application insights from an Azure machine learning service, you must have an Application Insights resource associated with your Azure Machine Learning workspace, and you must configure your service to use it for telemetry logging.

When you create an Azure Machine Learning workspace, you can select an Azure Application Insights resource to associate with it. If you do not select an existing Application Insights resource, a new one is created in the same resource group as your workspace.

You can determine the Application Insights resource associated with your workspace by viewing the Overview page of the workspace blade in the Azure portal, or by using the get_details() method of a Workspace object as shown in the following code example:

```python
from azureml.core import Workspace

ws = Workspace.from_config()
ws.get_details()['applicationInsights']
```

Enable application insights for a service:
When deploying a new real-time service, you can enable Application Insights in the deployment configuration for the service, as shown in this example:
```python
dep_config = AciWebservice.deploy_configuration(cpu_cores = 1,
                                                memory_gb = 1,
                                                enable_app_insights=True)
```                                               
If you want to enable Application Insights for a service that is already deployed, you can modify the deployment configuration for Azure Kubernetes Service (AKS) based services in the Azure portal. Alternatively, you can update any web service by using the Azure Machine Learning SDK, like this:
```python
service = ws.webservices['my-svc']
service.update(enable_app_insights=True)
```

## 2. Capturing and Viewing telemetry

Application Insights automatically captures any information written to the standard output and error logs, and provides a query capability to view data in these logs.

To capture telemetry data for Application insights, you can write any values to the standard output log in the scoring script for your service by using a print statement, as shown in the following example:

```python
def init():
    global model
    model = joblib.load(Model.get_model_path('my_model'))
def run(raw_data):
    data = json.loads(raw_data)['data']
    predictions = model.predict(data)
    log_txt = 'Data:' + str(data) + ' - Predictions:' + str(predictions)
    print(log_txt)
    return predictions.tolist()
```
Azure Machine Learning creates a custom dimension in the Application Insights data model for the output you write.
To analyze captured log data, you can use the Log Analytics query interface for Application Insights in the Azure portal. This interface supports a SQL-like query syntax that you can use to extract fields from logged data, including custom dimensions created by your Azure Machine Learning service.

For example, the following query returns the timestamp and customDimensions.Content fields from log traces that have a message field value of STDOUT (indicating the data is in the standard output log) and a customDimensions.["Service Name"] field value of my-svc:

```SQL
traces
|where message == "STDOUT"
  and customDimensions.["Service Name"] = "my-svc"
| project  timestamp, customDimensions.Content
```
This query returns the logged data as a table:

![image](https://user-images.githubusercontent.com/71245576/116636898-71efd700-a930-11eb-81a5-a583bf794f4e.png)


## Reference:

Build AI solutions with Azure Machine Learning, retrieved from https://docs.microsoft.com/en-us/learn/paths/build-ai-solutions-with-azure-ml-service/




