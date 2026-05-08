## Group Bar Chart

```python
import numpy as np
import matplotlib.pyplot as plt

# 1. Data
categories = ['unmatched', 'malformed', 'empty', 'matched']
bluesky_counts = [len(unmatched_objs_bluesky), len(malformed_lines_bluesky), len(empty_objs_bluesky), len(matched_objs_bluesky)]
mastodon_counts = [len(unmatched_objs_mastodon), len(malformed_lines_mastodon), len(empty_objs_mastodon), len(matched_objs_mastodon)]

# 2. X positions for groups
x = np.arange(len(categories))   # [0, 1, 2, 3]
width = 0.35                     # width of each bar

# 3. Create figure and axis
fig, ax = plt.subplots(figsize=(10, 6))

# 4. Plot grouped bars
bars1 = ax.bar(x - width/2, bluesky_counts, width, label='BlueSky')
bars2 = ax.bar(x + width/2, mastodon_counts, width, label='Mastodon')

# 5. Add labels and title
ax.set_xlabel('Category')
ax.set_ylabel('Count')
ax.set_title('Data Quality Comparison')
ax.set_xticks(x)
ax.set_xticklabels(categories)

# 6. Add legend
ax.legend()

# 7. Add value labels on top of bars
ax.bar_label(bars1, padding=3)
ax.bar_label(bars2, padding=3)

# 8. Adjust layout and show
plt.tight_layout()
plt.show()
```


```python
x = np.arange(len(categories))
x = array([0, 1, 2, 3])
```
`len(categories) = 4`
它不是给用户看的，而是给 matplotlib 看的“数值坐标”。

---

```python
width = 0.35
```
这个值决定：
- 柱子粗细
- 同一组里柱子之间的间隔效果

常用值一般在：
	- `0.25`
	- `0.3`
	- `0.35`
	- `0.4`

---

```python
fig, ax = plt.subplots(figsize=(10, 6))
```

- 一张图 `fig`
- 一个坐标轴 `ax`

`figsize=(10, 6)` 表示图的大小：宽 10 高 6

单位可以近似理解成英寸，不必深究。  只要知道数字越大，图越大。

---

```python
ax.bar(...)
```

第一组柱子 `bars1 = ax.bar(x - width/2, bluesky_counts, width, label='BlueSky')`

参数含义：
- `x - width/2`
	柱子的 x 坐标，也就是柱子中心的位置。
	这里把 BlueSky 放到每组中心的左边。
	
- `bluesky_counts`  柱子的高度。
	
- `width`  柱宽
	
- `label='BlueSky'` 这个标签是给 legend 用的。


`bars2 = ax.bar(x + width/2, mastodon_counts, width, label='Mastodon')`
完全一样，只是换成右移。

---


你现在的 x 坐标其实还是 `[0, 1, 2, 3]`。  
但用户不该看到 0、1、2、3，而应该看到类别名。


```python
ax.set_xticks(x)  
ax.set_xticklabels(categories)
```

这两句的作用是：

`ax.set_xticks(x)` 告诉 matplotlib x 轴刻度放在 `[0,1,2,3]`
`ax.set_xticklabels(categories)` 告诉 matplotlib：这些刻度分别显示成categories list 中的值

---

```python
ax.legend()
```

因为你前面写了：
```python
label='BlueSky'  
label='Mastodon'
```

所以 `legend()` 会自动生成图例。
```python
ax.legend()
```

效果就是右上角或合适位置出现：
- BlueSky
- Mastodon

---

```python
ax.bar_label(...)
```

这是给每根柱子顶部加数字。

```python
ax.bar_label(bars1, padding=3)  
ax.bar_label(bars2, padding=3)
```

这里：
- `bars1` 和 `bars2` 是 `ax.bar(...)` 返回的柱子对象集合
- `padding=3` 表示数字和柱顶的距离

如果你不写这两句，图也能正常显示，只是柱子上方没有具体数字。

---

```python
plt.tight_layout()
```

作用是自动调整边距，避免：
- 标题被切掉
- x 轴标签挤在边缘
- 图例和图重叠得太严重
