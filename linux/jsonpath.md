### 参考资料

课程：https://kodekloud.com/courses/json-path-quiz/

注意点：

- json query 得到的都是数组形式的 result

### list

list 有序（序号从0开始），而 dictionary 无序

- 取第0个和第3个：`$[0,3]`
- 取第0个到第3（不包含）个：`$[0:3]`
- 每隔2个取第0个到第8（不包含）个：`$[0:8:2]`
- 取最后1个：在某些实现下不起作用`$[-1]`，需要写成`$[-1:0]`，`$[-1:]`

### 操作符

|         操作符          |          描述          |
| :---------------------: | :--------------------: |
|            $            |       root根元素       |
|            @            | 表示list中的每一个元素 |
|            *            |        匹配所有        |
|           ?()           |       if过滤匹配       |
| ['<name>' (, '<name>')] |  用方括号和单引号取值  |

### 逻辑表达式

|       逻辑表达式        |        描述        |
| :---------------------: | :----------------: |
|  ==, !=, <, <=, >, >=   |  等于，不等于...   |
|           =~            | 右边匹配正则表达式 |
|         in, nin         | 匹配是否存在数组中 |
| subsetof, anyof, noneof |      子集匹配      |
|       size, empty       |    数组大小匹配    |

### 函数

|        函数        |               描述                |  输出类型  |
| :----------------: | :-------------------------------: | :--------: |
|   min, max, avg    |          最小最大平均值           |   Double   |
|       stddev       |            标准偏差值             |   Double   |
|        sum         |               求和                |   Double   |
|       length       |               长度                |  Integer   |
|        keys        | 提供属性键（终端波浪号~的替代项） |   Set<E>   |
|   concat, append   |            拼接，加入             | like input |
| first, last, index |             数组元素              | 由数组决定 |

### kubectl

`$`非强制需要写上，kubectl会帮助添加

拼接内容：`kubectl get nodes -o=jsonpath='{.items[*].metadata.name}{"\n"}{.items[*].status.capacity.cpu}'`

Loop：

```yaml
'{range .items[*]}
	{.metadata.name}{"\t"}{.status.capacity.cpu}{"\n"}
{end}'
```

custom-columns：

`kubectl get nodes -o=custom-columns=<COLUMN NAME>:<JSON PATH>`