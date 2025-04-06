+++
date = '2025-03-25T09:15:23+01:00'
draft = true
title = 'Multi Agent'
+++

# AI Agent Service

## Tools

Through the use of tools, you can provide your agent additional functionality to execute actions on your behalf.

The AI Agent Service provides built-in tools for gathering knowledge and interpreting code, which provide your agent with some powerful functionality. However, sometimes your agent needs to be able to complete specific tasks or actions that an AI model would struggle to handle on its own. To accomplish these actions, you can provide your agent a custom tool.

Benefits of custom tools

- Enhanced productivity: Automate repetitive tasks and streamline workflows.
- Improved accuracy: Provide precise and consistent outputs, reducing human error.
- Tailored solutions: Address specific business needs and optimize processes.

![agent-tool-diagram](/agent-tool-diagram.png)

The diagram shows the process of an agent choosing to use the provided tool:

1. A user asks an agent for a report, such as about recent snowfall in their local mountains.
2. The agent determines the provided tool to retrieve snowfall for a specific location will be useful, and calls that tool.
3. The agent may choose to use other tools to best accomplish the user's task, such as retrieve knowledge using a built-in tool.
4. The agent then outputs the report and responds to the user.

Tool Options:

- OpenAPI specified tools: These tools allow you to connect your Azure AI Agent to an external API using an OpenAPI 3.0 specification. This provides standardized, automated, and scalable API integrations that enhance the capabilities of your agent. OpenAPI specifications describe HTTP APIs, enabling people to understand how an API works, generate client code, create tests, and apply design standards.
- Function calling: Function calling allows you to describe the structure of functions to an agent and return the functions that need to be called along with their arguments. This feature is useful for integrating custom logic and workflows into your AI agents.
- Azure Functions: Azure Functions enable you to create intelligent, event-driven applications with minimal overhead. They support triggers and bindings, which simplify how your AI Agents interact with external systems and services. Triggers determine when a function executes, while bindings facilitate streamlined connections to input or output data sources.

Example - Customer Support Automation

Scenario: A retail company integrates a custom tool that connects the Azure AI Agent to their customer relationship management (CRM) system.
Functionality: The AI agent can retrieve customer order histories, process refunds, and provide real-time updates on shipping statuses.
Outcome: Faster resolution of customer queries, reduced workload for support teams, and improved customer satisfaction.

Example - Inventory Management

Scenario: A manufacturing company develops a custom tool to link the AI agent with their inventory management system.
Functionality: The AI agent can check stock levels, predict restocking needs using historical data, and place orders with suppliers automatically.
Outcome: Streamlined inventory processes and optimized supply chain operations.

## Defining and using a function

Snowfall tracking example:

```
import json

def recent_snowfall(location: str) -> str:
    """
    Fetches recent snowfall totals for a given location.
    :param location: The city name.
    :return: Snowfall details as a JSON string.
    """
    mock_snow_data = {"Seattle": "0 inches", "Denver": "2 inches"}
    snow = mock_snow_data.get(location, "Data not available.")
    return json.dumps({"location": location, "snowfall": snow})

user_functions: Set[Callable[..., Any]] = {
    recent_snowfall,
}
```

Register the function with your agent using the Azure AI Agent SDK:

```
# Initialize agent toolset with user functions
functions = FunctionTool(user_functions)
toolset = ToolSet()
toolset.add(functions)

# Create your agent with the toolset
agent = project_client.agents.create_agent(
    model="gpt-4o-mini",
    name="snowfall-agent",
    instructions="You are a weather assistant tracking snowfall. Use the provided functions to answer questions.",
    toolset={"functions": [recent_snowfall]}
)
```

The agent can now call recent_snowfall dynamically when prompted by the user.

## OpenAPI defined tools

OpenAPI defined tools allow agents to interact with external APIs using standardized specifications. This approach simplifies API integration and ensures compatibility with various services. Azure AI Agent Service uses OpenAPI 3.0 specified tools.

Currently, three authentication types are supported with OpenAPI 3.0 tools: anonymous, API key, and managed identity.

Prepare the OpenAPI spec: Create a JSON file (snowfall_openapi.json) describing the API.

```
{
  "openapi": "3.0.0",
  "info": {
    "title": "Snowfall API",
    "version": "1.0.0"
  },
  "paths": {
    "/snow": {
      "get": {
        "summary": "Get snowfall information",
        "parameters": [
          {
            "name": "location",
            "in": "query",
            "required": true,
            "schema": {
              "type": "string"
            }
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {
                "schema": {
                  "type": "object",
                  "properties": {
                    "location": {"type": "string"},
                    "snow": {"type": "string"}
                  }
                }
              }
            }
          }
        }
      }
    }
  }
}
```

