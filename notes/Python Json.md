## 1. `json.dump()`

把 Python 对象**直接写进文件对象**。

- 输入：Python dict / list
- 输出：写到一个已经打开的文件里

逻辑上像这样：
```python
json.dump(data, file)
```

适合你在处理完结果后，直接把结果保存成 `.json` 文件。

---

## 2. `json.dumps()`

把 Python 对象**转换成 JSON 格式的字符串**。

也就是：

- `dumps` = **dump to string**

逻辑上像这样：
```python
json_string = json.dumps(data)
```


### 1. `indent`

控制缩进层数，例如：
```python
json.dumps(data, indent=4)
```

### 2. `ensure_ascii=False`

如果有中文，避免变成 `\uXXXX`：
```python
print(json.dumps(data, indent=2, ensure_ascii=False))
```

### 3. `sort_keys=True`

按 key 排序，方便比较：
```python
print(json.dumps(data, indent=2, ensure_ascii=False, sort_keys=True))
```


---

## 1. `json.load()`

把 **文件对象里的 JSON** 读出来，转成 Python 对象。

也就是：

- `load` = **load from file**

例如文件里有：
```
{"lang": "en", "count": 10}
```


你这样读：
```python
import json  
  
with open("result.json", "r") as f:  
    data = json.load(f)  
    print(data)
```

得到的是 Python 对象：
```python
{'lang': 'en', 'count': 10}
```

---

## 2. `json.loads()`

把 **字符串形式的 JSON** 读出来，转成 Python 对象。

也就是：

- `loads` = **load from string**

例如：
```python
import json  
  
s = '{"lang": "en", "count": 10}'  
data = json.loads(s)  
  
print(data)
```

得到同样的 Python 对象：
```
{'lang': 'en', 'count': 10}
```
