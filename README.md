# Langchain-agent-study
Langchain-Agent学习，Langchain长短期记忆
## 模型结构化输出
 1. pydantic 最好用，体验最好，有数据验证，适配复杂数据类型
 2. 变量名:类型 = Field(Description = " ")
 3. TypeDict最轻量化，效果一般
 4. JsonSchema 跨语言，很灵活，效果也不错
## 模型工具调用
### 多工具调用 （查询某企业的股价与新闻）
1. 准备： 股价和新闻查询工具，大模型
2. 步骤：
   - 绑定LLM与工具 得到 LLM_T
   - 将 HumenMessage == 腾讯的股价+新闻 送入 LLM_T 构造 AIMessage,其中含有 tool_calls
   - 从 AIMessage 中拿出 tool_calls 来 执行工具 得到 ToolMessage，（目前做的是同步执行工具，异步还没做到）
   - 将所有的Message 组合 ，送入 LLM_T 得到最终的结果。
## 模型调用配置
- 在模型运行时可以让其动态的执行一些配置，比如模型调用基本参数 model,temperature。或者是 自定义回调函数
- 自定义回调函数，该函数可以在模型返回结果前后执行我们想做的事情，
  - 比如模型使用前 将信息记录到日志系统或数据库（模型配置信息持久化）
  - 比如模型使用前 根据 metadata 中的 user_id 进行用户行为分析（用户在LLM使用次数与时间的关系）
  - 比如模型使用后  可以在这里记录运行结束的信息，比如消耗的令牌数等存储
 ### 提示词
 - 动态提示词
   - 在 create_agent时通过 context_schema参数告知Agent你所定义的上下文结构，最后在调用 agent.invoke时，通过 context 参数传入符合该结构的实际数据：
   - 可以通过传入的 context 内容结合 设定的动态提示词中间件 来为不同用户设置不同的System_prompt，精准个性化服务
 ## 工具创建
 ### pydantic创建工具
-这种方式定义工具，通过 @tool(args_schema=PydanticModelCls)将这个 Pydantic 模型与工具函数关联。
- PydanticModelCls 需要继承自 BaseModel的类，使用类型提示（如 str, int）和 Field函数来声明每个字段的名称、类型、默认值和描述。每个字段的 description参数至关重要，它直接影响大模型理解参数含义的能力。
-  可以利用 Pydantic 的类型系统进行参数验证，当大模型需要调用工具前，Pydantic 会自动验证参数的类型和有效性。
-   在 convert_ticket_id_to_upper方法中的 cls，代表的是 这个 Pydantic 模型类本身，在这里也就是 TicketQueryInput这个类。@field_validator装饰器将对应方法标记为类方法，类方法的第一个参数约定俗成地命名为 cls，它指向类而不是类的实例。这样，在验证逻辑中如果需要访问类的其他属性或方法，就可以通过 cls来操作。
- json.dump()是 Python 标准库 json模块中的函数，用于将 Python 对象（如字典、列表）序列化成一个 JSON 格式的字符串。
 - ensure_ascii=False: 这个参数默认值为 True，表示所有非 ASCII 字符（如中文）会被转换成 \uXXXX形式的转义序列。设置为 False后，中文、表情符号等字符就能在 JSON 字符串中正常显示，而不是一堆乱码。
 - indent=2: 这个参数用于美化输出，使生成的 JSON 字符串具有可读性。数字 2表示使用两个空格作为缩进层级，让结构清晰可如果不需要格式化，可设为 None。
### 工具创建总结
LangChain提供的三种核心的工具创建方式对比特点如下：
#### @tool装饰器
- 最直接的方法，通过在函数上方添加一个装饰器，就能将其变成智能体可以调用的工具。优势是代码量极少，非常适合快速验证想法或创建参数简单的工具。
#### Pydantic 模型
- 当工具的参数变得复杂，需要枚举值、范围限制或更复杂的业务逻辑验证时，Pydantic 模型是理想选择。提供强大的类型检查和数据验证。可以使用 Literal类型限定参数为固定选项，或用 Field设置默认值、描述，甚至用 @field_validator实现自定义校验规则（如将工单ID转为大写）。非常适合参数结构稳定的企业内部工具，如工单查询、客户信息管理等。
#### JSON Schema
- 工具参数模式可以基于数据库配置或用户输入在运行时动态生成，可以选择这种方式，适用于需要高度动态行为的工具，或与已有 JSON Schema 定义的外部系统打通的场景。

### 调用工具错误处理
- 中间件的执行时机如下：用户提问 → LLM分析并决定调用工具 → LangChain准备工具调用参数 → 错误处理中间件介入 → 实际工具执行 → 结果/错误返回
- 即工具执行之前，先执行所有的错误处理中间件
- 评分例子“系统提示词中有检查到满分/10分，LLM 则将评分设置为10，但ProductEvaluation中评分范围为 1~5 ，在工具执行前，会报我们自定义错误处理函数中的范围错误异常，LLM再次推理，设置评分为当前最高分，随后在调用工具，得到结果，再LLM观察.....最终得到结果/错误返回

## Agent结构化输出
- 有三类，使用方式 在create_agent(response_format = ProviderStrategy（type[结构化响应]）/ToolStrategy(type[结构化响应]) / type[结构化响应])，
- 最后一种方式是自动挡，会根据LangChain会根据模型能力自动选择策略：如果模型支持原生结构化输出（如OpenAI、Anthropic Claude或xAI Grok），则优先使用ProviderStrategy；否则使用ToolStrategy。
- 特别注意：在LangChain 1.0及以上版本中，直接传递模式（如response_format=ContactInfo）不再支持，必须显式使用ToolStrategy或ProviderStrategy。（经过测试，目前langchain1.2版本还可使用）。

 
  