Register the OpenAPI tool:
```
from azure.ai.projects.models import OpenApiTool, OpenApiAnonymousAuthDetails

with open("snowfall_openapi.json", "r") as f:
    openapi_spec = json.load(f)

auth = OpenApiAnonymousAuthDetails()
openapi_tool = OpenApiTool(name="snowfall_api", spec=openapi_spec, auth=auth)

agent = project_client.agents.create_agent(
    model="gpt-4o-mini",
    name="openapi-agent",
    instructions="You are a snowfall tracking assistant. Use the API to fetch snowfall data.",
    tools=[openapi_tool]
)
```

## Using Azure Functions with a queue trigger

````
storage_service_endpoint = "https://<your-storage>.queue.core.windows.net"

azure_function_tool = AzureFunctionTool(
    name="get_snowfall",
    description="Get snowfall information using Azure Function",
    parameters={
            "type": "object",
            "properties": {
                "location": {"type": "string", "description": "The location to check snowfall."},
            },
            "required": ["location"],
        },
    input_queue=AzureFunctionStorageQueue(
        queue_name="input",
        storage_service_endpoint=storage_service_endpoint,
    ),
    output_queue=AzureFunctionStorageQueue(
        queue_name="output",
        storage_service_endpoint=storage_service_endpoint,
    ),
)

agent = project_client.agents.create_agent(
    model=os.environ["MODEL_DEPLOYMENT_NAME"],
    name="azure-function-agent",
    instructions="You are a snowfall tracking agent. Use the provided Azure Function to fetch snowfall based on location.",
    tools=azure_function_tool.definitions,
)
````


The agent can now send requests to the Azure Function via a storage queue and process the results.

By using one of the above methods (or a combination of these options) for implementing a custom tool, you can create powerful, flexible, and intelligent agents with Azure AI Agent Service. These integrations enable seamless interaction with external systems, real-time processing, and scalable workflows, making it easier to build custom solutions tailored to your needs.

# Multi Agent

## Semantic Kernel

A multi-agent solution allows agents to collaborate within the same conversation

Imagine you're trying to address common DevOps challenges such as monitoring application performance, identifying issues, and deploying fixes. A multi-agent system could consist of four specialized agents working collaboratively:

The Monitoring Agent continuously ingests logs and metrics, detects anomalies using natural language processing (NLP), and triggers alerts when issues arise.

The Root Cause Analysis Agent then correlates these anomalies with recent system changes, using machine learning models or predefined rules to pinpoint the root cause of the problem.

Once the root cause is identified, the Automated Deployment Agent takes over to implement fixes or roll back problematic changes by interacting with CI/CD pipelines and executing deployment scripts.

Finally, the Reporting Agent generates detailed reports summarizing the anomalies, root causes, and resolutions, and notifies stakeholders via email or other communication channels.

### Semantic Kernel Agent Framework

The Semantic Kernel Agent Framework is a framework designed to help developers build AI-powered agents. These agents can process user inputs, make decisions, and execute tasks autonomously by leveraging large language models and traditional programming logic. The framework provides structured components for defining AI-driven workflows, enabling agents to interact with users, APIs, and external services

**Core concepts**
The Agent Framework in Semantic Kernel provides architecture on top of existing Semantic Kernel resources, including:

Agents

Agents are intelligent, AI-driven entities capable of reasoning and executing tasks. They use language models, functions, and memory to make decisions dynamically.

Agent collaboration

Agents can collaborate together through an agent group chat, which enables multiple agents to join the same chat, even of different agent types. Agent group chats determine which agent should respond and how to determine if the conversation is finished.

The features that power Semantic Kernel are also still available within the Agent Framework, including:

Kernel

The kernel is the central component of the Semantic Kernel. The kernel acts as the execution engine, managing AI interactions, function orchestration, and memory.

Tools and plugins

Plugins align with existing Semantic Kernel features, enabling agents to dynamically interact with external services or execute complex tasks through function calling. Within the Agent Framework, tools are available to provide extra functionality to your agents, such as file searching or code interpreter, similar to tool usage in Azure AI Agent service. Agents use tools and plugins to perform specific tasks.

History

Agents can maintain chat history across multiple interactions, allowing them to track previous interactions and adapt responses accordingly. The conversation history is always accessible by the agents, either as a whole or for a specific agent's chat history.

### Types of agents

The Semantic Kernel Agent Framework supports several different types of agents, including:

Azure AI Agent - a specialized agent within the Semantic Kernel Agent Framework. The AsureAIAgent type is designed to provide advanced conversational capabilities with seamless tool integration. It automates tool calling and securely manages conversation history using threads, reducing the overhead of maintaining state. The AzureAIAgent also supports a variety of built-in tools, including file retrieval, code execution, and data interaction via Bing, Azure AI Search, Azure Functions, and OpenAPI.

