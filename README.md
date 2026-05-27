# Langchain-agent-study
Langchain-Agent学习，Langchain长短期记忆
## 2.3模型结构化输出
 1. pydantic 最好用，体验最好，有数据验证，适配复杂数据类型
 2. 变量名:类型 = Field(Description = " ")
 3. TypeDict最轻量化，效果一般
 4. JsonSchema 跨语言，很灵活，效果也不错
## 2.4 模型工具调用
### 多工具调用 （查询某企业的股价与新闻）
1. 准备： 股价和新闻查询工具，大模型
2. 步骤：
   - 绑定LLM与工具 得到 LLM_T
   - 将 HumenMessage == 腾讯的股价+新闻 送入 LLM_T 构造 AIMessage,其中含有 tool_calls
   - 从 AIMessage 中拿出 tool_calls 来 执行工具 得到 ToolMessage，（目前做的是同步执行工具，异步还没做到）
   - 将所有的Message 组合 ，送入 LLM_T 得到最终的结果。
  

