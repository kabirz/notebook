#BaseLLM
关键属性
`callbacks`
`callback_manager`
方法
`invoke`， `ainvoke`: llm调用， 里面直接调用基类的`generate_prompt` `agenerate_prompt`方法
`batch` `abatch`暂时未用到
`stream` `astream` 数据流处理方法， 需要实现`_stream` `_astream`, 主要这两个方法需要子类去实现
`_agenerate` 异步调用`_generate`
`generate`: llm调用
`agenerate`: llm异步调用
call方法： 调用 `generate`
异步call方法： 调用 `agenerate`
`save`: 保存llm

实现基类方法
`generate_prompt`: 调用`generate`
`agenerate_prompt`: 调用`agenerate`

`predict` `predict_messages` `apredict` `apredict_messages`
调用call方法实现预测


需要子类实现的方法:
`_generate` 需要子类实现






