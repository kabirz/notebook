#langchain 

## `BaseCallbackHandler`
继承自：
### `LLMManagerMixin`
`on_llm_new_token` :当有新的token来的时候执行
`on_llm_end`: llm结束的时候执行
`on_llm_error`: llm报错的时候执行

### `ChainManagerMixin`
`on_chain_end` chain调用报错的时候执行
`on_chain_error` chain调用报错的时候执行
`on_agent_action` agent action的时候执行
`on_agent_finish` agent finish的时候执行

### `ToolManagerMixin`
`on_tool_end` 工具调用结束的时候执行
`on_tool_error` 工具调用报错的时候执行

### `RetrieverManagerMixin`
`on_retriever_end` 检索结束的时候执行
`on_retriever_error` 检索报错的时候执行

### `CallbackManagerMixin`
提供以下接口需要子类实现
`on_llm_start` 当开始跑llm的时候执行
`on_chat_model_start` 当模型开始跑的时候执行
`on_retriever_start` 开始检索的时候执行
`on_chain_start` chain开始的时候执行
`on_tool_start` 工具开始的时候执行

### `RunManagerMixin`
`on_text` 当有任意text产生的时候执行
`on_retry` 当触发retry的时候执行

## `AsyncCallbackHandler` 继承自 `BaseCallbackHandler`, 无新增方法,  但是都是异步的， 方法都没有实现，需要子类去实现


## `AsyncIteratorCallbackHandler` 继承自`AsyncCallbackHandler`
异步迭代调用 类内有一个queue和一个event，
`on_llm_start` 时候清理 event,  `on_llm_end` 和 `on_llm_error` 的时候 设置event， 表示llm调用已经完成
`on_llm_new_token` 时把产生的token 放到queue中，
`aiter`留给用户调用，把queue中的token取出， 当用户终端完后需要设置event 

## `BaseCallbackManager` 继承自 `CallbackManagerMixin`, 可实例化
 管理callbackhandler， 可以实现设置添加删除 handler


## `AsyncCallbackManager` 继承自`BaseCallbackManager`, 异步callbackManager， 异步实现了`BaseCallbackManager`的调用，方法如下
```python

for prompt in prompts:
	run_id_ = uuid.uuid4()

	tasks.append(
		ahandle_event(
			self.handlers,
			"on_llm_start",
			"ignore_llm",
			serialized,
			[prompt],
			run_id=run_id_,
			parent_run_id=self.parent_run_id,
			tags=self.tags,
			metadata=self.metadata,
			**kwargs,
		)
	)

	managers.append(
		AsyncCallbackManagerForLLMRun(
			run_id=run_id_,
			handlers=self.handlers,
			inheritable_handlers=self.inheritable_handlers,
			parent_run_id=self.parent_run_id,
			tags=self.tags,
			inheritable_tags=self.inheritable_tags,
			metadata=self.metadata,
			inheritable_metadata=self.inheritable_metadata,
		)
	)

await asyncio.gather(*tasks)
```

`ahandle_event` 实现如下
```python
    await asyncio.gather(
        *(
            _ahandle_event_for_handler(
                handler,
                event_name,
                ignore_condition_name,
                *args,
                **kwargs,
            )
            for handler in handlers
            if not handler.run_inline
        )
    )
# _ahandle_event_for_handler
event = getattr(handler, event_name)
await event(*args, **kwargs)
```