Chat Completion Agent: designed for chat completion and conversation interfaces. The ChatCompletionAgent type mirrors the features and patterns in the underlying AI Service to support natural language processing, contextual understanding, and dialogue management.

OpenAI Assistant Agent: designed for more advanced capabilities and multi-step tasks. The OpenAIAssistantAgent type supports goal-driven interactions with additional features like code interpretation and file search.

## Design an agent selection strategy

Agent collaboration, called AgentGroupChat, has critical components to consider that aren't necessary with single agents or non-agentic Semantic Kernel applications.

The following units discuss an example multi-agent solution, where we have two agents in a writer-reviewer scenario:

A copywriter agent who writes online content, called CopywriterAgent.
A creative director only reviewing the proposals, called ReviewingDirectorAgent.

### Agent selection

Choosing the agent best suited to respond to a user's query is pivotal, especially in multi-agent systems where agents specialize in different domains.

For example, if you chat with the agents asking for a slogan for a new scrubbing brush, the ReviewingDirectorAgent shouldn't be invoked to respond since they don't know how to write slogans. Instead, having the CopywriterAgent respond would provide the user an accurate response.

### How does the framework select agents?

**Single-turn conversations**

Intent recognition: The framework analyzes the user's query to identify the intent and match it with the most relevant agent.
Predefined rules: Developers can configure routing rules to direct specific queries to designated agents in their application.

**Multi-turn conversations**

Context tracking: The framework maintains a record of the conversation history to understand the user's intent and select the appropriate agent.
Dynamic switching: If the topic shifts, the framework dynamically switches to an agent specializing in the new domain in the middle of the conversation.

For multi-turn agents, agent selection is determined by a selection strategy. The selection strategy is defined within the framework, either by using one of the predefined selection strategies or by extending a base class to define custom selection behavior. The selection strategy is defined in the creation of the AgentGroupChat.

Defining your selection function is done by creating a kernel function from a prompt. In our writer and reviewer example, your selection strategy prompt might be:

`````
prompt=f"""
    Determine which participant takes the next turn in a conversation based on the the most recent participant.
    State only the name of the participant to take the next turn.
    No participant should take more than one turn in a row.

    Choose only from these participants:
    - ReviewingDirectorAgent
    - CopywriterAgent

    Always follow these rules when selecting the next participant:
    - After user input, it is CopywriterAgent's turn.
    - After CopywriterAgent replies, it is ReviewingDirectorAgent's turn.
    - After ReviewingDirectorAgent provides feedback, it is CopywriterAgent's turn.

    History:
    {{$history}}
"""
`````
If your preferred interaction should always have a certain agent respond first, that can be specified in your selection strategy as seen in the prompt above.

## Define a chat termination strategy

Multi-turn conversations have responses returned asynchronously, so the conversation can develop naturally. However, the agents need to know when to stop a conversation, which is determined by the termination strategy

A termination strategy ensures that conversations or tasks conclude appropriately. This strategy prevents unnecessary messages to the user, limits resource usage, and improves the overall user experience.

For example, in the writer-reviewer agent scenario, once the ReviewingDirectorAgent reviews and approves our scrubbing brush slogan from the CopywriterAgent, us humans know the conversation should be over. However, if we don't define when to terminate the conversation, the CopywriterAgent is going to keep submitting slogans unnecessarily.

### Why use a termination strategy?

Efficiency: It prevents endless loops or prolonged interactions, saving computational resources.
User satisfaction: Users receive concise and relevant responses, avoiding frustration from overly long conversations.
Goal completion: The use of an agent is to complete a task. By terminating appropriately. it confirms when a task or conversation has achieved its intended purpose.

### How does the framework implement termination strategies?

Similar to how the selection strategy is specified, developers can specify a termination strategy or use one of the predefined strategies. Termination strategies can also define a maximum number of iterations a conversation should be limited to.

Termination strategies can be created using a prompt, such as:

`````
prompt="""
    Determine if the copy has been approved.  If so, respond with a single word: yes

    History:
    {{$history}}
    """
`````
You can also specify which agent should determine that termination, which in our case would be ReviewingDirectorAgent. The agents to determine termination are also defined in the AgentGroupChat.

### Conversation state

Whether you use AgentGroupChat for a single-turn or multi-turn conversation, the state is updated to completed once it meets the termination criteria. However, you may want to use the group chat instance again. To keep using the same chat instance, you'll need to reset the completion state to False. Without a state reset, the AgentGroupChat can't accept new interactions.

When a conversation hits the maximum number of iterations allowed, the conversation will end but won't be marked as completed. In this case, you can extend the conversation without resetting the conversation state.

By understanding these components, you can better utilize the Semantic Kernel Agent Framework to build intelligent multi-agent systems.