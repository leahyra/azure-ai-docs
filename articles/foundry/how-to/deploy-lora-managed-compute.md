---
title: Deploy fine-tuned LoRA adapters to managed compute (preview)
titleSuffix: Microsoft Foundry
description: Deploy and test Foundry fine-tuned LoRA adapters on a shared managed compute target by using the portal or REST.
author: chillatom
ms.author: gulsimoosimi
ms.reviewer: mopeakande
ms.service: microsoft-foundry
ms.subservice: foundry-model-inference
ms.topic: how-to
ms.date: 10/06/2026
ms.custom: doc-kit-assisted
ai-usage: ai-assisted
#CustomerIntent: As a Microsoft Foundry developer, I want to deploy and test one or more fine-tuned LoRA adapters on managed compute so that the adapters can share a compatible target deployment.
---

# Deploy fine-tuned LoRA adapters to managed compute (preview)

[!INCLUDE [Feature preview](../includes/feature-preview.md)]

Deploy one or more low-rank adaptation (LoRA) adapters produced by Foundry fine-tuning to a compatible managed compute target deployment. You can deploy an adapter, test it, and attach more adapters to the same target deployment.

This article applies only to LoRA adapters produced as the output of fine-tuning in Foundry. For terminology, architecture, shared inference behavior, lifecycle, billing, and limits, see [Adapter deployment overview for managed compute](../concepts/adapter-deployment-overview.md).

## Prerequisites

- An Azure subscription and a [Foundry project](create-projects.md).
- A registered LoRA adapter produced by fine-tuning in Foundry and visible on the project's **Models** page.
- Permission to manage deployments in the Foundry account, such as the **Foundry Owner** or **Cognitive Services Contributor** role. You also need access to the registered adapter model.

  [!INCLUDE [Role rename](../includes/role-rename-note.md)]

