### 1. 排序算法

**稳定性**：用于描述排序算法在处理相等元素时是否保持它们的相对顺序不变。

**各种排序算法的时间和空间复杂性**

<center><table>
  <tr>
    <th rowspan="2" align="center">类别</th>
    <th rowspan="2" align="center">排序方法</th>
    <th colspan="3" align="center">时间复杂度</th>
    <th align="center">空间复杂度</th>
    <th rowspan="2" align="center">稳定性</th>
  </tr>
  <tr>
    <td align="center">平均情况</td>
    <td align="center">最好情况</td>
    <td align="center">最坏情况</td>
    <td align="center">辅助存储</td>
  </tr>
  <tr>
    <td rowspan="2" align="center">插入排序</td>
    <td align="center">直接插入</td>
    <td align="center">O(n<sup>2</sup>)</td>
    <td align="center">O(n)</td>
    <td align="center">O(n<sup>2</sup>)</td>
    <td align="center">O(1)</td>
    <td align="center">稳定</td>
  </tr>
  <tr>
    <td align="center">希尔排序</td>
    <td align="center">O(n<sup>1.3</sup>)</td>
    <td align="center">O(n)</td>
    <td align="center">O(n<sup>2</sup>)</td>
    <td align="center">O(1)</td>
    <td align="center">不稳定</td>
  </tr>
  <tr>
    <td rowspan="2" align="center">选择排序</td>
    <td align="center">直接选择</td>
    <td align="center">O(n<sup>2</sup>)</td>
    <td align="center">O(n<sup>2</sup>)</td>
    <td align="center">O(n<sup>2</sup>)</td>
    <td align="center">O(1)</td>
    <td align="center">不稳定</td>
  </tr>
  <tr>
    <td align="center">堆排序</td>
    <td align="center">O(nlogn)</td>
    <td align="center">O(nlogn)</td>
    <td align="center">O(nlogn)</td>
    <td align="center">O(1)</td>
    <td align="center">不稳定</td>
  </tr>
  <tr>
    <td rowspan="2" align="center">交换排序</td>
    <td align="center">冒泡排序</td>
    <td align="center">O(n<sup>2</sup>)</td>
    <td align="center">O(n)</td>
    <td align="center">O(n<sup>2</sup>)</td>
    <td align="center">O(1)</td>
    <td align="center">稳定</td>
  </tr>
  <tr>
    <td align="center">快速排序</td>
    <td align="center">O(nlogn)</td>
    <td align="center">O(nlogn)</td>
    <td align="center">O(n<sup>2</sup>)</td>
    <td align="center">O(nlogn)</td>
    <td align="center">不稳定</td>
  </tr>
  <tr>
    <td colspan="2" align="center">归并排序</td>
    <td align="center">O(nlogn)</td>
    <td align="center">O(nlogn)</td>
    <td align="center">O(nlogn)</td>
    <td align="center">O(n)</td>
    <td align="center">稳定</td>
  </tr>
  <tr>
    <td colspan="2" align="center">基数排序</td>
    <td align="center">O(d(r+n))</td>
    <td align="center">O(d(n+rd))</td>
    <td align="center">O(d(r+n))</td>
    <td align="center">O(rd+n)</td>
    <td align="center">稳定</td>
  </tr>
</table></center>

说明：希尔排序根据不同的增量序列得到的复杂度分析也不同，这里取 N//2。

**InsertionSort 插入排序：**

```go
// InsertionSort 插入排序 稳定
// 时间复杂度：平均 O(n^2) 最好 O(n) 最坏 O(n^2) 
// 空间复杂度：O(1)
func InsertionSort(nums []int) {
	for i := 1; i < len(nums); i++ {
		temp := nums[i] /* 取出未排序序列中的第1个元素*/
		var j int
		for j = i; j > 0 && nums[j-1] > temp; j-- {
			nums[j] = nums[j-1] /*依次与已排序序列中元素比较并右移*/
		}
		nums[j] = temp /* 放进合适的位置 */
	}
}
```

**ShellSort 希尔排序：**

