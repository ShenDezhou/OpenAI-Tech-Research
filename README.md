# GPT-3.5和GPT-4的新特性：函数调用

OpenAI 为其 ChatGPT(GPT-3.5-Turbo) 和 GPT-4 发布了一项新功能“函数调用”。

## 函数调用

在API调用中，您可以描述 `gpt-3.5-turbo-0613` 和 `gpt-4-0613` 的函数，并让模型智能地选择输出包含调用这些函数的参数的JSON对象。聊天完成API不会调用该函数；相反，模型会生成JSON，您可以在代码中使用该JSON来调用函数。

最新的模型（gpt-3.5-turbo-0613和gpt-4-0613）已经被微调，以便检测何时应该调用函数（取决于输入），并以符合函数签名的JSON响应。但是，这种能力也带来了潜在的风险。我们强烈建议在代表用户采取影响世界的行动之前构建用户确认流程（发送电子邮件、发布内容在线、进行购买等）。

在内部，函数以模型已经训练过的语法注入到系统消息中。这意味着函数会影响模型的上下文限制，并作为输入令牌计费。如果遇到上下文限制，我们建议限制函数数量或提供函数参数文档的长度。

函数调用允许您更可靠地从模型中获取结构化数据。例如，您可以：

- 创建聊天机器人，通过调用外部API来回答问题（例如ChatGPT插件）
例如，定义函数`send_email(to: string, body: string)`或`get_current_weather(location: string, unit: 'celsius' | 'fahrenheit')`

- 将自然语言转换为API调用
例如，将“谁是我的头号客户？”转换为`get_customers(min_revenue: int, created_before: string, limit: int)`，并调用您的内部API

- 从文本中提取结构化数据
例如，定义一个名为`extract_data(name: string, birthday: string)`或`sql_query(query: string)`的函数
...等等！

函数调用的基本步骤如下：

1. 使用用户查询和在函数参数中定义的一组函数调用模型。
2. 模型可以选择调用函数；如果是这样，内容将是符合您的自定义模式的字符串化JSON对象（注意：模型可能会生成无效的JSON或产生参数幻觉）。
3. 在您的代码中将字符串解析为JSON，并使用提供的参数调用您的函数（如果存在）。
4. 通过将函数响应附加为新消息再次调用模型，并让模型向用户总结结果。

您可以通过以下示例了解这些步骤的实现：

```python
import openai
import json

# Example dummy function hard coded to return the same weather
# In production, this could be your backend API or an external API
def get_current_weather(location, unit="fahrenheit"):
    """Get the current weather in a given location"""
    weather_info = {
        "location": location,
        "temperature": "72",
        "unit": unit,
        "forecast": ["sunny", "windy"],
    }
    return json.dumps(weather_info)

# Step 1, send model the user query and what functions it has access to
def run_conversation():
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo-0613",
        messages=[{"role": "user", "content": "What's the weather like in Boston?"}],
        functions=[
            {
                "name": "get_current_weather",
                "description": "Get the current weather in a given location",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "location": {
                            "type": "string",
                            "description": "The city and state, e.g. San Francisco, CA",
                        },
                        "unit": {"type": "string", "enum": ["celsius", "fahrenheit"]},
                    },
                    "required": ["location"],
                },
            }
        ],
        function_call="auto",
    )

    message = response["choices"][0]["message"]

    # Step 2, check if the model wants to call a function
    if message.get("function_call"):
        function_name = message["function_call"]["name"]

        # Step 3, call the function
        # Note: the JSON response from the model may not be valid JSON
        function_response = get_current_weather(
            location=message.get("location"),
            unit=message.get("unit"),
        )

        # Step 4, send model the info on the function call and function response
        second_response = openai.ChatCompletion.create(
            model="gpt-3.5-turbo-0613",
            messages=[
                {"role": "user", "content": "What is the weather like in boston?"},
                message,
                {
                    "role": "function",
                    "name": function_name,
                    "content": function_response,
                },
            ],
        )
        return second_response

print(run_conversation())
```

在上面的示例中，我们将函数响应发送回模型并让它决定下一步。

它以一条面向用户的消息进行响应，该消息告诉用户波士顿的温度，但根据查询，它可能会选择再次调用一个函数。

例如，如果你问模型“查找本周末波士顿的天气，预订周六的两人晚餐，更新我的日历”并为这些查询提供相应的功能，它可能会选择背对背地调用它们，并且只在 最后创建一条面向用户的消息。

如果你想强制模型生成面向用户的消息，你可以通过设置 function_call: "none" 来实现（如果没有提供函数，则不会调用任何函数）。

您可以在 OpenAI 说明书中找到更多函数调用示例：

[How_to_call_functions_with_chat_models.ipynb](colab%2FHow_to_call_functions_with_chat_models.ipynb)

