
拿到llm的实列
比如openai
```python
from langchain.llms import OpenAI

llm = OpenAI(temperature=0.1)
```
调用方法
1. `OpenAI`的父类是`LLM` 的父类 `BaseLLM` 实现了call同步调用，所有直接调用`llm("你是谁")`即可 
2. 



![[Pasted image 20240203135550.png]]