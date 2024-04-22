命令行和openai resful api通信方法

openai -b http://10.239.137.65:20000/v1/ -k EMPTY -t openai api chat.completions.create -m chatglm3-6b -g user 你是谁 -n 2 --stream -t 0.5 -P 1.0