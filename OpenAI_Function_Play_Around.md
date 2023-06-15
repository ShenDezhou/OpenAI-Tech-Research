# OpenAI LLM Function

OpenAI开放了一个新功能，在langchain0.0.199版本下，可以试用很多功能。

# 工具

langchain自带了如下几种工具：

    "AIPluginTool",
    "APIOperation",
    "AzureCogsFormRecognizerTool",
    "AzureCogsImageAnalysisTool",
    "AzureCogsSpeech2TextTool",
    "AzureCogsText2SpeechTool",
    "BaseTool",
    "BaseTool",
    "BaseTool",
    "BingSearchResults",
    "BingSearchRun",
    "ClickTool",
    "CopyFileTool",
    "CurrentWebPageTool",
    "DeleteFileTool",
    "DuckDuckGoSearchResults",
    "DuckDuckGoSearchRun",
    "ExtractHyperlinksTool",
    "ExtractTextTool",
    "FileSearchTool",
    "GetElementsTool",
    "SteamshipImageGenerationTool",
    "GmailCreateDraft",
    "GmailGetMessage",
    "GmailGetThread",
    "GmailSearch",
    "GmailSendMessage",
    "GooglePlacesTool",
    "GoogleSearchResults",
    "GoogleSearchRun",
    "GoogleSerperResults",
    "GoogleSerperRun",
    "HumanInputRun",
    "IFTTTWebhook",
    "InfoPowerBITool",
    "ListDirectoryTool",
    "ListPowerBITool",
    "MetaphorSearchResults",
    "MoveFileTool",
    "NavigateBackTool",
    "NavigateTool",
    "OpenAPISpec",
    "OpenWeatherMapQueryRun",
    "QueryPowerBITool",
    "ReadFileTool",
    "SceneXplainTool",
    "ShellTool",
    "StructuredTool",
    "Tool",
    "VectorStoreQATool",
    "VectorStoreQAWithSourcesTool",
    "WikipediaQueryRun",
    "WolframAlphaQueryRun",
    "WriteFileTool",
    "ZapierNLAListActions",
    "ZapierNLARunAction",
    "tool",
    "YouTubeSearchTool",
    "BraveSearch",
    "PubmedQueryRun",
    "format_tool_to_openai_function",

Tool可以用名字加载，示例如下：
```python
from langchain.agents import load_tools
tool_names = [...]
tools = load_tools(tool_names)
```

其中本文待尝试的有：ShellTool、PythonREPLTool

## ShellTool尝试

```python
from langchain.chat_models import ChatOpenAI
from langchain.schema import HumanMessage

model = ChatOpenAI(model="gpt-3.5-turbo-0613")
from langchain.tools import ShellTool, format_tool_to_openai_function

tools = [ShellTool()]
functions = [format_tool_to_openai_function(t) for t in tools]
message = model.predict_messages([HumanMessage(content='count how many lines in a.txt?')], functions=functions)

print(message.additional_kwargs['function_call'])

```

## Python_RwriteEvalPrintLoop

```python
from langchain.chat_models import ChatOpenAI
from langchain.schema import HumanMessage
from langchain.agents import Tool
from langchain.utilities import PythonREPL
python_repl = PythonREPL()
# You can create the tool to pass to an agent
repl_tool = Tool(
    name="python_repl",
    description="A Python shell. Use this to execute python commands. Input should be a valid python command. If you want to see the output of a value, you should print it out with `print(...)`.",
    func=python_repl.run
)

model = ChatOpenAI(model="gpt-3.5-turbo-0613")
from langchain.tools import format_tool_to_openai_function

tools = [repl_tool]
functions = [format_tool_to_openai_function(t) for t in tools]
message = model.predict_messages([HumanMessage(content='Write a python code to eval 1 plus 1.')], functions=functions)

print(message.additional_kwargs['function_call'])
```

