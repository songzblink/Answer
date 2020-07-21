## 题目

**题目描述：**给定一个非空的整数数组，返回其中出现频率前 **k** 高的元素。

**示例1：**

```c++
输入: nums = [1,1,1,2,2,3], k = 2
输出: [1,2]
```

**示例2：**

```c++
输入: nums = [1], k = 1
输出: [1]
```

**提示：**

- 你可以假设给定的 k 总是合理的，且 1 ≤ k ≤ 数组中不相同的元素的个数。
- 你的算法的时间复杂度必须优于 O(n log n) , n 是数组的大小。
- 题目数据保证答案唯一，换句话说，数组中前 k 个高频元素的集合是唯一的。
- 你可以按任意顺序返回答案。

## 解析

【小顶堆】

==降序用小顶堆 升序用大顶堆==

小顶堆的实现利用*PriorityQueue*类（即优先队列）来实现。**优先队列的作用是能保证每次取出的元素都是队列中权值最小的，元素大小的评判可以通过元素本身的自然顺序，也可以通过构造时传入的比较器。**该类的定义如下：

```java
public class PriorityQueue<E>
extends AbstractQueue<E>
implements Serializable
```

可以发现它实现了*Queue*接口，不允许放入`null`元素。它有重要方法：

- public boolean add([E](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/PriorityQueue.html) e)：将指定的元素插入此优先级队列，失败跑出异常；
- public boolean offer([E](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/PriorityQueue.html) e)：将指定的元素插入此优先级队列，失败返回`null`；
- public [E](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/AbstractQueue.html) element()：检索但不删除此队列的头；
- public [E](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/util/AbstractQueue.html) remove()：检索并删除此队列的头，返回头；
- public boolean remove([Object](https://docs.oracle.com/en/java/javase/14/docs/api/java.base/java/lang/Object.html) o)：从队列中移除指定元素的单个实例，失败抛出异常；

于是我们可以用一下思路来解题：

- 借助*HashMap*建立数字和其出现次数的映射，遍历一遍数组统计元素出现的频率；
- 维护一个元素数目为*k*的小顶堆；
- 每次都将新的元素与堆顶元素（堆中频率最小的元素）进行比较
  - 如果新的元素的频率比堆顶端元素大，弹出堆顶端的元素，将新元素添加进堆中；
- 最终，堆中的k个元素即为前k个高频元素

**代码：**

```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        Map<Integer, Integer> map = new HashMap<Integer, Integer>();
        int[] topK = new int[k];
        for(int num : nums){
            map.put(num, map.getOrDefault(num, 0) + 1);
        }
        
        // 遍历map，用最小堆保存频率最大的k个元素
        PriorityQueue<Integer> pq = new PriorityQueue<Integer>(new Comparator<Integer>() {
            @Override
            public int compare(Integer obj1, Integer obj2) {
                return map.get(obj1) - map.get(obj2);
            }
        });
        for(int key : map.keySet()) {
            if(pq.size() < k) { // 维护一个元素数目为k的小顶堆
                pq.add(key);
            } else { // 每次都将新的元素与堆顶元素（堆中频率最小的元素）进行比较
                if(map.get(pq.element()) < map.get(key)) { // 如果新的元素的频率比堆顶端元素大，弹出堆顶端的元素，将新元素添加进堆中
                    pq.remove();
                    pq.add(key);
                }
            }
        }
        
        // 取出最小堆中的元素
        int x = 0;
        while(!pq.isEmpty()){
            topK[x++] = pq.remove();
        }
        return topK;
    }
}
```



【桶排序法】

和上面的方法类似，利用HashMap统计频率。在统计完成之后，创建一个数组，将频率作为数组下表，对于出现频率不同的数字集合，存入对应的数组下标，再倒序从最高频率开始取*k*个就完成啦。

```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        Map<Integer, Integer> map = new HashMap<Integer, Integer>();
        int[] topK = new int[k];

        for(int num : nums){
            map.put(num, map.getOrDefault(num, 0) + 1);
        }

        //桶排序
        //将频率作为数组下标，对于出现频率不同的数字集合，存入对应的数组下标
        int[] res = new int[k];
        List<Integer>[] list = new List[nums.length+1];
        for(int key : map.keySet()) {
            if(list[map.get(key)] == null) {
                list[map.get(key)] = new ArrayList();
            }
            list[map.get(key)].add(key);
        }

        // 倒序遍历数组获取出现顺序从大到小的排列
        for(int i = list.length - 1, j = 0; i > 0 && j < k; i--) {
            if(list[i] == null) continue;
            // res.addAll(list[i]);
            for(int key : list[i]) {
                res[j++] = key;
            }
        }
        return res;
    }
}
```

