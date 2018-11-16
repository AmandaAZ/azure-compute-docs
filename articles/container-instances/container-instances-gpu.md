---
title: Deploy GPU-enabled container instances 
description: Learn how to deploy container instances to run on GPUs.
services: container-instances
author: dlepow
manager: jeconnoc

ms.service: container-instances
ms.topic: article
ms.date: 09/24/2018
ms.author: danlep
---

# Deploy container instances that use GPU resources

To run certain compute-intensive workloads on Azure Container Instances, deploy your container groups with *GPU resources*. The container instances can access one or more NVIDIA Tesla GPUs while running the container workloads.

As shown in this article, you can add GPU resources when you deploy a container group using the Azure CLI, a YAML file, or a Resource Manager template.

> [!IMPORTANT]
> This feature is currently in preview, and some [limitations apply](#preview-limitations). Previews are made available to you on the condition that you agree to the [supplemental terms of use][terms-of-use]. Some aspects of this feature may change prior to general availability (GA).


## Preview limitations

In preview, the following limitations apply when using GPU resources in container groups. 

**Supported regions**:

* East US (eastus)
* West US 2 (westus2)
* South Central US (southcentralus)
* West Europe (westeurope)

Support will be added for additional regions over time.

**Supported OS types**: Linux only

**Additional limitations**: GPU resources can't be used when deploying a container group into a [virtual network](container-instances-vnet.md).

## About GPU resources

To use GPUs in a container instance, specify a *GPU resource* with the following information:

* **Count** - The number of GPUs. You can add 1, 2, or 4 GPUs to a container instance.
* **SKU** - The NVIDIA Tesla GPU SKU, which is one of the following values: K80, P100, or V100. Each SKU maps to the Tesla GPU in one the following Azure GPU-enabled VM families, designed for various GPU workloads:

  | SKU | VM family |
  | --- | --- |
  | K80 | [NC](../virtual-machines/linux/sizes-gpu.md#nc-series) |
  | P100 | [NCv2](../virtual-machines/linux/sizes-gpu.md#ncv2-series) |
  | V100 | [NCv3](../virtual-machines/linux/sizes-gpu.md#ncv3-series) |

### CPU and Memory resources

When deploying GPU resources, set CPU and Memory resources appropriate for the workload, up to the maximum values supported in the underlying VM size: 

| SKU | Count | CPU |  Memory (GB) |
| --- | --- | --- | --- |
| K80 | 1 | 6 | 56 |
| K80 | 2 | 12 | 112 |
| K80 | 4 | 24 | 224 |
| P100 | 1 | 6 | 112 |
| P100 | 2 | 12 | 224 |
| P100 | 4 | 24 | 448 |
| V100 | 1 | 6 | 112 |
| V100 | 2 | 12 | 224 |
| V100 | 4 | 24 | 448 |

### Things to know

* Creation of a container group containing GPU resources may take up to **10 minutes**. This is due to the additional time to provision and configure a GPU VM in Azure. 
* As with container groups without GPU resources, Azure bills for the *duration* of the container group (after the group is created). The duration is calculated from the time to pull your first container's image until the container group terminates.
* Pricing for container group duration is greater for container groups with GPU resources than for container groups without.
* GPU resources are pre-provisioned with CUDA drivers, so you can run container images developed for CUDA workloads. 


## Azure CLI example

[waiting for CLI support]

## YAML example

Copy the following YAML into a new file named *vnet-deploy-aci.yaml*, then save the file. This YAML creates a container group named *gpucontainer* with a K80 GPU. The image runs a CUDA vector addition application.

```YAML
additional_properties: {}
apiVersion: '2018-10-01'
location: southcentralus
name: gpucontainer
properties:
  containers:
  - name: gpucontainer
    properties:
      environmentVariables: []
      image: k8s-gcrio.azureedge.net/cuda-vector-add:v0.1
      ports:
      - port: 80
      resources:
        requests:
          cpu: 2.0
          memoryInGB: 3.0
          gpu:
            count: 1
            sku: K80
  ipAddress:
    ports:
    - port: 80
      protocol: TCP
    type: Public
  osType: Linux
  restartPolicy: OnFailure
```

Deploy the container group with the [az container create][az-container-create] command, specifying the YAML file name for the `--file` parameter. You need to supply the name of a resource group that was created in a region that supports GPU resources.  

```azurecli
az container create --resource-group myResourceGroup --file gpu-deploy-aci.yaml
```

The deployment takes some time. Once the deployment has completed, the container starts and runs a CUDA vector addition operation. Run the [az container logs][az-container-logs] command to view the log output:

```Console
[Vector addition of 50000 elements]
Copy input data from the host memory to the CUDA device
CUDA kernel launch with 196 blocks of 256 threads
Copy output data from the CUDA device to the host memory
Test PASSED
Done
```

## Resource Manager template example


Start by creating a file named `gpudeploy.json`, then copy the following JSON into it. This example deploys a container instance that runs a [TensorFlow](https://www.tensorflow.org/versions/r1.1/get_started/mnist/beginners) job against the [MNIST dataset](http://yann.lecun.com/exdb/mnist/).

```JSON
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "containerGroupName": {
        "type": "string",
        "defaultValue": "gpucontainer",
        "metadata": {
          "description": "Container Group name."
        }
      }
    },
    "variables": {
      "containername": "gpucontainer",
      "containerimage": "microsoft/samples-tf-mnist-demo:gpu"
    },
    "resources": [
      {
        "name": "[parameters('containerGroupName')]",
        "type": "Microsoft.ContainerInstance/containerGroups",
        "apiVersion": "2018-10-01",
        "location": "[resourceGroup().location]",
        "properties": {
            "containers": [
            {
              "name": "[variables('containername')]",
              "properties": {
                "image": "[variables('containerimage')]",
                "resources": {
                  "requests": {
                    "cpu": 4,
                    "memoryInGb": 6.0,
                    "gpu": {
                        "count": 1,
                        "sku": "k80"
                  }
                }
              }
            }
          }
        ],
        "osType": "Linux"
        }
      }
    ]
}
```

Deploy the template with the [az group deployment create][az-group-deployment-create] command. You need to supply the name of a resource group that was created in a region that supports GPU resources.

```azurecli-interactive
az group deployment create --resource-group myResourceGroup --template-file gpudeploy.json
```

The deployment takes some time. Once the deployment has completed, the container starts and runs the TensorFlow job. Run the [az container logs][az-container-logs] command to view the log output:

```Console
2018-10-25 18:31:10.155010: I tensorflow/core/platform/cpu_feature_guard.cc:137] Your CPU supports instructions that this TensorFlow binary was not compiled to use: SSE4.1 SSE4.2 AVX AVX2 FMA
2018-10-25 18:31:10.305937: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1030] Found device 0 with properties:
name: Tesla K80 major: 3 minor: 7 memoryClockRate(GHz): 0.8235
pciBusID: ccb6:00:00.0
totalMemory: 11.92GiB freeMemory: 11.85GiB
2018-10-25 18:31:10.305981: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1120] Creating TensorFlow device (/device:GPU:0) -> (device: 0, name: Tesla K80, pci bus id: ccb6:00:00.0, compute capability: 3.7)
2018-10-25 18:31:14.941723: I tensorflow/stream_executor/dso_loader.cc:139] successfully opened CUDA library libcupti.so.8.0 locally
Successfully downloaded train-images-idx3-ubyte.gz 9912422 bytes.
Extracting /tmp/tensorflow/input_data/train-images-idx3-ubyte.gz
Successfully downloaded train-labels-idx1-ubyte.gz 28881 bytes.
Extracting /tmp/tensorflow/input_data/train-labels-idx1-ubyte.gz
Successfully downloaded t10k-images-idx3-ubyte.gz 1648877 bytes.
Extracting /tmp/tensorflow/input_data/t10k-images-idx3-ubyte.gz
Successfully downloaded t10k-labels-idx1-ubyte.gz 4542 bytes.
Extracting /tmp/tensorflow/input_data/t10k-labels-idx1-ubyte.gz
Accuracy at step 0: 0.097
Accuracy at step 10: 0.6993
Accuracy at step 20: 0.8208
Accuracy at step 30: 0.8594
...
Accuracy at step 990: 0.969
Adding run metadata for 999
```

## Next steps

* Learn more about [GPU optimized VM sizes](../virtual-machines/linux/sizes-gpu.md) in Azure.


<!-- IMAGES -->
[aci-vnet-01]: ./media/container-instances-vnet/aci-vnet-01.png

<!-- LINKS - External -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-show]: /cli/azure/container#az-container-show
[az-container-logs]: /cli/azure/container#az-container-logs
[az-group-deployment-create]: /cli/azure/group/deployment#az-group-deployment-create