- The **Foundry User** role on the account for inference with Microsoft Entra ID. See [Managed compute access control](../concepts/managed-compute-overview.md#access-control).
- A compatible base model and LoRA-enabled deployment template available in your region.
- Sufficient [managed compute quota](deploy-models-managed.md#request-more-quota) for the target deployment's accelerator family and region. You can select an existing target deployment or create one during adapter deployment.

Managed compute preview doesn't include built-in content filtering. Review its [limitations](../concepts/managed-compute-overview.md#limitations) and your application's safety requirements before deploying.

## Deploy adapters in the portal

Start from a registered LoRA adapter produced by fine-tuning in Foundry. You can find the adapter on **Build** > **Models** or start from the **Fine-tuning** tab.

One target deployment supports up to eight adapter deployments.

### 1. Select the adapter and start the deployment flow

In the [Foundry portal](https://ai.azure.com), use one of these entry points:

- Open **Build** > **Models**, select the LoRA adapter you want to deploy, and then select **Deploy**. This entry point opens the managed compute deployment flow directly.
- Open the **Fine-tuning** tab, select the adapter produced by your completed fine-tuning job, and then select **Deploy**. Select **Managed compute** from the available deployment options.

### 2. Select or create the target deployment

A registered adapter doesn't need its own compute, but it needs a compatible target deployment to serve requests. Select it in the **Target deployment** field.

- If a compatible LoRA-enabled target deployment already exists, select it in **Target deployment** and continue to [Deploy the adapter](#3-deploy-the-adapter).
- If no compatible deployment exists, create a target deployment within the adapter deployment flow.

The target deployment creation window has LoRA support enabled by default. To create the deployment:

1. Select a LoRA-enabled **Deployment template**.
1. Review the template details, including the inference runtime, model ID, and deployment template ID.
1. Select the **Accelerator type**, such as H100, that matches your selected template.
1. Complete the target deployment configuration and select **Deploy**.
1. Wait for the target deployment to finish provisioning. This operation can take several minutes. When provisioning completes, the **Target deployment** field is populated automatically.

For more information about target deployment configuration and capacity, see [Deploy open-source models with managed compute](deploy-models-managed.md).

### 3. Deploy the adapter

1. Enter an adapter deployment name, such as `qwen-support`. Use a name that's unique within the account and different from the target deployment's name.
1. Review the selected target deployment and select **Deploy**.
1. Wait for the adapter deployment to complete. The deployed adapter appears in the adapter deployment list.

Adapter deployment names can contain 1 to 64 letters, numbers, periods, underscores, or hyphens. Start the name with a letter or number.

Select the deployed adapter to open its details. From this page, you can:

- Open it in the playground.
- Add it to an agent.
- View the inference endpoint and authentication details.
- View its target deployment.
- Copy a code snippet to call the adapter.
- Use the quick setup for Visual Studio Code.

Open the adapter in the playground and send a prompt relevant to its fine-tuning task. Use the adapter deployment's code snippet to test it from your application. Calling only the target deployment doesn't test the adapter.

### 4. Deploy more adapters to the same target deployment

To share a target deployment between adapters:

1. Select another compatible adapter on **Build** > **Models** and select **Deploy**.
1. Give it a different deployment name, such as `qwen-finance`.
1. Select the same target deployment in **Target deployment**.
1. Select **Deploy** and wait for deployment to complete.
1. Open the target deployment's details to confirm that both adapter deployments are listed.

This configuration is a multi-LoRA deployment. Each adapter has its own deployment name, but both share the target deployment's compute and inference endpoint.

## Test an adapter with chat completions

After the adapter deployment reaches the `Succeeded` provisioning state, send a request to the OpenAI-compatible chat-completions route. Copy the account endpoint from the adapter deployment details in the portal. For Microsoft Entra ID authentication, acquire an access token for the `https://ai.azure.com/.default` scope.

Set `model` to the adapter deployment name, not the target deployment name. The service uses this value to select the adapter and resolves its target deployment internally.

```http
POST {account-endpoint}/openai/v1/chat/completions
Authorization: Bearer {access-token}
Content-Type: application/json

{
  "model": "qwen-support",
  "messages": [
    {
      "role": "user",
      "content": "How do I reset my account password?"
    }
  ]
}
```

The response follows the chat-completions response schema and returns the selected adapter deployment name in `model`. To test the second adapter in the multi-LoRA example, change `model` to `qwen-finance` and send a prompt for its fine-tuned task. The request doesn't require an `api-version` query parameter.

## Deploy and manage adapters with REST

Use REST to create and manage adapter deployments with API version `2026-09-15-preview`.

Create an adapter deployment by providing its registered source model ID and target deployment name. Use an access token for Azure Resource Manager in the authorization header; see [Manage Azure resources with REST](/azure/azure-resource-manager/management/manage-resources-rest).

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.CognitiveServices/accounts/{accountName}/adapterDeployments/{adapterDeploymentName}?api-version=2026-09-15-preview
Authorization: ******
Content-Type: application/json

{
  "properties": {
    "sourceModelId": "azureai://accounts/{accountName}/projects/{projectName}/models/{modelName}/versions/{version}",
    "targetDeploymentName": "{targetDeploymentName}"
  }
}
```

Deployment is asynchronous. Poll the URL returned in `Azure-AsyncOperation`, respecting `retry-after`, until the operation succeeds or fails. Confirm that `properties.provisioningState` is `Succeeded` before testing the adapter.

Check an adapter deployment's status:

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.CognitiveServices/accounts/{accountName}/adapterDeployments/{adapterDeploymentName}?api-version=2026-09-15-preview
Authorization: ******
```

List adapters attached to a target deployment:

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.CognitiveServices/accounts/{accountName}/adapterDeployments?api-version=2026-09-15-preview&$filter=properties%2FtargetDeploymentName%20eq%20%27{targetDeploymentName}%27
Authorization: ******
```

The response is paginated; follow `nextLink` until it's absent.

Delete an adapter deployment:

```http
DELETE https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.CognitiveServices/accounts/{accountName}/adapterDeployments/{adapterDeploymentName}?api-version=2026-09-15-preview
Authorization: ******
```

Deletion can also be asynchronous. For a `202` response, follow the operation URL until deletion completes. A `204` response requires no further polling.

## Manage and clean up deployments

Manage each adapter by its deployment name. To move an adapter to a different compatible target deployment, update `targetDeploymentName` while keeping the same `sourceModelId`. Wait for the update to succeed before relying on the new target deployment.

You can't change an existing adapter deployment's source model version. To deploy a new fine-tuned version, create a new adapter deployment with a different name, or delete and recreate the existing deployment.

Delete an adapter deployment when you no longer need to serve it. Use the REST delete request in [Deploy and manage adapters with REST](#deploy-and-manage-adapters-with-rest).

Deleting an adapter deployment doesn't delete its registered model version or target deployment. Other adapters still use the shared compute.

Remove all attached adapter deployments before deleting a target deployment you created for this exercise. If you added the optional `qwen-finance` adapter, delete it too. Don't delete a shared target deployment that other applications still need.

Before deleting a registered adapter model version, remove the adapter deployments that use it.

Scale shared compute through the [target deployment](../concepts/managed-compute-overview.md#scaling). A capacity change affects the compute shared by its adapters.

## Troubleshoot

Check the adapter registration, base model compatibility, and deployment status before retrying.

| Symptom | What to check |
| --- | --- |
| The adapter isn't listed on **Models** | Confirm that the adapter is registered in the selected project. Finishing training alone isn't enough. |
| An adapter can't be deployed | Confirm that the adapter is registered as the output of a Foundry fine-tuning job. This flow supports only adapters produced by fine-tuning in Foundry. |
| No deployment appears in **Target deployment** | Create a compatible LoRA-enabled target deployment within the adapter deployment flow. |
| No compatible template appears | Confirm that a LoRA-enabled template is available for the model and region. A template without LoRA support can't host the adapter. |
| Target deployment creation fails with a quota error | Check the accelerator family, region, and instance count. [Request managed compute quota](deploy-models-managed.md#request-more-quota) if needed. |
| The adapter and base model don't match | Compare the exact base model and version used for fine-tuning with the deployed base model. |
| The adapter exceeds a template limit | Check the adapter count, LoRA rank, and memory requirements against the selected template's supported limits. |
| A response looks like the base model | Confirm that adapter deployment completed and that you're testing the adapter, not just the target deployment. Use the adapter's portal code snippet. |
| The source model version can't be updated | Create a new adapter deployment, or delete and recreate the existing one. The source model version is fixed for an existing deployment. |

## Related content

Learn more about adapter deployments, target deployments, shared compute, and other fine-tuned model deployment options:

- [Adapter deployment overview for managed compute](../concepts/adapter-deployment-overview.md).
- [Deploy open-source models with managed compute](deploy-models-managed.md).
