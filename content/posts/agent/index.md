+++
date = '2025-04-02T13:15:46+02:00'
draft = false
title = 'Intro to Azure AI Agent Service'
categories = ['AI']
tags = ['Agents', 'Foundry', 'Azure']
+++

# Intro

AI Agents are in talks, and we are going to take a look at the **Azure AI Agent Service**. Why should you care? Because AI Agents are the next evolution of AI-driven applications, and they will help you to automate and execute more complicated multistep tasks completely autonomously or with a human in a loop. Here are two blog posts that are worth reading to set the scene:

- [AI agents — what they are, and how they’ll change the way we work](https://news.microsoft.com/source/features/ai/ai-agents-what-they-are-and-how-theyll-change-the-way-we-work/)
- [Introducing Azure AI Agent Service](https://techcommunity.microsoft.com/blog/azure-ai-services-blog/introducing-azure-ai-agent-service/4298357)

In this blog post, I will give you a short overview about what the Azure AI Agent Service is, what it can do for you, and how to quickly get started via the provided Python [Azure AI Foundry SDK](https://aka.ms/aifoundrysdk).

**NOTE**: At the time of writing, the Azure AI Agent Service is in public preview. 

## Overview

The Azure [Azure AI Agent Service](https://learn.microsoft.com/en-us/azure/ai-services/agents/overview) is part of [Azure AI Foundry](https://learn.microsoft.com/en-us/azure/ai-foundry/what-is-ai-foundry). Azure AI Foundry is a unified AI platform that allows you to manage the whole lifecycle of your AI application. Key features, are:
  
- Rich model catalog (OpenAI, DeepSeek, Cohere, Meta, Mistral...)
- Deploy and experiment with different models in playgrounds
- Seamless Integration to other Azure services such as Azure OpenAI, Azure AI Services, and Azure AI Search
- Project Management with features for project creation, resource management, and access control
- Simplified coding experience with unified SDK
- Build and deploy AI agents with the Azure AI Agent Service
- and more...

With the Azure AI Agent Service, Developers can easily build extensible AI Agents using out of the box tooling and integrations into Azure services. Some of the highlights are:

- Flexible model selection (not solely OpenAI models)
- Knowledge tools such as Azure AI Search, Grounding with Bing Search, Microsoft Fabric and file uploads 
- Action tools such as OpenAPI 3.0 specified tools and Code Interpreter, Azure Functions and custom functions 
- Conversation state management (providing consistent context)
- and more...

## Requirements

To start with Azure AI Agent Service, we first need to deploy the necessary Azure Services and create an Azure AI Foundry Hub and Project. Additionally, we will prepare our local development environment. 

### Prepare infrastructure

In this blog post, we want to focus on how to use these services from a coding perspective. Hence, we are not going to cover the infrastructure deployment in much detail. However, we will simply use the provided bicep template from the Azure-Samples repository [here](https://github.com/Azure-Samples/azureai-samples/tree/main/scenarios/Agents/setup/standard-agent).

After a successful deployment, we should see the following services in your resource group.

![Azure Resources](/resources.png)

The Azure AI Foundry portal can be found under https://ai.azure.com/. We should now have an Azure AI Foundry Hub and Project created for us. 

![Foundry Hub / Project](/foundry-hub.png)

If you want to learn more about Azure AI Foundry Hubs and Projects, visit the documentation page [here](https://learn.microsoft.com/en-us/azure/ai-foundry/concepts/ai-resources).

We can also view the connected resources that have been created as part of the deployment and can be used for the AI agents that we want to build. 

![Connected resources](/project-resources.png)

### Prepare local dev environment

For our local development environment, we need to install the following packages:
```sh
pip install python-dotenv azure-ai-projects azure-identity 
```

Next, we will use a local .env file to specify some variables to connect to our Azure AI Foundry Project. 

````
AZURE_AI_AGENT_PROJECT_CONNECTION_STRING="your_project_connection_string"
AZURE_AI_AGENT_MODEL_DEPLOYMENT_NAME="your_model_deployment"
````


## Coding

If everything is running we can start coding. My goal is to build an AI Agent that acts as a Basketball Assistant Coach. For those who don't know me, I am a big Basketball fan and combing two things that I love is pure excitement for me. 

### Building the app

First we need to import the necessary classes from the SDK. In this case we are using the python SDK. 

```python
import os
from dotenv import load_dotenv
from azure.identity import DefaultAzureCredential
from azure.ai.projects import AIProjectClient
from azure.ai.projects.models import CodeInterpreterTool, FilePurpose
from pathlib import Path
```

We will then load the variables from the .env file, and create the AIProjectClient and connect to our Azure AI Foundry Project.

```python
# load environment variables from local .env file
load_dotenv()
PROJECT_CONNECTION_STRING = os.getenv("AZURE_AI_AGENT_PROJECT_CONNECTION_STRING")
MODEL_DEPLOYMENT = os.getenv("AZURE_AI_AGENT_MODEL_DEPLOYMENT_NAME")

# create ai project client
project = AIProjectClient.from_connection_string(
  conn_str=PROJECT_CONNECTION_STRING,
  credential=DefaultAzureCredential()
)
with project_client:
```

We want our agent (Assistant Coach) to be able to analyze and interpret certain Basketball statistics. In this case, we are going to upload a csv file with statistics about 3-point shots from the NBA seasons 1996 until 2020. To achieve that, we will make use of the [CodeInterpreterTool](https://learn.microsoft.com/en-us/azure/ai-services/agents/how-to/tools/code-interpreter?tabs=python&pivots=code-example).

```python
    # upload a file and add it to the client 
    file = project_client.agents.upload_file_and_poll(
        file_path="nba3p.csv", purpose=FilePurpose.AGENTS
    )
    print(f"Uploaded file, file ID: {file.id}")

    # create a code interpreter tool instance referencing the uploaded file
    code_interpreter = CodeInterpreterTool(file_ids=[file.id])
```

Now we can define the agent with instructions and tools. 

```python
    # create an agent
    agent = project_client.agents.create_agent(
        model="gpt-4o-mini",
        name="assistant-coach-agent",
        instructions="You are a Basketball assistant coach that knows everything about the game of Basketball. You give advice about Basketball rules, training, statistics and strategies.",
        tools=code_interpreter.definitions,
        tool_resources=code_interpreter.resources,
    )
    print(f"Using agent: {agent.name}")
```

Next, we create a thread, which is basically the conversation between the user and the agent. In the message itself, we define the ask or task for our agent. 

```python
    # create a thread with message
    thread = project_client.agents.create_thread()
    print(f"Thread created: {thread.id}")

    message = project_client.agents.create_message(
        thread_id=thread.id,
        role="user",
        content="Could you please create a bar chart for 3Pointers made vs attempts during the NBA seasons 1996 until 2020 and save it as a .png file?",
    )
```

Almost done, we now have to specify the run or activation part in which the agent will perform our task. Additionally, we have to fetch the output in reversed chronological order, providing a clear view of the most recent interactions first. 

```python
    # ask the agent to perform work on the thread
    run = project_client.agents.create_and_process_run(thread_id=thread.id, agent_id=agent.id)

    # fetch and print the conversation history including the last message
    print("\nConversation Log:\n")
    messages = project_client.agents.list_messages(thread_id=thread.id)
    for message_data in reversed(messages.data):
        last_message_content = message_data.content[-1]
        print(f"{message_data.role}: {last_message_content.text.value}\n")
```

The agent should generate a graph png that we want to look at. Hence, we have to download the file with the following code snippet. 

```python
    # fetch any generated files
    for file_path_annotation in messages.file_path_annotations:
        project_client.agents.save_file(file_id=file_path_annotation.file_path.file_id, file_name=Path(file_path_annotation.text).name)
        print(f"File saved as {Path(file_path_annotation.text).name}")
```

Lastly, we should clean up and delete the agent and the thread.

```python
    # clean up
    project_client.agents.delete_agent(agent.id)
    project_client.agents.delete_thread(thread.id)
```

### Running the app

**Note**: The code snippets above should just give you an overview of the Azure AI Agent Service and are not representing a production-grade application. Error handling, logging, config management and reusability are just some things you need to add to your final application code. Nevertheless, you can find the application code and the used files on my GitHub repo [here](https://github.com/beyondelastic/basketball-ai-agent/tree/main). 

Let's run our code and see the results.

```bash
(.venv) ➜  basketball-ai-agent python main.py
Uploaded file, file ID: assistant-J7sVvG3wVZXRkXHwSxELJY
Using agent: assistant-coach-agent
Thread created: thread_zqMD7857xH4zsKwN6olVvkmx

Conversation Log:

MessageRole.USER: Could you please create a bar chart for 3Pointers made vs attempts during the NBA seasons 1996 until 2020 and save it as a .png file?

MessageRole.AGENT: Let's start by examining the contents of the uploaded file to understand the data it contains. After that, we can create the bar chart for 3-pointers made versus attempts for the NBA seasons from 1996 to 2020.

MessageRole.AGENT: The uploaded data contains the following columns:

- **NBASeasons**: The season year.
- **3PointersMade**: The number of 3-pointers made.
- **3PointersAttempts**: The number of 3-pointers attempted.
- **3PointersPercentage**: The percentage of successful 3-point shots.
- **3PointersPercentageShareInTotalPoints**: The share of 3-pointers in total points.

We'll create a bar chart comparing the number of 3-pointers made versus the number of 3-pointers attempted for the NBA seasons from 1996 to 2020. Let's proceed with that.

MessageRole.AGENT: Here is the bar chart comparing 3-pointers made versus 3-pointers attempted from the NBA seasons 1996 to 2020. 

You can download it using the link below:

[Download the chart](sandbox:/mnt/data/3_pointers_made_vs_attempts_1996_2020.png)

File saved as 3_pointers_made_vs_attempts_1996_2020.png
```

During the execution of our app, we can see the agent and thread getting created in Azure AI Foundry portal.

![Agents](/agents.jpg)
![Threads](/threads.png)

After the run the agent and the thread will be deleted due to our clean-up code. If you want to keep it for troubleshooting proposes, just comment or remove the clean-up section.

In the Agents view we can also see the Code Interpreter got added as action tool for our agent, and we can see the file we have uploaded.

![Action tools](/action-tools.jpg)
![Files uploaded](/files-uploaded.png)

Let's check the results. The graph generated is correct and shows quite a raise in 3 Pointers made and attempts. Thanks [Steph Curry](https://de.wikipedia.org/wiki/Stephen_Curry). 

![3Pointers Graph](/3_pointers_made_vs_attempts_1996_2020.png)

## Summary

The Azure AI Agent Service allows you to quickly and with less effort build, deploy and manage AI Agents. The comprehensive out of the box toolkit and the possibility to select different models give Developers the flexibility they need to develop agent-based AI apps that can successfully accomplish complex tasks. 

The Azure AI Agent Service can also easily be used with multi-agent frameworks such as [Semantic Kernel](https://learn.microsoft.com/en-us/semantic-kernel/overview/). Stay tuned for another blog post about multi-agent apps. 

## Resources

- [AI agents — what they are, and how they’ll change the way we work](https://news.microsoft.com/source/features/ai/ai-agents-what-they-are-and-how-theyll-change-the-way-we-work/)
- [Introducing Azure AI Agent Service](https://techcommunity.microsoft.com/blog/azure-ai-services-blog/introducing-azure-ai-agent-service/4298357)
- [What is Azure AI Agent Service](https://learn.microsoft.com/en-us/azure/ai-services/agents/overview)
- [Azure AI Agent standard deployment via bicep template from Azure AI Samples repository](https://github.com/Azure-Samples/azureai-samples/tree/main/scenarios/Agents/setup/standard-agent)
- [Azure AI Agent Code Interpreter](https://learn.microsoft.com/en-us/azure/ai-services/agents/how-to/tools/code-interpreter?tabs=python&pivots=code-example)
- [What is Azure AI Foundry](https://learn.microsoft.com/en-us/azure/ai-foundry/what-is-ai-foundry)
- [Azure AI Foundry SDK](https://aka.ms/aifoundrysdk)
- [Azure AI Foundry Hubs and Projects ](https://learn.microsoft.com/en-us/azure/ai-foundry/concepts/ai-resources)


