漏洞出现在这个函数上，他会匹配返回结果中用三个反引号包裹的代码，并使用eval执行

```python
def eval_code_blocks(text):
    pattern = r"```(?:python)?\s*(.*?)```"
    blocks = re.findall(pattern, text, re.S)

    for block in blocks:
        try:
            result = eval(block)
        except Exception as e:
            result = f"[Error]: {e}"

        text = text.replace(f"```{block}```", str(result))

    return text
```

这里用加号来混淆，让ai识别不到我们的真实目的即可

![image-20251211201506610](wp/image-20251211201506610.png)





