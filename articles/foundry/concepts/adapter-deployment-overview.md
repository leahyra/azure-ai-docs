---
title: Adapter deployments for managed compute
titleSuffix: Microsoft Foundry
description: Learn how LoRA adapter deployments share managed compute, capacity, and inference endpoints through compatible target deployments in Microsoft Foundry.
author: chillatom
ms.author: gulsimoosimi
ms.reviewer: mopeakande
ms.service: microsoft-foundry
ms.subservice: foundry-model-inference
ms.topic: concept-article
ms.date: 10/06/2026
ms.custom: doc-kit-assisted
ai-usage: ai-assisted
#CustomerIntent: As a Microsoft Foundry developer, I want to understand adapter deployments and multi-LoRA shared serving so that I can plan how to serve fine-tuned adapters on managed compute.
---

# Adapter deployment overview for managed compute (preview)

[!INCLUDE [Feature preview](../includes/feature-preview.md)]

An adapter deployment serves a low-rank adaptation (LoRA) adapter on a compatible managed compute target deployment in Microsoft Foundry. Adapter deployments don't require additional compute. They share the compute provided by the base model deployment, referred to as the target deployment. Even though adapter deployments don't require additional compute, they come with their own deployment name, which enables you to route inference requests through the inference endpoint of the Foundry account. 

Multiple compatible adapters can share one target deployment. This configuration is called multi-LoRA serving. 

This article applies only to LoRA adapters produced as the output of fine-tuning in Foundry. You can't yet import your own adapters.

## How LoRA adapters specialize a base model

Low-rank adaptation is a fine-tuning technique that trains a small set of additional parameters instead of updating every parameter in the base model. The resulting LoRA adapter is a lightweight set of learned changes that specializes the model for a task or domain.

A LoRA adapter can't run by itself. At inference time, the serving runtime combines the adapter with the compatible base model. The exact base model and version matter because the adapter's learned changes apply to that model's architecture and weights.

## Resources in an adapter deployment

Adapter serving uses related model and deployment resources. Each resource has a separate responsibility.

| Term | Meaning |
| --- | --- |
| **Base model** | The underlying model and version used to create the adapter. A target deployment must run a compatible base model and version. |
| **Registered LoRA adapter** | The model asset created from a Foundry fine-tuning output. Registration makes the adapter available on **Build > Models**, but doesn't provide compute or make the adapter callable. |
| **Deployment template** | A named, versioned asset that defines how a model runs, including the serving runtime and accelerator configuration. For adapter serving, select a template that supports LoRA. |
| **Target deployment** | A managed compute deployment that runs the compatible base model by using a LoRA-enabled deployment template. It owns the accelerators, capacity, replicas, and inference endpoint that its adapters share. |
| **Adapter deployment** | A deployed connection between one registered LoRA adapter version and one target deployment. It has its own deployment name, but it doesn't provision separate compute, capacity, replicas, or an endpoint. |
| **Multi-LoRA deployment** | Two or more compatible adapter deployments that use the same target deployment. Each adapter keeps its own deployment name and specialized behavior while sharing the target deployment's infrastructure. |

For example, a target deployment named `qwen-prod` can host adapter deployments named `qwen-support` and `qwen-finance`. Both adapters use the base model, compute, and endpoint supplied by `qwen-prod`. Their separate deployment names identify which fine-tuned behavior an application selects.

## How multi-LoRA sharing works

The target deployment is the shared serving foundation. Its deployment template determines whether the runtime supports LoRA and defines the accelerator type, model instances, and other runtime configuration.

Attaching an adapter doesn't create another model instance or increase the target deployment's capacity. Every adapter on the target draws from the same serving capacity. Scaling the target deployment changes the capacity available to the base model and all attached adapters.

## How inference selects an adapter

The adapter must finish provisioning before it can serve requests. A successful deployment reports a `Succeeded` provisioning state.

Suppose `qwen-prod` is the base model's target deployment and `qwen-support` and `qwen-finance` are its adapter deployments. Send requests to the Foundry account's `/openai/v1/chat/completions` endpoint. Set `model` to the deployment name you want to invoke:

| `model` value | Selected behavior |
| --- | --- |
| `qwen-support` | Base model with the support adapter. |
| `qwen-finance` | Base model with the finance adapter. |

The following request selects the support adapter. Copy the account endpoint from the adapter deployment details in the portal. For Microsoft Entra ID authentication, acquire an access token for the `https://ai.azure.com/.default` scope.

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

To invoke the finance adapter, change `model` to `qwen-finance` and use a prompt for its task. The service resolves the adapter's target deployment internally.

For authentication and testing instructions, see [Test an adapter with chat completions](../how-to/deploy-lora-managed-compute.md#test-an-adapter-with-chat-completions).

## Compatibility and capacity requirements

Foundry validates the adapter and target deployment before serving the adapter. The configuration must meet these requirements:

- The adapter is a registered output of a Foundry fine-tuning job.
- The target deployment runs the compatible base model and version.
- The target uses a deployment template that supports LoRA.
- The adapter fits within the template's supported adapter count, LoRA rank, and available memory.
- The target has sufficient managed compute quota and capacity for its accelerator family and region.

## Adapter lifecycle and dependencies

An adapter deployment and its registered LoRA adapter are separate resources. The following rules describe how deployment changes affect the adapter and target deployment.

An existing adapter deployment references a fixed version of the registered LoRA adapter, not the base model. To serve a different adapter version, create a new adapter deployment, or delete and recreate the existing deployment.

You can move the same adapter source to another compatible target deployment by updating the target deployment name. Wait for the update to succeed before routing requests to the new target.

Deleting an adapter deployment doesn't delete its registered model version or target deployment. It also doesn't affect other adapters that share the target.

Remove dependent adapter deployments before deleting a registered adapter model version or target deployment. This ordering prevents an adapter deployment from referring to a source or target that no longer exists.

## Billing, quota, and limits

The target deployment is billed hourly per accelerator. Managed compute quota is allocated per accelerator family and region, separately from Azure virtual machine quota. For more information, see [Managed compute billing and quota](managed-compute-overview.md#billing-quota-and-deployment-scopes).

Adapter deployments have no separate pricing. Billing applies to the target deployment's compute.

For the adapter count limit, see [Deploy adapters in the portal](../how-to/deploy-lora-managed-compute.md#deploy-adapters-in-the-portal).

For current managed compute rates, see the [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator/).

## Next step

Deploy and test one or more registered LoRA adapters on a compatible target deployment.

> [!div class="nextstepaction"]
> [Deploy fine-tuned LoRA adapters to managed compute](../how-to/deploy-lora-managed-compute.md)
