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
### 模型调用配置
- 在模型运行时可以让其动态的执行一些配置，比如模型调用基本参数 model,temperature。或者是 自定义回调函数
- 自定义回调函数，该函数可以在模型返回结果前后执行我们想做的事情，
  - 比如模型使用前 将信息记录到日志系统或数据库（模型配置信息持久化）
  - 比如模型使用前 根据 metadata 中的 user_id 进行用户行为分析（用户在LLM使用次数与时间的关系）
  - 比如模型使用后  可以在这里记录运行结束的信息，比如消耗的令牌数等存储
 
  

