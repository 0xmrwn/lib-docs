# How to add message history

Prerequisites

This guide assumes familiarity with the following concepts:

- [Chaining runnables](https://python.langchain.com/docs/how_to/sequence/)
- [Prompt templates](https://python.langchain.com/docs/concepts/prompt_templates/)
- [Chat Messages](https://python.langchain.com/docs/concepts/messages/)
- [LangGraph persistence](https://langchain-ai.github.io/langgraph/how-tos/persistence/)

note

This guide previously covered the [RunnableWithMessageHistory](https://python.langchain.com/api_reference/core/runnables/langchain_core.runnables.history.RunnableWithMessageHistory.html) abstraction. You can access this version of the guide in the [v0.2 docs](https://python.langchain.com/v0.2/docs/how_to/message_history/).

As of the v0.3 release of LangChain, we recommend that LangChain users take advantage of [LangGraph persistence](https://langchain-ai.github.io/langgraph/concepts/persistence/) to incorporate `memory` into new LangChain applications.

If your code is already relying on `RunnableWithMessageHistory` or `BaseChatMessageHistory`, you do **not** need to make any changes. We do not plan on deprecating this functionality in the near future as it works for simple chat applications and any code that uses `RunnableWithMessageHistory` will continue to work as expected.

Please see [How to migrate to LangGraph Memory](https://python.langchain.com/docs/versions/migrating_memory/) for more details.

Passing conversation state into and out a chain is vital when building a chatbot. LangGraph implements a built-in persistence layer, allowing chain states to be automatically persisted in memory, or external backends such as SQLite, Postgres or Redis. Details can be found in the LangGraph [persistence documentation](https://langchain-ai.github.io/langgraph/how-tos/persistence/).

In this guide we demonstrate how to add persistence to arbitrary LangChain runnables by wrapping them in a minimal LangGraph application. This lets us persist the message history and other elements of the chain's state, simplifying the development of multi-turn applications. It also supports multiple threads, enabling a single application to interact separately with multiple users.

## Setup [​](https://python.langchain.com/docs/how_to/message_history/\#setup "Direct link to Setup")

Let's initialize a chat model:

Select [chat model](https://python.langchain.com/docs/integrations/chat/):

Google Gemini▾

[OpenAI](https://python.langchain.com/docs/how_to/message_history/#)
[Anthropic](https://python.langchain.com/docs/how_to/message_history/#)
[Azure](https://python.langchain.com/docs/how_to/message_history/#)
[Google Gemini](https://python.langchain.com/docs/how_to/message_history/#)
[Google Vertex](https://python.langchain.com/docs/how_to/message_history/#)
[AWS](https://python.langchain.com/docs/how_to/message_history/#)
[Groq](https://python.langchain.com/docs/how_to/message_history/#)
[Cohere](https://python.langchain.com/docs/how_to/message_history/#)
[NVIDIA](https://python.langchain.com/docs/how_to/message_history/#)
[Fireworks AI](https://python.langchain.com/docs/how_to/message_history/#)
[Mistral AI](https://python.langchain.com/docs/how_to/message_history/#)
[Together AI](https://python.langchain.com/docs/how_to/message_history/#)
[IBM watsonx](https://python.langchain.com/docs/how_to/message_history/#)
[Databricks](https://python.langchain.com/docs/how_to/message_history/#)
[xAI](https://python.langchain.com/docs/how_to/message_history/#)
[Perplexity](https://python.langchain.com/docs/how_to/message_history/#)

```codeBlockLines_e6Vv
pip install -qU "langchain[google-genai]"

```

```codeBlockLines_e6Vv
import getpass
import os

if not os.environ.get("GOOGLE_API_KEY"):
  os.environ["GOOGLE_API_KEY"] = getpass.getpass("Enter API key for Google Gemini: ")

from langchain.chat_models import init_chat_model

llm = init_chat_model("gemini-2.0-flash", model_provider="google_genai")

```

## Example: message inputs [​](https://python.langchain.com/docs/how_to/message_history/\#example-message-inputs "Direct link to Example: message inputs")

Adding memory to a [chat model](https://python.langchain.com/docs/concepts/chat_models/) provides a simple example. Chat models accept a list of messages as input and output a message. LangGraph includes a built-in `MessagesState` that we can use for this purpose.

Below, we:

1. Define the graph state to be a list of messages;
2. Add a single node to the graph that calls a chat model;
3. Compile the graph with an in-memory checkpointer to store messages between runs.

info

The output of a LangGraph application is its [state](https://langchain-ai.github.io/langgraph/concepts/low_level/). This can be any Python type, but in this context it will typically be a `TypedDict` that matches the schema of your runnable.

```codeBlockLines_e6Vv
from langchain_core.messages import HumanMessage
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import START, MessagesState, StateGraph

# Define a new graph
workflow = StateGraph(state_schema=MessagesState)

# Define the function that calls the model
def call_model(state: MessagesState):
    response = llm.invoke(state["messages"])
    # Update message history with response:
    return {"messages": response}

# Define the (single) node in the graph
workflow.add_edge(START, "model")
workflow.add_node("model", call_model)

# Add memory
memory = MemorySaver()
app = workflow.compile(checkpointer=memory)

```

**API Reference:** [HumanMessage](https://python.langchain.com/api_reference/core/messages/langchain_core.messages.human.HumanMessage.html) \| [MemorySaver](https://langchain-ai.github.io/langgraph/reference/checkpoints/#langgraph.checkpoint.memory.MemorySaver) \| [StateGraph](https://langchain-ai.github.io/langgraph/reference/graphs/#langgraph.graph.state.StateGraph)

When we run the application, we pass in a configuration `dict` that specifies a `thread_id`. This ID is used to distinguish conversational threads (e.g., between different users).

```codeBlockLines_e6Vv
config = {"configurable": {"thread_id": "abc123"}}

```

We can then invoke the application:

```codeBlockLines_e6Vv
query = "Hi! I'm Bob."

input_messages = [HumanMessage(query)]
output = app.invoke({"messages": input_messages}, config)
output["messages"][-1].pretty_print()  # output contains all messages in state

```

```codeBlockLines_e6Vv
==================================[1m Ai Message [0m==================================\
\
It's nice to meet you, Bob! I'm Claude, an AI assistant created by Anthropic. How can I help you today?\
\
```\
\
```codeBlockLines_e6Vv\
query = "What's my name?"\
\
input_messages = [HumanMessage(query)]\
output = app.invoke({"messages": input_messages}, config)\
output["messages"][-1].pretty_print()\
\
```\
\
```codeBlockLines_e6Vv\
==================================[1m Ai Message [0m==================================\
\
Your name is Bob, as you introduced yourself at the beginning of our conversation.\
\
```\
\
Note that states are separated for different threads. If we issue the same query to a thread with a new `thread_id`, the model indicates that it does not know the answer:\
\
```codeBlockLines_e6Vv\
query = "What's my name?"\
config = {"configurable": {"thread_id": "abc234"}}\
\
input_messages = [HumanMessage(query)]\
output = app.invoke({"messages": input_messages}, config)\
output["messages"][-1].pretty_print()\
\
```\
\
```codeBlockLines_e6Vv\
==================================[1m Ai Message [0m==================================\
\
I'm afraid I don't actually know your name. As an AI assistant, I don't have personal information about you unless you provide it to me directly.\
\
```\
\
## Example: dictionary inputs [​](https://python.langchain.com/docs/how_to/message_history/\#example-dictionary-inputs "Direct link to Example: dictionary inputs")\
\
LangChain runnables often accept multiple inputs via separate keys in a single `dict` argument. A common example is a prompt template with multiple parameters.\
\
Whereas before our runnable was a chat model, here we chain together a prompt template and chat model.\
\
```codeBlockLines_e6Vv\
from langchain_core.prompts import ChatPromptTemplate, MessagesPlaceholder\
\
prompt = ChatPromptTemplate.from_messages(\
    [\
        ("system", "Answer in {language}."),\
        MessagesPlaceholder(variable_name="messages"),\
    ]\
)\
\
runnable = prompt | llm\
\
```\
\
**API Reference:** [ChatPromptTemplate](https://python.langchain.com/api_reference/core/prompts/langchain_core.prompts.chat.ChatPromptTemplate.html) \| [MessagesPlaceholder](https://python.langchain.com/api_reference/core/prompts/langchain_core.prompts.chat.MessagesPlaceholder.html)\
\
For this scenario, we define the graph state to include these parameters (in addition to the message history). We then define a single-node graph in the same way as before.\
\
Note that in the below state:\
\
- Updates to the `messages` list will append messages;\
- Updates to the `language` string will overwrite the string.\
\
```codeBlockLines_e6Vv\
from typing import Sequence\
\
from langchain_core.messages import BaseMessage\
from langgraph.graph.message import add_messages\
from typing_extensions import Annotated, TypedDict\
\
class State(TypedDict):\
    messages: Annotated[Sequence[BaseMessage], add_messages]\
    language: str\
\
workflow = StateGraph(state_schema=State)\
\
def call_model(state: State):\
    response = runnable.invoke(state)\
    # Update message history with response:\
    return {"messages": [response]}\
\
workflow.add_edge(START, "model")\
workflow.add_node("model", call_model)\
\
memory = MemorySaver()\
app = workflow.compile(checkpointer=memory)\
\
```\
\
**API Reference:** [BaseMessage](https://python.langchain.com/api_reference/core/messages/langchain_core.messages.base.BaseMessage.html) \| [add\_messages](https://langchain-ai.github.io/langgraph/reference/graphs/#langgraph.graph.message.add_messages)\
\
```codeBlockLines_e6Vv\
config = {"configurable": {"thread_id": "abc345"}}\
\
input_dict = {\
    "messages": [HumanMessage("Hi, I'm Bob.")],\
    "language": "Spanish",\
}\
output = app.invoke(input_dict, config)\
output["messages"][-1].pretty_print()\
\
```\
\
```codeBlockLines_e6Vv\
==================================[1m Ai Message [0m==================================\
\
¡Hola, Bob! Es un placer conocerte.\
\
```\
\
## Managing message history [​](https://python.langchain.com/docs/how_to/message_history/\#managing-message-history "Direct link to Managing message history")\
\
The message history (and other elements of the application state) can be accessed via `.get_state`:\
\
```codeBlockLines_e6Vv\
state = app.get_state(config).values\
\
print(f'Language: {state["language"]}')\
for message in state["messages"]:\
    message.pretty_print()\
\
```\
\
```codeBlockLines_e6Vv\
Language: Spanish\
================================[1m Human Message [0m=================================\
\
Hi, I'm Bob.\
==================================[1m Ai Message [0m==================================\
\
¡Hola, Bob! Es un placer conocerte.\
\
```\
\
We can also update the state via `.update_state`. For example, we can manually append a new message:\
\
```codeBlockLines_e6Vv\
from langchain_core.messages import HumanMessage\
\
_ = app.update_state(config, {"messages": [HumanMessage("Test")]})\
\
```\
\
**API Reference:** [HumanMessage](https://python.langchain.com/api_reference/core/messages/langchain_core.messages.human.HumanMessage.html)\
\
```codeBlockLines_e6Vv\
state = app.get_state(config).values\
\
print(f'Language: {state["language"]}')\
for message in state["messages"]:\
    message.pretty_print()\
\
```\
\
```codeBlockLines_e6Vv\
Language: Spanish\
================================[1m Human Message [0m=================================\
\
Hi, I'm Bob.\
==================================[1m Ai Message [0m==================================\
\
¡Hola, Bob! Es un placer conocerte.\
================================[1m Human Message [0m=================================\
\
Test\
\
```\
\
For details on managing state, including deleting messages, see the LangGraph documentation:\
\
- [How to delete messages](https://langchain-ai.github.io/langgraph/how-tos/memory/delete-messages/)\
- [How to view and update past graph state](https://langchain-ai.github.io/langgraph/how-tos/human_in_the_loop/time-travel/)\
\
- [Setup](https://python.langchain.com/docs/how_to/message_history/#setup)\
- [Example: message inputs](https://python.langchain.com/docs/how_to/message_history/#example-message-inputs)\
- [Example: dictionary inputs](https://python.langchain.com/docs/how_to/message_history/#example-dictionary-inputs)\
- [Managing message history](https://python.langchain.com/docs/how_to/message_history/#managing-message-history)