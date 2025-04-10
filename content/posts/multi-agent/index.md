+++
date = '2025-04-10T09:15:23+01:00'
draft = false
title = 'Semantic Kernel and Multi-Agent AI Apps'
categories = ['AI']
tags = ['Agents', 'Foundry', 'Azure', 'Semantic Kernel', 'Multi-Agent']
+++

# Intro to Semantic Kernel and Multi-Agent AI Apps

In my last post, I have covered Azure [AI Agent Service](https://beyondelastic.github.io/posts/agent/) and how it can be used to easily build and run AI agents on Azure. This time, we are going to look at Semantic Kernel as a framework to build, orchestrate and deploy AI agents or multi-agent applications. Semantic Kernel is an open-source development kit by Microsoft that offers a unified framework with a plugin-based architecture for easier integration and reduced complexity. It serves as efficient middleware, enabling fast development of enterprise-grade solutions by combining prompts with existing APIs.

The Semantic Kernel SDK is available for C#, Python and Java. More details can be found on the [official GitHub repository](https://github.com/microsoft/semantic-kernel) and the [official Microsoft documentation](https://learn.microsoft.com/en-us/semantic-kernel/overview/).

In this blog post, we are going to combine the Azure AI Agent Service with Semantic Kernel to build a multi-agent AI application with group chat functionality. 

## Background

If you are new to the topic, it can be a bit confusing as there are many options to choose from. Which framework should I use? AutoGen or Semantic Kernel? Which API should I use the Chat Completions API, the Assistants API or the Azure AI Agent Service? Well, famous last words "it depends" and "things are evolving fast". 

The framework discussion is manly driven by whether you need enterprise-grade support or not. If that is a yes, you should look towards Semantic Kernel. If you are still in the ideation/testing phase, and you need the latest and greatest functionality, take a look at AutoGen. Both teams are working on strategic convergence and integrations between both frameworks, as you can read in the following blog posts:

- [Microsoft’s Agentic AI Frameworks: AutoGen and Semantic Kernel](https://devblogs.microsoft.com/semantic-kernel/microsofts-agentic-ai-frameworks-autogen-and-semantic-kernel/)
- [Semantic Kernel Roadmap H1 2025: Accelerating Agents, Processes, and Integration](https://devblogs.microsoft.com/semantic-kernel/semantic-kernel-roadmap-h1-2025-accelerating-agents-processes-and-integration/)
- [AutoGen and Semantic Kernel, Part 2](https://devblogs.microsoft.com/semantic-kernel/semantic-kernel-and-autogen-part-2/)

In terms of API, the Chat Completions API is lightweight and stateless and can be a good fit for simple tasks. The Assistants API is stateful (managing conversation history) and can be a good fit for more complex scenarios. Azure AI Agent Service delivers all the functionality of the Assistants API plus flexible model choice, out of the box tools, tracing and more. 

## Requirements

As an execution engine, we will use the Azure AI Agent Service. We are not going to cover the infrastructure requirements in this blog post. Nevertheless, if you want to get started quickly, simply deploy this [bicep template](https://github.com/Azure-Samples/azureai-samples/tree/main/scenarios/Agents/setup/standard-agent) for an standard Azure AI Agent deployment. 

### Prepare local dev envrionment

For our local development environment, we need to install the following packages:
```sh
pip install python-dotenv azure-identity semantic-kernel[azure]  
```

Next, we will use a local .env file to specify some variables to connect to our Azure AI Foundry Project. 

````
AZURE_AI_AGENT_PROJECT_CONNECTION_STRING="your_project_connection_string"
AZURE_AI_AGENT_MODEL_DEPLOYMENT_NAME="your_model_deployment"
````

## Coding

My goal is to create multiple AI agents that act as Basketball coaches. A Head Coach and an Assistant Coach that will exchange ideas and come up with a game plan for a specific game situation. 

For this blog, we will keep it very simple and don't play too much with plugins or extensions. We just want to create a group chat with specialized agents. 

### Building the mulit-agent app

As always we need to add some references first. This time we are going to import the asyncio model as we need to execute the main function asynchronously. This allows the program to perform non-blocking operations, such as interacting with Azure AI services, creating agents, and managing the group chat. Additionally, we are going to import some components from the Semantic Kernel library. More about these Semantic Kernel classes can be found later in the text. 

```python
# add references
import asyncio
from dotenv import load_dotenv
from azure.identity.aio import DefaultAzureCredential
from semantic_kernel.agents import AzureAIAgent, AzureAIAgentSettings, AgentGroupChat
from semantic_kernel.agents.strategies import TerminationStrategy, SequentialSelectionStrategy
from semantic_kernel.contents.utils.author_role import AuthorRole
```

Now we are going to load the environment variables from the .env file and define the agent names and instructions as well as the task they should work on. 

```python
# get configuration settings
load_dotenv()

# agent instructions
HEAD_COACH = "HeadCoach"
HEAD_COACH_INSTRUCTIONS = """
You are a Basketball Head Coach that knows everything about offensive plays and strategies. 
You respond to specific game situations with advice on how to change the game plan. You can ask for more information about the game situation if needed. 
For offensive plays and strategies, you will decide on the strategy yourself. If the game situation demands a change for defensive plays and strategies, you will ask your Assistant Coach for advice.
You will use the advice given by the Assistant Coach regarding defensive adjustments and your own decision for offensive adjustments to create the final game plan.

RULES:
- Use the instructions provided.
- Prepend your response with this text: "head_coach > "
- Do not directly answer the question if it is related to defensive strategies. Instead, ask your Assistant Coach for advice.
- Do not use the words "final game plan" unless you have created a final game plan according to the instructions.
- Add "final game plan" to the end of your response if you have created a final game plan according to the instructions.
"""

ASSISTANT_COACH = "AssistantCoach"
ASSISTANT_COACH_INSTRUCTIONS = """
You are a Basketball Assistant Coach that knows defensive plays and strategies.
You give advice to your Head Coach for specific game situations that require defensive adjustment.

RULES:
- Use the instructions provided.
- Prepend your response with this text: "assistant_coach > "
- You are not allowed to give advice on offensive plays and strategies.
- You don't decide the final game strategy and plan, you only give advice to the Head Coach.
- Your advice should be clear and concise and should not include any unnecessary information.
"""

# agent task
TASK = "Could you please give me advice on how to change the game strategy for the next quarter? We are playing zone defense, and the other team just scored 10 points in a row. We need to change our strategy to stop them. What should we do?"
```

So far, so good. We will now start with our main function. We retrieve the configuration settings with the AzureAIAgentSettings.create method and use the DefaultAzureCredential class to authenticate against our Azure Services, and lastly, we create a client for interacting with the Azure AI agent service.

```python
async def main():

    ai_agent_settings = AzureAIAgentSettings.create()

    async with (
        DefaultAzureCredential(exclude_environment_credential=True, 
            exclude_managed_identity_credential=True) as creds,
        AzureAIAgent.create_client(credential=creds) as client,
    ):
```

The next step is to create our agents on the Azure AI Agent Service and wrap them into Semantic Kernel agents using the AzureAIAgent class.

```python
        # create the head-coach agent on the Azure AI agent service
        headcoach_agent_definition = await client.agents.create_agent(
            model=ai_agent_settings.model_deployment_name,
            name=HEAD_COACH,
            instructions=HEAD_COACH_INSTRUCTIONS,
        )
        
        # create a Semantic Kernel agent for the Azure AI head-coach agen
        agent_headcoach = AzureAIAgent(
            client=client,
            definition=headcoach_agent_definition,
        )
        
        # create the assistant coach agent on the Azure AI agent service
        assistantcoach_agent_definition = await client.agents.create_agent(
            model=ai_agent_settings.model_deployment_name,
            name=ASSISTANT_COACH,
            instructions=ASSISTANT_COACH_INSTRUCTIONS,
        )

        # create a Semantic Kernel agent for the assistant coach Azure AI agent
        agent_assistantcoach = AzureAIAgent(
            client=client,
            definition=assistantcoach_agent_definition,
        )
```

This is where the fun part begins. We are initializing a group chat via the AgentGroupChat class and adding our agents to it. We need to define who comes next and when the group chat should end. To do so, we define a termination and selection strategy. Additionally, we define which agent is contributing to the termination strategy. In our case, only the Head Coach is in charge. Furthermore, we are defining a maximum of 4 iterations until the chat will be terminated.  

```python
        # add the agents to a group chat with a custom termination and selection strategy
        chat = AgentGroupChat(
            agents=[agent_headcoach, agent_assistantcoach],
            termination_strategy=ApprovalTerminationStrategy(
                agents=[agent_headcoach], 
                maximum_iterations=4, 
                automatic_reset=True
            ),
            selection_strategy=SelectionStrategy(agents=[agent_headcoach,agent_assistantcoach]),      
        )
```

In this section, we handle the execution of the group chat, including adding the task, invoking the chat, and performing cleanup operations. The AuthorRole.USER constant is used to explicitly identify the role of the message sender as the user to ensure clarity in the conversation flow. 

```python
        try:
            # add the task as a message to the group chat
            await chat.add_chat_message(message=TASK)
            print(f"# {AuthorRole.USER}: '{TASK}'")
            # invoke the chat
            async for content in chat.invoke():
                print(f"# {content.role} - {content.name or '*'}: '{content.content}'")
        finally:
            # cleanup and delete the agents
            print("--chat ended--")
            await chat.reset()
            await client.agents.delete_agent(agent_headcoach.id)
            await client.agents.delete_agent(agent_assistantcoach.id)
```

Almost at the end, we just need to add two classes for the termination and selection function that we have used in the group chat definition. As we defined in the Head Coach agent instructions, as soon as the final game plan is ready, it should add "final game plan" to its message. We are checking if the last message in the history contains this phrase, and we will terminate the group chat. 

```python
# class of termination strategy
class ApprovalTerminationStrategy(TerminationStrategy):
    """A strategy for determining when an agent should terminate."""

    async def should_agent_terminate(self, agent, history):
        """Check if the agent should terminate."""
        return "final game plan" in history[-1].content.lower()
```

The second class adds a selection function that defines which agent should take the next turn in the chat. If the last message comes from the User or the Assistant Coach, it is the Head Coaches turn. 

```python
# class for selection strategy
class SelectionStrategy(SequentialSelectionStrategy):
    """A strategy for determining which agent should take the next turn in the chat."""
    
    # select the next agent that should take the next turn in the chat
    async def select_agent(self, agents, history):
        """"Check which agent should take the next turn in the chat."""

        # the Head Coach should go after the User or the Assistant Coach
        if (history[-1].name == ASSISTANT_COACH or history[-1].role == AuthorRole.USER):
            agent_name = HEAD_COACH
            return next((agent for agent in agents if agent.name == agent_name), None)
        
        # otherwise it is the Assistant Coach's turn
        return next((agent for agent in agents if agent.name == ASSISTANT_COACH), None)
```

Lastly, we define the entry point for our app, and that it is executed as an asynchronous coroutine.

```python
if __name__ == "__main__":
    asyncio.run(main())
```

### Running the mulit-agent app

Ok, we should have something to play and test with. The finale code can be found on my GitHub repo [here](https://github.com/beyondelastic/basketball-ai-agent/tree/main). 

Let's see if we can get a proper game plan from our coaching staff. 

```text
➜ python app.py
# AuthorRole.USER: 'Could you please give me advice on how to change the game strategy for the next quarter? We are playing zone defense and the other team just scored 10 points in a row. We need to change our strategy to stop them. What should we do?'
# AuthorRole.ASSISTANT - HeadCoach: 'head_coach > I'll need to consult with the Assistant Coach about defensive adjustments since that's not my area of expertise. Assistant Coach, what adjustments do you recommend for our zone defense to stop the opposing team who has just scored 10 points in a row? 

In terms of our offensive strategy, we should focus on enhancing our ball movement and executing quick passes to exploit the gaps in their defense. Let's emphasize perimeter shooting and look for opportunities to drive to the basket, ensuring we spread the floor to create space. 

Please provide your defensive advice, and I'll integrate that with our offensive strategy for the necessary adjustments.'
# AuthorRole.ASSISTANT - AssistantCoach: 'assistant_coach > Consider switching to a man-to-man defense to apply more pressure on their shooters and disrupt their rhythm. This will help limit their easy scoring opportunities and force them into more contested shots. Ensure our players communicate effectively and switch on screens. Additionally, encourage tighter closeouts on shooters to contest their shots and deny open looks. If they continue to score, we could also implement a trap to force turnovers and get out in transition.'
# AuthorRole.ASSISTANT - HeadCoach: 'head_coach > Thank you, Assistant Coach. Based on your advice, we'll switch to a man-to-man defense to apply pressure and limit their scoring opportunities. We'll focus on strong communication and switching on screens, as well as tighter closeouts on shooters.

On the offensive side, we'll continue to enhance our ball movement, emphasizing quick passes and prioritizing perimeter shooting, alongside drive opportunities. This blend of a more aggressive defensive approach and a fluid offensive strategy should help us regain control of the game.

Now, let's put this all together: we'll implement a man-to-man defense while enhancing our offensive ball movement and exploiting gaps in their defense. 

final game plan'
--chat ended--
```

Nice! Thanks Coaching staff! That indeed sounds like a plan to win the game in the end. 

In Azure AI Foundry, we can see that the corresponding Azure AI Agents are getting created during the runtime and cleaned up afterward. 

![Agents](/agents.jpg)

## Summary

This was a very simple example, but it shows how multiple agents can have different expertise and exchange ideas or knowledge about a specific topic via the Semantic Kernel group chat. Additionally, we can facilitate a structured conversation flow that allows the agents to efficiently collaborate and work on user provided tasks. Imagine that these agents would have access to different tools or knowledge sources to make them specialists for a specific task. We already looked at how to add tools (Code Interpreter Tool) to Azure AI Agents in my last blog [post](https://beyondelastic.github.io/posts/agent/). The same approach can be used in combination with Semantic Kernel to make our agents even smarter. 

## Sources

- [Semantic Kernel Microsoft documentation](https://learn.microsoft.com/en-us/semantic-kernel/overview/)
- [Semantic Kernel GitHub repository](https://github.com/microsoft/semantic-kernel)
- [Microsoft’s Agentic AI Frameworks: AutoGen and Semantic Kernel](https://devblogs.microsoft.com/semantic-kernel/microsofts-agentic-ai-frameworks-autogen-and-semantic-kernel/)
- [Semantic Kernel Roadmap H1 2025: Accelerating Agents, Processes, and Integration](https://devblogs.microsoft.com/semantic-kernel/semantic-kernel-roadmap-h1-2025-accelerating-agents-processes-and-integration/)
- [AutoGen and Semantic Kernel, Part 2](https://devblogs.microsoft.com/semantic-kernel/semantic-kernel-and-autogen-part-2/)
- [Azure AI Agent standard setup bicep template](https://github.com/Azure-Samples/azureai-samples/tree/main/scenarios/Agents/setup/standard-agent) 
- [Intro to Azure AI Agent Service](https://beyondelastic.github.io/posts/agent/)
- [Microsoft Learn AI Agent Fundamentals](https://learn.microsoft.com/en-us/training/modules/ai-agent-fundamentals/)
- [Semantic Kernel Agents are now Generally Available](https://devblogs.microsoft.com/semantic-kernel/semantic-kernel-agents-are-now-generally-available/)