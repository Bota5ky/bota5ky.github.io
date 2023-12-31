### 匹配模式
- 讲道理的正则表达式对于 *+? 的匹配模式是<font color=orange>贪心匹配</font>
  - 用<font color=orange>a\*</font><font color=green>a\*</font>b匹配aab时，<font color=orange>a\*</font>吃掉了两个a，而<font color=green>a\*</font>没得a吃
- 在 *+? 后面加上一个 ? 可以把匹配模式变为<font color=orange>惰性匹配</font>
  - 用<font color=orange>a\*?</font><font color=green>a\*?</font>b 匹配aab时，<font color=orange>a\*?</font>一个a都不吃，而<font color=green>a\*?</font>被迫吃掉了两个a
- 在 *+? 后面加上一个+可以把匹配模式变为<font color=red>侵占匹配</font>
  - 用<font color=orange>a\*+</font><font color=green>a</font>无法匹配aaa，因为<font color=orange>a\*+</font>抢走了所有的a，导致<font color=green>a</font>匹配失败

### 分组
- <font color=green>\\<no.></font> 如\1，引用前面的第1个分组
```csharp
var match = Regex.Match("abcabc", @"(.+)\1");
// match.Groups[1] = "abc"
```
```csharp
var result = Regex.Replace(" 358 * (126 + 282)", @"(\d+) \* \((\d+) \+ (\d+)\)", "$1 * $2 + $1 * $3");
// result = "358 * 126 + 358 * 282"
```
- 对于不想要的分组，开头加上 ?:
```csharp
var match = Regex.Match("abcabc", @"(?:.+)\1");
// Reference to undefined group number 1
```
注意只要有一对单独的圆括号就存在一个组，顺序为开括号出现的顺序
```csharp
 var match = Regex.Match("abcdef", @"(?:a(bc)(d(ef)))");
 // match.Groups[1] = bc
 // match.Groups[2] = def
 // match.Groups[3] = ef
```
- 分组时用 ?<name> 指定名字，即可用 \k<name> 的方式引用
```csharp
var match = Regex.Match("abcabc", @"(?<x>.+)\k<x>")
// match.Groups["x"] = abc 正则替换的引用方式:
var result = Regex.Replace(" 358 + 126", @"(?<A>\d+) \+ (?<B>\d+)", "${B} + ${A}")
// result = "126 + 358"
```

### 断言
- `(?<=x)y` 匹配前面匹配 x 的 y
- `x(?=y)` 匹配后面匹配 y 的 x
- `(?<!x)y` 匹配前面不匹配 x 的 y
- `x(?!y)` 匹配后面不匹配 y 的 x
- `^` 断言当前位置为字符串开头
- `$` 断言当前位置为字符串末尾
- `\b` 断言当前位置为单词(包括数字下划线)的边界

### C# 特供
- C# 对于正则的分组行为强于其它语言
  - 组以堆栈的形式存在而非单个元素
  - 对于同组匹配多个的正则，能同时存储被匹配的内容
  - 对于带名称的分组效果相同
  - 除了放入元素还可以取出元素，遵循堆栈的后入先出原则
```csharp
Pattern p = Pattern.compile("(.)+")
Matcher m = p.matcher("abc");
if (m.matches())
  System.out.println(m.group(1));
// 输出:c
```
```csharp
var groups = Regex.Match("abc", "(.)+").Groups;
var captures = groups[1].Captures.ToArray();
var output = string.Join<Capture>(","), captures);
Console.WriteLine(output);
// 输出:a,b,c
```

[Multiple outputs from T4 made easy – revisited](https://damieng.com/blog/2009/11/06/multiple-outputs-from-t4-made-easy-revisited/)