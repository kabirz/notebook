- sed: 
[useful-sed](https://github.com/adrianlarion/useful-sed)
- awk:
[AWK 程序设计语言](https://github.com/wuzhouhui/awk)

- **rename**：rename 是一个内置命令，用于批量修改文件名。rename 的语法如下：

```shell
rename [OPTIONS] [old_name] [new_name] [files...]
```

例如，要将所有以 .jpg 结尾的文件重命名为 .png，可以使用以下命令：

```shell
rename 's/\.jpg/.png/' *.jpg
```
