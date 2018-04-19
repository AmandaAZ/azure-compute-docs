---
title: Monitor containers in Azure Container Instances
description: Details on how to monitor the consumption of compute resources like CPU and memory by your containers in Azure Container Instances.
services: container-instances
author: nepeters
manager: jeconnoc

ms.service: container-instances
ms.topic: overview
ms.date: 04/25/2018
ms.author: neilpeterson
---
# Monitor container resources in Azure Container Instances

Azure Monitor provides insight into the compute resources used by your containers instances. Use Azure Monitor to track the CPU and memory utilization of container groups and their containers. Create alerts to be notified when certain resource metrics are outside thresholds you specify. This resource usage data helps you determine the best CPU and memory settings for your container groups. Alerts provide you with the opportunity to make configuration adjustments before applications running in your containers are impacted by low-resource situations.

## Metrics

Azure Monitor currently provides metrics for two resource types in Azure Container instances: CPU and memory. Metrics for each are available both at the container group and individual container levels.

### CPU

CPU metrics are expressed in **millicores**. One millicore is 1/1000th of a CPU core, so 500 millicores (or 500m) represents 50% utilization of a CPU core.

### Memory

Memory metrics are expressed in **bytes**.

## Alerts

Create alerts based on specific metrics to be notified when they drift outside thresholds you set. For example, create an alert to be notified when average CPU usage exceeds a certain threshold, or available memory drops below a certain level. Configure alerts in the [Azure portal](../monitoring-and-diagnostics/insights-alerts-portal.md), with the [Azure CLI](../monitoring-and-diagnostics/insights-alerts-command-line-interface.md), or in [Azure PowerShell](../monitoring-and-diagnostics/insights-alerts-powershell.md).

## Create an alert

Use the following procedure to create an alert.

Go to [Azure Monitoring](https://portal.azure.com/?feature.customportal=false&feature.canmodifystamps=true&feature.testingGenv2Alerts=true#blade/Microsoft_Azure_Monitoring/AzureMonitoringBrowseBlade/alertsV2) in the Azure portal and select **New Alert Rule**.

[ IMAGE ]

### Define alert condition

* Select **Select target**
  * **Select a resource** > **Filter by resource type** > **Container instances**
  * Select one of the displayed **container groups**
  * Select **Done**
* Select **Add criteria**
  * From the available signals, select one of the available **Metric** signal types:
    * CPU Usage
    * Memory Usage
  * Select a container in the **DIMENSION VALUES** drop-down
    * If you choose not to select a specific container, the alert will fire based on the aggregated metrics for the selected resource type (CPU or memory) for all containers in the container group
  * Under **Alert logic**, configure the **Condition** and its **Threshold** under which the alert should fire
  * Also configure the time **Period** that metrics should be evaluated
  * Select **Done**

### Define alert details

* Configure the alert's details: **name**, **description**, **severity** level

* Define action group
  * Select **New action group**
    * Configure the action group properties, which defines where the alerts are sent.
    * Configure an **Email** action and enter your email address to which to send alerts, select **OK**
    * Select **OK** under **Add action group**
  * Select **Select action group**, select the action group you created, and click **Add**

### Finish alert creation

Select **Create alert rule** to create the alert

You've created an alert that will send email to the address you specified when CPU usage in the container averages over 500 millicores in any one minute period.

## Next steps

Like all Azure services, Azure Container Instances includes certain default limits and quotas for resources and features. Find details on these limits and how to request quota increases in [Quotas and region availability for Azure Container Instances](container-instances-quotas.md).

<!-- IMAGES -->
<!-- LINKS - External -->
<!-- LINKS - Internal -->