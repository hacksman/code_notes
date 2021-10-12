将 `'1+2+3'` 转换成 `[1, +, 2, +, 3]`

```python
def split_keep_deli(string_to_split, deli):  
    result_list = []  
    tokens = string_to_split.split(deli)  
    for i in range(len(tokens) - 1):  
        result_list.append(tokens[i])  
        result_list.append(deli)  
    result_list.append(tokens[len(tokens)-1])  
    return result_list
```

运行以下命令
```python
s = '1+2+3'
r = split_keep_deli(s, '+')
print(r)
```

若还需要分割其它的字符，比如原表达式中可能存在 `+ - * /` 运算符，那么我们可以定义一个函数这样分割

```python
def split_deli(string):  
    s_s = split_keep_deli(string, '+')  
  
    s_s = [split_keep_deli(item, '-') for item in s_s]  
    s_s = [_ for r in s_s for _ in r]  
  
    s_s = [split_keep_deli(item, '*') for item in s_s]  
    s_s = [_ for r in s_s for _ in r]  
  
    s_s = [split_keep_deli(item, '/') for item in s_s]  
    s_s = [_ for r in s_s for _ in r]  
  
    s_s = [split_keep_deli(item, '|') for item in s_s]  
    s_s = [_ for r in s_s for _ in r]  
  
    return s_s
```

## 参考资料
[Python: How can I include the delimiter(s) in a string split? [duplicate]](https://stackoverflow.com/questions/21208223/python-how-can-i-include-the-delimiters-in-a-string-split)

[How to make a flat list out of a list of lists](https://stackoverflow.com/questions/952914/how-to-make-a-flat-list-out-of-a-list-of-lists)