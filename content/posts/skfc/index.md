+++
date = '2025-04-23T09:42:05+02:00'
draft = false
title = 'Semantic Kernel Function Calling & Plugins'
categories = ['AI']
tags = ['Agents', 'Azure', 'Semantic Kernel', 'Function Calling', 'Plugins']
+++

# Intro

These days, LLMs are not good enough anymore. We build agentic applications that perform complex tasks with up-to-date data and external systems. This is why we need multiple specialized AI Agents that can access all sorts of APIs and data sources to provide additional context to our LLMs. In this blog post, I want to take a closer look at Semantic Kernel Function Calling & Plugins and how it can help in such scenarios. 

## Semantic Kernel

Semantic Kernel is a lightweight multiagent orchestration framework. It allows us to build enterprise-grade multiagent AI apps by combining and integrating multiple AI models and systems. In one of my last [blog posts](https://beyondelastic.github.io/posts/multi-agent/), I explained how to use Semantic Kernel to have multiple AI agents interacting with each other via a group chat. Today we are going to look at extensibility! 

### Function Calling

What is [function calling](https://learn.microsoft.com/en-us/semantic-kernel/concepts/ai-services/chat-completion/function-calling/?pivots=programming-language-python)? Function calling is a feature that enables AI models to request the execution of specific functions, allowing them to perform actions based on user inputs. Semantic Kernel then marshals the request to the appropriate function in your codebase and returns the results to the LLM, so the LLM can generate a final response. This capability is particularly useful for enabling AI to interact with and invoke your APIs. It is a native feature in most of the latest large language models (LLMs), facilitating planning and execution of tasks. 

### Plugins

Aren't [plugins](https://learn.microsoft.com/en-us/semantic-kernel/concepts/plugins/?pivots=programming-language-python) the same? Yes and no. At a high level, a plugin is a collection of functions that can be made available to AI applications and services. These functions can be orchestrated by an AI application to fulfill user requests. Behind the scenes, Semantic Kernel utilizes function calling to enable this process. Additionally, plugins support dependency injection, allowing essential services such as database connections or HTTP clients to be injected into the plugin's constructor.

If you do some research about Semantic Kernel plugins and function calling, you  might come across **skills**. What used to be skills became plugins to align with the OpenAI plugin specification. You can read more about it in the following post:

- [Skills to plugins: fully embracing the OpenAI plugin spec in Semantic Kernel](https://devblogs.microsoft.com/semantic-kernel/skills-to-plugins-fully-embracing-the-openai-plugin-spec-in-semantic-kernel/)

There are three main ways to get plugins into Semantic Kernel:

1. Using [native code](https://learn.microsoft.com/en-us/semantic-kernel/concepts/plugins/adding-native-plugins?pivots=programming-language-python)
2. Using an [OpenAPI specification](https://learn.microsoft.com/en-us/semantic-kernel/concepts/plugins/adding-openapi-plugins?pivots=programming-language-python)
3. Using an [MCP Server](https://learn.microsoft.com/en-us/semantic-kernel/concepts/plugins/adding-mcp-plugins?pivots=programming-language-python)

The latter two are programming languages and platform-agnostic. However, we will use the **native code** option as it is the easiest to start with. 

Enough theory, let's do some coding!

## Coding

For this demonstration, I want to write a simple Semantic Kernel AI app that uses a native code plugin. For those who know me, I am a huge 🏀 Basketball fan. In my spare time, I coach a children's team in my hometown, and I still play Basketball myself. The Basketball EuroLeague is currently in playoff mode, and I like to follow the latest game results. That's why I am going to write a plugin that uses the [EuroLeague API](https://api-live.euroleague.net/swagger/index.html) to provide the latest game results to my LLM. 

### Writing our Semantic Kernel plugin

When writing the plugin, it is important to give special attention to function naming. It is recommended to use [snake case](https://en.wikipedia.org/wiki/Snake_case) for function names and properties, as most models are trained with Python for function calling. The model needs to understand its intent and parameters. If needed, you can, and in some cases, you should add a description to it, but it will increase the token usage. In our case, it is a simple function with only one input parameter and a self-explanatory name, **get_latest_euroleague_game_results**. Nevertheless, I have annotated the **season** input parameter for the model to know what is expected. 

All we really need to do is create a class and annotate its methods with the [**@kernel_function**](https://learn.microsoft.com/en-us/python/api/semantic-kernel/semantic_kernel.functions?view=semantic-kernel-python) attribute. This way, Semantic Kernel knows that this is a function that can be called by our LLM. Note that the helper function **xml_to_dict** is not annotated and won't be sent to the model. 

```python
import xml.etree.ElementTree as ET
import httpx
from typing import Any, Dict, Annotated
from datetime import datetime
from semantic_kernel.functions import kernel_function

# This plugin fetches EuroLeague game results and processes them.
class EuroleaguePlugin:

    # This function converts an XML element and its children to a dictionary.
    def xml_to_dict(self, element: ET.Element) -> Any:
        """Recursively converts an XML element and its children to a dictionary."""
        node = {}
        if element.attrib:
            node.update(element.attrib)
        children = list(element)
        if children:
            for child in children:
                child_dict = self.xml_to_dict(child)
                if child.tag not in node:
                    node[child.tag] = []
                node[child.tag].append(child_dict)
        else:
            node = element.text or ""
        return node

    # This function fetches the latest six EuroLeague game results for a given season.
    @kernel_function
    async def get_latest_euroleague_game_results(self, season: Annotated[str, "The year of the season, e.g. 2024 for season 2024/25"]) -> Dict[str, Any]:
        """Fetches EuroLeague game results and returns only the last 6 games by date."""
        url = "https://api-live.euroleague.net/v1/results"
        params = {"season_code": "E" + season}
        headers = {"Accept": "application/xml"}
        try:
            async with httpx.AsyncClient() as client:
                response = await client.get(url, params=params, headers=headers)
                response.raise_for_status()
                root = ET.fromstring(response.text)
                data = self.xml_to_dict(root)

            # Extract the games from the parsed XML data
            games = data.get("game", [])  

            # Parse and sort games by date
            def parse_date(game):
                
                # The date is a list, so get the first element
                date_val = game.get("date", [""])
                if isinstance(date_val, list):
                    date_str = date_val[0]
                else:
                    date_str = date_val
                try:
                    return datetime.strptime(date_str, "%b %d, %Y")
                except Exception:
                    return datetime.min
                
            # Sort the games by date in descending order
            games_sorted = sorted(games, key=parse_date, reverse=True)
            last_6_games = games_sorted[:6]

            # Replace the games in the data dict with only the last 6
            data["game"] = last_6_games
            return data
        
        # Exception handling
        except Exception as e:
            print(f"Exception when making direct request: {e}")
            return {}
```

So far, so good. We have a plugin. What is doing exactly? It is getting the latest six Euroleague game results of a specific season. The helper function xml_to_dict converts the XML response from the EuroLeague API to a Python dictionary, so it is easier to process for the model. We then sort the data in descending order by date and truncate it, leaving only the last six games. 

### Writing our Semantic Kernel AI app

Now that we have a plugin, let's write our AI app that will use the plugin. This script uses Azure OpenAI [**chat completion**](https://learn.microsoft.com/en-us/semantic-kernel/concepts/ai-services/chat-completion/), adds our Euroleague plugin to the kernel, and creates a [**chat history**](https://learn.microsoft.com/en-us/semantic-kernel/concepts/ai-services/chat-completion/chat-history?pivots=programming-language-python) to interact with the assistant.

Additionally, we define the [**AzureChatPromptExecutionSettings**](https://learn.microsoft.com/en-us/python/api/semantic-kernel/semantic_kernel.connectors.ai.open_ai.azurechatpromptexecutionsettings?view=semantic-kernel-python) to configure how prompts are executed when interacting with Azure OpenAI. It allows you to set options such as function calling behavior, temperature, and other model parameters. In our case, we set the [**FunctionChoiceBehavior**](https://learn.microsoft.com/en-us/semantic-kernel/concepts/ai-services/chat-completion/function-calling/function-choice-behaviors?pivots=programming-language-python#using-auto-function-choice-behavior) to auto. This setting lets the model automatically select the most relevant function based on the user's input.

Lastly, the script sends a user message requesting the latest Euroleague game results and prints the AI's response.

```python
import asyncio
import os
import euroleague_plugin

from dotenv import load_dotenv
from semantic_kernel import Kernel
from semantic_kernel.connectors.ai.open_ai import AzureChatCompletion, AzureChatPromptExecutionSettings
from semantic_kernel.connectors.ai import FunctionChoiceBehavior
from semantic_kernel.contents import ChatHistory

load_dotenv()

# Azure OpenAI config (set your environment variables)
AZURE_OPENAI_KEY = os.getenv("AZURE_OPENAI_KEY")
AZURE_OPENAI_ENDPOINT = os.getenv("AZURE_OPENAI_ENDPOINT")
AZURE_OPENAI_DEPLOYMENT = os.getenv("AZURE_OPENAI_DEPLOYMENT")

async def main():
   # Initialize the kernel
   kernel = Kernel()
   
   # Add Azure OpenAI chat completion
   chat_completion = AzureChatCompletion(
      deployment_name=AZURE_OPENAI_DEPLOYMENT,
      api_key=AZURE_OPENAI_KEY,
      endpoint=AZURE_OPENAI_ENDPOINT,
   )
   kernel.add_service(chat_completion)

   # Add a plugin (the EuroleaguePlugin class is defined above)
   kernel.add_plugin(
      euroleague_plugin.EuroleaguePlugin(),
      plugin_name="Euroleague"
   )
   
   # Enable planning
   execution_settings = AzureChatPromptExecutionSettings()
   execution_settings.function_choice_behavior = FunctionChoiceBehavior.Auto()

   # Create a history of the conversation
   history = ChatHistory()
   history.add_user_message("Please give me the latest Euroleague game results for the 2024 season.")
   
   # Get the response from the AI
   result = await chat_completion.get_chat_message_content(
      chat_history=history,
      settings=execution_settings,
      kernel=kernel,
   ) 
   
   # Print the results
   print("Assistant > " + str(result))

   # Add the message from the agent to the chat history
   history.add_message(result)

if __name__ == "__main__":
    asyncio.run(main())
```

Let's see if our code works as expected and if our AI app will indeed use the plugin to fetch the latest game results from the EuroLeague API. 

### Running our Semantic Kernel AI app

**Note:** We haven't covered how to create the Azure OpenAI Service or deployment. If you need some guidance, you can follow the documentation [here](https://learn.microsoft.com/en-us/powershell/utility-modules/aishell/developer/deploy-azure-openai?view=ps-modules). Make sure you have an up-to-date model (e.g., gpt-4o) deployed and a local .env file with the endpoint, key and deployment name filled out. 

I will simply execute the app in my terminal and see if we get the expected response back.

```bash
➜ python app3.py
Assistant > Here are the latest Euroleague game results for the 2024 season:

1. **April 24, 2025** 
   - **Panathinaikos Aktor Athens** vs. **Anadolu Efes Istanbul**: 76 - 79 
   - **Fenerbahce Beko Istanbul** vs. **Paris Basketball**: 89 - 72

2. **April 23, 2025** 
   - **Olympiacos Piraeus** vs. **Real Madrid**: 84 - 72
   - **AS Monaco** vs. **FC Barcelona**: 97 - 80

3. **April 22, 2025**
   - **Fenerbahce Beko Istanbul** vs. **Paris Basketball**: 83 - 78
   - **Panathinaikos Aktor Athens** vs. **Anadolu Efes Istanbul**: 87 - 83

These games were part of the playoffs.
```

Great! This is accurate and everything I wanted. The model provides me with the latest six-game results that happened just a couple of days ago. We can verify if the results are correct by checking the [EuroLeague Game Center](https://www.euroleaguebasketball.net/en/euroleague/game-center/) page. 

If you want to check out the code, you can find it on my GitHub [repo](https://github.com/beyondelastic/basketball-ai-agent). 

## Summary

This was a very straightforward example of how to use Semantic Kernel plugins to reach out to external systems to provide up-to-date data to our LLM. Encapsulating native code within a plugin is a simple method to equip an AI agent with capabilities that it doesn't inherently support. This approach enables you to utilize your existing app development skills and code to enhance the functionality of your AI agents. Besides native code, MCP, and OpenAPI plugins, we can also make use of Retrieval Augmented Generation (RAG) patterns when it comes to data access and grounding. For example, we can build a data retrieval plugin using Azure AI Search as described [here](https://learn.microsoft.com/en-us/semantic-kernel/concepts/plugins/using-data-retrieval-functions-for-rag).

Plenty of possibilities to make our AI agents smarter. I am especially excited about the MCP option, as I recently wrote an intro blog post to [MCP](https://beyondelastic.github.io/posts/mcp/), and I can't wait to combine it with Semantic Kernel. 

## Sources

- [Semantic Kernel function calling](https://learn.microsoft.com/en-us/semantic-kernel/concepts/ai-services/chat-completion/function-calling/?pivots=programming-language-python)
- [Semantic Kernel plugins](https://learn.microsoft.com/en-us/semantic-kernel/concepts/plugins/?pivots=programming-language-python)
- [Skills to plugins: fully embracing the OpenAI plugin spec in Semantic Kernel](https://devblogs.microsoft.com/semantic-kernel/skills-to-plugins-fully-embracing-the-openai-plugin-spec-in-semantic-kernel/)
- [Semantic Kernel plugins - native code](https://learn.microsoft.com/en-us/semantic-kernel/concepts/plugins/adding-native-plugins?pivots=programming-language-python)
- [Semantic Kernel plugins - OpenAPI specification](https://learn.microsoft.com/en-us/semantic-kernel/concepts/plugins/adding-openapi-plugins?pivots=programming-language-python)
- [Semantic Kernel plugins - MCP Server](https://learn.microsoft.com/en-us/semantic-kernel/concepts/plugins/adding-mcp-plugins?pivots=programming-language-python)
- [Snake case](https://en.wikipedia.org/wiki/Snake_case)
- [Semantic Kernel KernelFunctions](https://learn.microsoft.com/en-us/python/api/semantic-kernel/semantic_kernel.functions?view=semantic-kernel-python)
- [Semantic Kernel ChatCompletion](https://learn.microsoft.com/en-us/semantic-kernel/concepts/ai-services/chat-completion/)
- [Semantic Kernel ChatHistory](https://learn.microsoft.com/en-us/semantic-kernel/concepts/ai-services/chat-completion/chat-history?pivots=programming-language-python)
- [Semantic Kernel AzureChatPromptExecutionSettings](https://learn.microsoft.com/en-us/python/api/semantic-kernel/semantic_kernel.connectors.ai.open_ai.azurechatpromptexecutionsettings?view=semantic-kernel-python)
- [Semantic Kernel FunctionChoiceBehavior](https://learn.microsoft.com/en-us/semantic-kernel/concepts/ai-services/chat-completion/function-calling/function-choice-behaviors?pivots=programming-language-python#using-auto-function-choice-behavior)
- [Deploy Azure OpenAI via Bicep](https://learn.microsoft.com/en-us/powershell/utility-modules/aishell/developer/deploy-azure-openai?view=ps-modules)
- [Semantic Kernel RAG plugin](https://learn.microsoft.com/en-us/semantic-kernel/concepts/plugins/using-data-retrieval-functions-for-rag)
- [EuroLeague Game Center](https://www.euroleaguebasketball.net/en/euroleague/game-center/)
- [EuroLeague API](https://api-live.euroleague.net/swagger/index.html)












