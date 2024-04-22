以chatglm为例：

ChatGLM继承以LLM
实现父类方法

`_llm_type`: chat_glm
`_identifying_params`: 提供给llm的`save`函数保存llm
`_call`: 实现llm调用， 使用requests的post方法和llm通信，进行问答
