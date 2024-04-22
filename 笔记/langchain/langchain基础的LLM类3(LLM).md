#langchain #llm 

`LLM`:
实现方法

`_acall`: 异步调用抽象方法`_call`

抽象方法：
`_call` : 内部调用llm
`_llm_type`:  模型类型，返回字符串，比如 chat_glm

实现基类方法：
`_generate`: 内部调用`_call`
`_agenerate`: 内部调用`_acall`