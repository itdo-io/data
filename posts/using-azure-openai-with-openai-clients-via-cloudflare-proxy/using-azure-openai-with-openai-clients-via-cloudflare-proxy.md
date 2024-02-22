---
title: 'Leveraging Azure OpenAI with OpenAI Clients through a Cloudflare Proxy'
description: 'Learn how to integrate Azure OpenAI with OpenAI clients using a Cloudflare Worker proxy. This method allows access to Azure OpenAIs features, including GPT-3, GPT-4, and DALL-E-3, without openai clients'
slug: 'using-azure-openai-with-openai-clients-via-cloudflare-proxy'
category: 
    - ai
    - cloudflare
published: '2024-02-21'
---

## Table of Contents

## Introduction

Whole code is available at [GitHub](https://github.com/pvbhanuteja/cf-openai-azure-proxy)

Azure OpenAI Service offers a compelling alternative to OpenAI's offerings, providing easy application and card binding processes, along with a free credits for startups. However, many OpenAI clients do not support Azure OpenAI Service out of the box. This guide introduces a workaround using a free Cloudflare Worker as a proxy, enabling these clients to communicate with Azure OpenAI Service seamlessly.

## Why Use a Cloudflare workers?

Cloudflare workers will work as a proxy between the client and Azure OpenAI Service. This approach offers several advantages:

- **No Server Required:** The proxy script runs on Cloudflare Workers, eliminating the need for a server. Cloudflare offers 100,000 free requests per day.
- **No Domain Required:** If you don’t own a domain, you can still use this method. For details, refer to the original documentation.
- **Printer Mode:** Azure OpenAI Service sends responses in segments. This project processes and delivers these segments sequentially to the client, achieving a "printer mode" effect.

## Supported Models

All deployments of Azure OpenAI Service are supported, including GPT-3, GPT-4, and DALL-E-3. Create multiple deployments and map them to different endpoints to use them with the proxy incase if a single deployment is not enough.

## Deployment Steps

To proxy OpenAI requests to Azure OpenAI Service, follow these steps:

First setup the Azure OpenAI Service and create a deployment and get the resource name and deployment mapper.

{% img src="azure.png" alt="Diagram showing Azure OpenAI Service" %}

1. **Register and Log In to Cloudflare:** Create a new Cloudflare Worker.
2. **Script Setup:** Copy and paste the `cf-openai-azure-proxy.js`[code](https://raw.githubusercontent.com/pvbhanuteja/cf-openai-azure-proxy/main/cf-openai-azure-proxy.js) script into the Cloudflare Worker editor.
3. **Configuration:** Adjust the `resourceName` and deployment mapper values directly in the script or via environment variables.

    `Directly in the script:`

    ```js:sample
    // The name of your Azure OpenAI Resource.
    const resourceName="codegpt"

    const mapper:any = {
        'gpt-3.5-turbo': 'gpt3',
        'gpt-4': 'gpt4' 
    };
    ```

    `Via environment variables:`

    Go to the Cloudflare Worker console, navigate to Workers script > Settings > Add variable under Environment Variables.

    {% img src="cf_envs.png" alt="Diagram showing Cloudflare Worker Environment Variable" %}

4. **Deployment:** Save and deploy your Cloudflare Worker.
5. **Custom Domain (Optional):** Bind a custom domain to your Worker for easier access.
