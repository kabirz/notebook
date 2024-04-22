#langchain #llm
`BaseLanguageModel`

主要包含以下方法

`generate_prompt`： 
```python
def generate_prompt(
	self,
	prompts: List[PromptValue],
	stop: Optional[List[str]] = None,
	callbacks: Callbacks = None,
	**kwargs: Any,
) -> LLMResult:
```
这个方法是抽象方法，需要子类来实现，功能是通过提示词调用llm并返回输出，其中
1. `prompts`是一个可换成字符串和`messages`类的元素的列表
2.  `stop`  生成时使用的停用词。模型输出在第一次出现这些子字符串中的任何一个时被截断
3. `callbacks` 回调函数
4. `kwargs` 其他参数

`agenerate_prompt`
```python
async def agenerate_prompt(
	self,
	prompts: List[PromptValue],
	stop: Optional[List[str]] = None,
	callbacks: Callbacks = None,
	**kwargs: Any,
) -> LLMResult:
```

是`generate_prompt`的异步调用

`predict` `predict_messages` `apredict` `apredict_messages`
这四个方法是需要子类实现的抽象方法，都是处理raw 数据

`get_token_ids` 把str转换成token的方法，输出是List[int]
`get_num_tokens` 计算token的长度
`get_num_tokens_from_messages`通过输入的messages计算输入的tokens
