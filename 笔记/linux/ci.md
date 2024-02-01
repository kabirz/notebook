#jenkins
[Jenkins 用户手册](https://www.jenkins.io/zh/doc/)
```
```
```diff
commit 77200ced08a699cc156844f007ceeeef46d714bd

Author: kabirz <jxwazxzhp@126.com>

Date:   Thu Jan 25 14:37:46 2024 +0800

  

    update chat

    Signed-off-by: kabirz <jxwazxzhp@126.com>

  

diff --git a/chat/chat_full.py b/chat/chat_full.py

new file mode 100644

index 0000000..c2beb23

--- /dev/null

+++ b/chat/chat_full.py

@@ -0,0 +1,68 @@

+import streamlit as st

+from streamlit_chatbox import ChatBox, Image

+import zhipuai

+

+st.set_page_config('AI聊天室', '🤖')

+

+history_len = 4

+if not st.session_state.get('zhipu_key'):

+    zhipu_key = st.text_input("请输入智谱的api key")

+    if st.button('确定'):

+        st.session_state['zhipu_key'] = zhipu_key

+        st.rerun()

+    st.stop()

+

+st.session_state.setdefault('tokens', {'pts': 0, 'cts': 0, 'tts': 0})

+

+

+def show_tokens(id: st = st.empty()):

+    return id.markdown('''**Tokens**:

+| prompts | completions | totals |

+|   ----  |     ----    |  ----  |

+|   {pts} |     {cts}   |  {tts} |

+'''.format(**st.session_state['tokens']))

+

+

+chatbox = ChatBox(assistant_avatar='🤖', user_avatar='🏆')

+with st.sidebar:

+    model = st.selectbox('请选择模型', ['glm-4', 'glm-3-turbo', '绘画'])

+    history_len = st.number_input("历史对话轮数：", 0, 20, 4)

+    chatbox.use_chat_name(model)

+    if model != '绘画':

+        with st.container():

+            token = st.empty()

+            token = show_tokens(token)

+

+st.markdown('我是智谱大语言模型，您有任何问题都可以问我。')

+chatbox.output_messages()

+

+if prompt := st.chat_input("请输入您要问的问题"):

+    client = zhipuai.ZhipuAI(api_key=st.session_state.get('zhipu_key'))

+    messages = []

+    chatbox.user_say(prompt)

+    if model in ('glm-4', 'glm-3-turbo'):

+        for his in chatbox.history[-history_len*2:]:

+            messages.append({'role': his['role'], 'content': his['elements'][0].content})

+        chatbox.ai_say('正在思考...')

+        response = client.chat.completions.create(model=model, messages=messages, stream=True)

+        text = ''

+        for chunk in response:

+            text += chunk.choices[0].delta.content

+            if chunk.usage:

+                st.session_state['tokens']['pts'] += chunk.usage.prompt_tokens

+                st.session_state['tokens']['cts'] += chunk.usage.completion_tokens

+                st.session_state['tokens']['tts'] += chunk.usage.total_tokens

+                show_tokens(token)

+            chatbox.update_msg(text)

+        chatbox.update_msg(text, streaming=False)

+    elif model == '绘画':

+        chatbox.ai_say('正在绘画中...')

+        try:

+            response = client.images.generations(model='cogview-3', prompt=prompt)

+            if response.data:

+                chatbox.update_msg(Image(response.data[0].url), streaming=False)

+            else:

+                chatbox.update_msg('绘画生成失败', streaming=False)

+        except zhipuai.core._errors.APIRequestFailedError as e:

+            out = eval(e.response.text)

+            chatbox.update_msg(out['error']['message'], streaming=False)

diff --git a/chat/chat_input_key.py b/chat/chat_input_key.py

old mode 100755

new mode 100644

diff --git a/chat/chat_out.py b/chat/chat_out.py

old mode 100755

new mode 100644

index b437509..0ecb7ee

--- a/chat/chat_out.py

+++ b/chat/chat_out.py

@@ -1,7 +1,6 @@

 import streamlit as st

 from streamlit_chatbox import ChatBox

 from zhipuai import ZhipuAI

-import os

 st.set_page_config('AI聊天室', '🤖')

@@ -19,7 +18,7 @@ st.session_state.setdefault('tokens', {'pts': 0, 'cts': 0, 'tts': 0})

 with st.sidebar:

     with st.container():

-        st.markdown('''**Tokens**:  

+        st.markdown('''**Tokens**:

 |   promts | completions | totals |

 |    ----  |     ----    | ----   |

 |    {pts} |     {cts}   |  {tts} |

@@ -44,4 +43,3 @@ if prompt := st.chat_input("请输入您要问的问题"):

             st.session_state['tokens']['tts'] += chunk.usage.total_tokens

         chatbox.update_msg(text)

     chatbox.update_msg(text, streaming=False)

-        

diff --git a/chat/chat_request.py b/chat/chat_request.py

new file mode 100644

index 0000000..1ba53d0

--- /dev/null

+++ b/chat/chat_request.py

@@ -0,0 +1,90 @@

+import requests

+import streamlit as st

+from streamlit_chatbox import ChatBox

+import json

+import time

+import jwt

+

+st.set_page_config('AI聊天室', '🤖')

+

+

+def generate_token(apikey: str, exp_seconds: int):

+    try:

+        id, secret = apikey.split(".")

+    except Exception as e:

+        raise Exception("invalid apikey", e)

+

+    payload = {

+        "api_key": id,

+        "exp": int(round(time.time() * 1000)) + exp_seconds * 1000,

+        "timestamp": int(round(time.time() * 1000)),

+    }

+

+    return jwt.encode(

+        payload,

+        secret,

+        algorithm="HS256",

+        headers={"alg": "HS256", "sign_type": "SIGN"},

+    )

+

+

+def show_tokens(id: st = st.empty()):

+    return id.markdown('''**Tokens**:

+| prompts | completions | totals |

+|   ----  |     ----    |  ----  |

+|   {pts} |     {cts}   |  {tts} |

+'''.format(**st.session_state['tokens']))

+

+

+if not st.session_state.get('zhipu_key'):

+    zhipu_key = st.text_input("请输入智谱的api key")

+    if st.button('确定'):

+        st.session_state['zhipu_key'] = zhipu_key

+        st.rerun()

+    st.stop()

+

+history_len = 4

+st.session_state.setdefault('tokens', {'pts': 0, 'cts': 0, 'tts': 0})

+chatbox = ChatBox(assistant_avatar='🤖', user_avatar='🏆')

+with st.sidebar:

+    model = st.selectbox('请选择模型', ['glm-4', 'glm-3-turbo'])

+    history_len = st.number_input("历史对话轮数：", 0, 20, 4)

+    chatbox.use_chat_name(model)

+    with st.container():

+        token = st.empty()

+        token = show_tokens(token)

+

+st.markdown('我是智谱大语言模型，您有任何问题都可以问我。')

+chatbox.output_messages()

+history_len = 4

+url = 'https://open.bigmodel.cn/api/paas/v4/chat/completions'

+headers = {

+    "Accept": "application/json",

+    "Content-Type": "application/json; charset=UTF-8",

+    "Authorization": generate_token(st.session_state['zhipu_key'], 120)

+}

+messages = []

+

+if prompt := st.chat_input('请输入您要问的问题'):

+    chatbox.user_say(prompt)

+    for his in chatbox.history[-history_len*2:]:

+        messages.append({'role': his['role'], 'content': his['elements'][0].content})

+    data = {'model': 'glm-4', 'messages': messages, 'stream': True}

+    chatbox.ai_say('正在思考...')

+    response = requests.post(url, json=data, headers=headers, stream=True)

+    text = ''

+    for chunk in response.iter_lines():

+        data_str = chunk.decode()

+        if len(data_str) > 20:

+            data_dict = json.loads(data_str[6:])

+            buf = data_dict['choices'][0]['delta']['content']

+            text += buf

+            if data_dict.get('usage'):

+                usage = data_dict.get('usage')

+                st.session_state['tokens']['pts'] += usage['prompt_tokens']

+                st.session_state['tokens']['cts'] += usage['completion_tokens']

+                st.session_state['tokens']['tts'] += usage['total_tokens']

+                show_tokens(token)

+                chatbox.update_msg(text, streaming=False)

+            else:

+                chatbox.update_msg(text)

diff --git a/chat/chat_tokens.py b/chat/chat_tokens.py

index 5ba6fd0..af0ae67 100644

--- a/chat/chat_tokens.py

+++ b/chat/chat_tokens.py

@@ -14,7 +14,7 @@ st.session_state.setdefault('tokens', {'pts': 0, 'cts': 0, 'tts': 0})

 with st.sidebar:

     with st.container():

-        st.markdown('''**Tokens**:  

+        st.markdown('''**Tokens**:

 |   promts | completions | totals |

 |    ----  |     ----    | ----   |

 |    {pts} |     {cts}   |  {tts} |
```