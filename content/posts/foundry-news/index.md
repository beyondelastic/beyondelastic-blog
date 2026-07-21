+++
date = '2025-07-03T11:42:12+02:00'
draft = false
title = 'Azure AI Foundry News & Changes'
description = 'Highlights from Microsoft Build 2025 for Azure AI Foundry, including Agent Service GA, Connected Agents, Model Router, Tracing and new VS Code capabilities.'
categories = ['AI']
tags = ['Agents', 'Foundry', 'Azure', 'VS Code', 'MS Build']
+++

## Intro

In this short blog post, I want to share with you some of the news and changes from [Microsoft Build 2025](https://news.microsoft.com/build-2025/) around Azure AI Foundry. For example, the Azure AI Foundry Agent Service aka Azure AI Agent Service is now GA. The blog post for the GA announcement can be found [here](https://techcommunity.microsoft.com/blog/azure-ai-services-blog/announcing-general-availability-of-azure-ai-foundry-agent-service/4414352). Additionally, there were plenty of new public preview features for Azure AI Foundry announced, such as **Connected Agents**, **Model Router**, **Tracing** and many more. A summary of the Azure AI Foundry announcements can be found [here](https://azure.microsoft.com/en-us/blog/azure-ai-foundry-your-ai-app-and-agent-factory/). I am not going to cover every feature in this blog post, instead we are going to focus on the introduced changes and tease some of the VS Code related capabilities. 

### New resources & changes

As mentioned above, besides all the new announcements, there were some changes around the Azure AI Foundry resources and APIs. The former construct was built with **Hub**-based projects hosted in an Azure AI Foundry Hub.

![Hub](/hub.png)

This is now being replaced by **Foundry** projects built on an Azure AI Foundry resource. This provides a simplified setup and coding, improved governance and management.

![Foundry](/foundry.png)

Compared to the five Azure resources required for the old construct (Key Vault, AI Services, Storage Account, Azure AI Hub, Azure AI Project), it only needs one Azure AI Foundry and a project resource for the new approach. 

![Resources](/resources.png)

There is a great blog article covering the changes, [here](https://techcommunity.microsoft.com/blog/aiplatformblog/build-recap-new-azure-ai-foundry-resource-developer-apis-and-tools/4427241). In terms of capabilities of the different project types, there is a comparison matrix and additional information to be found, [here](https://learn.microsoft.com/en-us/azure/ai-foundry/what-is-azure-ai-foundry#which-type-of-project-do-i-need). 

### API & SDK changes

From an API and SDK perspective, there were also a couple of changes introduced to unify the development of the core building blocks of your AI application. Here are a few examples:

- Connecting via the *PROJECT_CONNECTION_STRING* is not possible anymore and got changed to **PROJECT_ENDPOINT**, see following example:
```python
# create ai project client
project_client = AIProjectClient(
  endpoint=PROJECT_ENDPOINT,
  credential=DefaultAzureCredential()
)
```

The AI Foundry project endpoint url can be found under the **Overview** tab of your project, see following screenshot:

![Project endpoint](/endpoint.png)


- Creating a thread changed from *project_client.agents.create_thread* to **project_client.agents.threads.create**, see following example:
```python
# create a thread
thread = project_client.agents.threads.create()
```

- Creating a message changed from *project_client.agents.create_message* to **project_client.agents.messages.create**
```python
# create a message
message = project_client.agents.messages.create(
    thread_id=thread.id,
    role="user",
    content="Who is the greatest Basketball player of all time?",
    )
```

- Creating a run changed from *project_client.agents.create_and_process_run* to **project_client.agents.runs.create_and_process**, see example here:
```python
# ask the agent to perform work on the thread
run = project_client.agents.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)
```
- ...
  
The blog post [here](https://devblogs.microsoft.com/foundry/coding-the-future-of-ai-with-azure-ai-foundry-api-and-sdk/) introduces the new Azure AI Foundry API and SDK with various examples. 

If you were following my previous blog posts about Azure AI Agent Service and Azure AI Foundry, you will notice that the code on the blog posts is still using the old approach. However, on the referenced [GitHub repo](https://github.com/beyondelastic/basketball-ai-agent/tree/main), I have already changed the code to work with the latest changes. 

### Build and deploy Azure AI Foundry Agents via VS Code

If you haven't done it already, you should take a look at the [Azure AI Foundry VS Code extension](https://marketplace.visualstudio.com/items?itemName=TeamsDevApp.vscode-ai-foundry) (currently in Preview). It allows you to build, test, and deploy Azure AI Foundry Agents from within VS Code. Agents and tools can be defined via yaml files as you can see in the following screenshot:

![VS Code - Foundry Agent yaml](/vscode-foundry-agent-yaml.jpg)

Additionally, they can be defined and deployed via a graphical editor in VS Code, as you can see here:

![VS Code - Foundry Agent editor](/vscode-foundry-agent-editor.png)

Check out the following [blog post](https://techcommunity.microsoft.com/blog/azuredevcommunityblog/create-enterprise-ai-agents-with-azure-ai-foundry-vscode-extension/4400803) and the documentation link [here](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/develop/vs-code-agents) if you are keen to learn more about working with Azure AI Foundry from within VS Code. 

### Open in VS Code workflow

Another nice little new feature is the option to open a [VS Code for the Web](https://code.visualstudio.com/docs/setup/vscode-web) session directly from the Azure AI Foundry Agent playground. 

![Foundry playground](/playground.png)

From the sample code page, you can press **Open in VS Code**.

![Open in VS Code](/open-in-vscode.png)

This will open a **VS Code for the Web** session.

![VS Code for the Web](/vscode-web.png)

If you want to learn more about this feature, have a look at the following [blog post](https://devblogs.microsoft.com/foundry/open-in-vscode/). 

### Infrastructure templates

In terms of infrastructure deployment, you can find bicep templates creating the new Azure AI Foundry resources [here](https://github.com/azure-ai-foundry/foundry-samples/blob/main/samples/microsoft/infrastructure-setup/README.md). 

## Summary

If you are already using Azure AI Foundry and you are using the Hub-based construct, you can continue using it, as it will remain accessible for now. However, if you are not using any feature that is solely available on Hub-based projects, you should consider moving to the new Foundry-based projects, as new capabilities and services in GA (e.g., Foundry Agent Service & Foundry API) will only be made available there.

As I spend most of my time in VS Code, I love the integration to Azure AI Foundry via the extension. It simply is a big time saver, and I can iterate fast from within VS Code. If you haven't, you should definitely check it out.  

## Sources

- [Announcing General Availability of Azure AI Foundry Agent Service](https://techcommunity.microsoft.com/blog/azure-ai-services-blog/announcing-general-availability-of-azure-ai-foundry-agent-service/4414352)
- [What's new in Azure AI Foundry - Microsoft Build 2025](https://azure.microsoft.com/en-us/blog/azure-ai-foundry-your-ai-app-and-agent-factory/)
- [Build recap: new Azure AI Foundry resource, Developer APIs and Tools](https://techcommunity.microsoft.com/blog/aiplatformblog/build-recap-new-azure-ai-foundry-resource-developer-apis-and-tools/4427241)
- [Documentation & comparison matrix Azure AI Foundry projects](https://learn.microsoft.com/en-us/azure/ai-foundry/what-is-azure-ai-foundry#which-type-of-project-do-i-need)
- [Coding the Future of AI with Azure AI Foundry API and SDK](https://devblogs.microsoft.com/foundry/coding-the-future-of-ai-with-azure-ai-foundry-api-and-sdk/)
- [Create Enterprise AI Agents with Azure AI Foundry VSCode Extension](https://techcommunity.microsoft.com/blog/azuredevcommunityblog/create-enterprise-ai-agents-with-azure-ai-foundry-vscode-extension/4400803)
- [Documentation - Work with Azure AI Foundry Agent Service in Visual Studio Code (Preview)](https://learn.microsoft.com/en-us/azure/ai-foundry/how-to/develop/vs-code-agents)
- [Azure AI Foundry bicep templates](https://github.com/azure-ai-foundry/foundry-samples/blob/main/samples/microsoft/infrastructure-setup/README.md)
- [Documentation - VS Code for the Web](https://code.visualstudio.com/docs/setup/vscode-web)
- [Code quicker with Azure AI Foundry playgrounds and Visual Studio Code](https://devblogs.microsoft.com/foundry/open-in-vscode/)