```go
// ShellSort 希尔排序 不稳定
// 时间复杂度：平均 O(n^1.3) 最好 O(n) 最坏 O(n^2) 
// 空间复杂度：O(1)
func ShellSort(nums []int) {
	for k := len(nums) / 2; k > 0; k /= 2 {
		for i := k; i < len(nums); i++ {
			temp := nums[i] /* 取出未排序序列中的第k个元素*/
			var j int
			for j = i; j >= k && nums[j-k] > temp; j -= k { /* 注意界限 j >= k */
				nums[j] = nums[j-k] /*依次与已排序序列中元素比较并右移*/
			}
			nums[j] = temp /* 放进合适的位置 */
		}
	}
}
```

**HeapSort 堆排序：**

```go
// HeapSort 堆排序 不稳定
// 时间复杂度：平均 O(nlogn) 最好 O(nlogn) 最坏 O(nlogn)
// 空间复杂度：O(1)
func HeapSort(nums []int) {
	for i := len(nums); i > 0; i-- {
		heapify(nums[:i])
		nums[0], nums[i-1] = nums[i-1], nums[0]
	}
}

func heapify(nums []int) {
	last := len(nums)/2 - 1 //最后一个非叶子节点
	for i := last; i >= 0; i-- {
		max := i
		left := 2*i + 1
		right := 2*i + 2
		if left < len(nums) && nums[left] > nums[max] {
			max = left
		}
		if right < len(nums) && nums[right] > nums[max] {
			max = right
		}
		nums[max], nums[i] = nums[i], nums[max]
	}
}
```

**QuickSort 快速排序：**

```java
public void quickSort(int head, int tail) {
    if (tail - head < 1) {
        return;
    }
    var pivot = nums[head];
    var i = head;
    var j = tail;
    while (i < j) {
        while (i < j && nums[j] >= pivot) {
            j--;
        }
        nums[i] = nums[j];
        while (i < j && nums[i] <= pivot) {
            i++;
        }
        nums[j] = nums[i];
    }
    nums[i] = pivot;
    quickSort(head, i - 1);
    quickSort(i + 1, tail);
}
```

### 2. LRU (Least Recently Used) 最近最少使用

JDK自带实现，继承`LinkedHashMap`，重写`removeEldestEntry`方法

```java
class LRUCache extends LinkedHashMap<Integer, Integer> {
    //构造器等实现

    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
        return size() > capacity;
    }
}
```

对应力扣题 146. LRU 缓存，采用双向链表实现

### 3. LFU (Least Frequently Used) 最不经常使用

对应力扣题 460. LFU 缓存，采用多条双向链表实现

```java
class LFUCache {
    private final DoubleLinkedList tailList;
    private final Map<Integer, Node> cache;
    private final int capacity;

    private static class DoubleLinkedList {
        int frequency;
        Node head;
        Node tail;
        DoubleLinkedList moreFreq;
        DoubleLinkedList lessFreq;

        DoubleLinkedList(int frequency) {
            //只保留了基本的数据结构
        }
    }

    private static class Node {
        int key;
        int val;
        Node pre;
        Node next;
        DoubleLinkedList doubleLinkedList;
    }
}
```

### 4. KMP 算法

`next[j-1]`代表子串可以“跳过匹配”的字符个数，参考教程 [BV1AY4y157yL](https://www.bilibili.com/video/BV1AY4y157yL/)

| 子串 |  A   |  B   |  A   |  C   |  A   |  B   |  A   |  B   |
| :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: | :--: |
| next |  0   |  0   |  1   |  0   |  1   |  2   |  3   |  2   |

```java
var next = new int[pattern.length()];
for (int i = 1, j = 0; i < pattern.length(); i++) {
    while (j > 0 && pattern.charAt(i) != pattern.charAt(j)) {
        j = next[j - 1];
    }
    if (pattern.charAt(i) == pattern.charAt(j)) {
        j++;
    }
    next[i] = j;
}
```

遍历时能保证`i`一直增加

### 5. 位运算

**绝对值**

```java
mask=num>>(num.bit_length()-1)  # 获取符号位，正数为0，负数为-1

(num+mask)^mask  # 使用异或运算去除符号位 -6^-1=5 6^0=6
(num^mask)-mask  # 另一种写法
```

