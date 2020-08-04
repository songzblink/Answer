**题目描述：**给定两个大小为 m 和 n 的正序（从小到大）数组 `nums1` 和 `nums2`。

请你找出这两个正序数组的中位数，并且要求算法的时间复杂度为 *O(log(m + n))*。

你可以假设 `nums1` 和 `nums2` 不会同时为空。

**示例1：**

```
nums1 = [1, 3]
nums2 = [2]

则中位数是 2.0
```

**示例2：**

```
nums1 = [1, 2]
nums2 = [3, 4]

则中位数是 (2 + 3)/2 = 2.5
```

**解析：**

## 暴力

用`len`表示两个数组合并后的长度，如果是奇数，那么我们需要知道第`(len+1)/2+1`个数就能得到中位数，需要遍历`int(len/2)+1`次；如果是偶数，需要知道第`len/2`和`len/2+1`个数，同样也是需要遍历`int(len/2)+1`次。所以不论是奇数还是偶数，都是`int(len/2)+1`次。

设`left`和`right`分别记录上一次循环的结果和当前循环的结果，每次循环前将`right`的值赋给`left`，这样在最后一次循环的时候，`left`得到`right`的值，即第`len/2`个数，right更新为最后一次循环的结果，即`len/2+1`。

```
初始态：left = -1, right = -1
第一次：left = -1, right = 第1个数
第二次：left = 第1个数, right = 第2个数
......
第int(len/2)+1次：left = 第(len/2)个数, right = 第(len/2+1)个数
```

在循环中利用`aStart`和`bStart`分别表示当前指向`nums1[]`和`nums2[]`数组的位置。在循环中：

- 当`nums1[aStart]`小于等于`nums2[bStart]`的时候，`aStart`加1往后移；
- 当`nums1[aStart]`大于`nums2[bStart]`的时候，`bStart`加1往后移；

现在应当考虑数组越界的问题，首先分析`aStart`加1往后移的情况，此时必须有`aStart`是小于数组`nums1[]`的长度的，可以是在`nums1[aStart]`小于等于`nums2[bStart]`的情况下，也可以是在`bStart`超出了`nums2[]`的范围时：

```java
if(aStart < nums1.length && (bStart >= nums2.length || nums1[aStart] < nums2[bStart])
```

**这里的`bStart`越界条件必须放在两数组当前头元素比较大小之前，利用短路规避掉数组`nums2[]`为空的情况，否则会出现`nums2[0]`数组越界的情况**

那么对立的情况即是将`bStart`加1往后移。

**代码**

```java
class Solution {
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int n = nums1.length;
        int m = nums2.length;
        int left = -1, right = -1;
        int aStart = 0, bStart = 0;
        int len = n + m;
        for(int i = 0; i < len / 2 + 1; i++) {
            left = right;
            if(aStart < n && (bStart >= m || nums1[aStart] <= nums2[bStart])) {
                right = nums1[aStart++];
            } else {
                right = nums2[bStart++];
            }
        }
        if(len % 2 == 0){
            return (left + right) / 2.0;
        } else {
            return right;
        }    
    }
}
```

时间复杂度：遍历`len/2+1`次，`len=m+n`，所以时间复杂度是*O(m+n)*。

## 二分查找

暴力法时间复杂度达不到题目的要求$O(log(m+n))$，看到$log$应该想到利用二分查找才能达到。于是我们把题目中的求中位数，转换成为**求第k小数**的一种特殊情况。

暴力的时候，我们一次遍历就是在排除去掉一个不可能是中位数的一个值，由于数列是有序的，所以我们可以一半一半的排除。

现假设我们要找第`k`小数，每次循环排除掉`k/2`个数。下面看一个例子，假设要找第7小的数字。

- $k=7,k/2=3$
- 数组`A`：(1,3,**4**),9
- 数组`B`：(1,2,**3**),4,5,6,7,8,9,10

比较两个数组的第`k/2`个数字（向下取整），也就是比较第**3**个数字，数组`A`中的**4**和数组`B`中的**3**，因为**3**比**4**小，所以可以表明数组`B`的前`k/2`个数字都不是第`k`个数字。也就是说**1,2,3**这三个数字不可能是第**7**小的数字，我们可以将它们给排除掉。然后将数组`A`和排除掉前`k/2`个数字的数组`B`重新比较，如下：

- $k=4,k/2=2$
- 数组`A`：(1,**3**),4,9
- 数组`B`：(4,**5**),6,7,8,9,10

此时已经去掉了**3**个数字，那么此时$4=k=k-3$，再次比较两个数组的第`k/2`个数字，**3**比**5**小，所以表明数组`A`的前`k/2`个数字都不是第`k`个数字，即数字**1,3**不可能是当前数列的第4小数字，可以排除掉，剩余如下：

- $k=2,k/2=1$
- 数组`A`：(**4**),9
- 数组`B`：(**4**),5,6,7,8,9,10

此时更新$2=k=k-2$，再次比较两个数组的第`k/2`个数字，**4**和**4**相等，无论去掉哪个数组中的都行，没有影响。所以此时将数组`B`的**4**去掉。

- $k=1$
- 数组`A`：(**4**),9
- 数组`B`：(**5**),6,7,8,9,10

由于又去掉了一个数字，此时我们要找第1小的数字，只需要判断两个数组中第一个数字哪个小就可以了，即**4**。



更一般情况，对于`A[1]`，`A[2]`，`A[3]`，...，`A[k/2]`，...，`B[1]`，`B[2]`，`B[3]`，...，`B[k/2]`，...，如果`A[k/2]`$<$`B[k/2]`，那么`A[1]`，`A[2]`，`A[3]`，...，`A[k/2]`都不可能是第`k`小的数字。



**特殊情况**

每次都取`k/2`的数进行比较，当数组`A`的长度小于`k/2`的时候，我们应该做如下判断：

- $k=7,k/2=3$
- 数组`A`：(1,**2**)
- 数组`B`：(1,2,**3**),4,5,6,7,8,9,10

此时数组`A`的长度是2，而`k/2=3`，我们将此时应取数组`A`的末尾就可以了，然后进行比较，由于**2**小于**3**，导致数组`A`的1,2都被排除，结果如下：

- $k=5$
- 数组`A`：
- 数组`B`：1,2,3,4,5,6,7,8,9,10

由于数组`A`的所有元素都被排除，此时$k=5$，我们只需要返回下边数组的第**5**个数字就好了。

**总结**

- 设`(n+m)`为两个数组合起来的长度，那么不论奇数还是偶数，我们**求第k小的数字**都可以通过计算第$(n+m+1)/2$和$(n+m+2)/2$个数字来解决。
  - 当`(n+m)`是偶数时，两数相加除以2即可得到中位数；
  - 当`(n+m)`是奇数时，计算了两次中位数，也应该是两数相加除以2。
- 当`nums1[k/2]`小于`nums2[k/2]`时，丢掉`nums1[]`的前`k/2`个数字；
- 当`nums1[k/2]`大于等于`nums2[k/2]`时，丢掉`nums2[]`的前`k/2`个数字；
- 当其中一个数组的长度为**0**时，只需要求另一个数组的第**k**小的数字；
- 当$k=1$时，求两个数组第一个数字中较小的那个；
- 当`k/2`超过数组长度时，取数组最后一个数字即可。

**代码**

```java
class Solution {

        public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int n = nums1.length;
        int m = nums2.length;
        int left = (n + m + 1) / 2;
        int right = (n + m + 2) / 2;
        // 当n+m为偶数时，应该计算 (left+right)/2
        // 当n+m为奇数时，计算了两次中位数，也应该是 (left+right)/2
        // 于是不论是奇数还是偶数，都可以用一套代码解决
        return (getKth(nums1, 0, n - 1, nums2, 0, m - 1, left) + getKth(nums1, 0, n - 1, nums2, 0, m - 1, right)) / 2.0;
    }

    public static int getKth(int[] nums1, int start1, int end1, int[] nums2, int start2, int end2, int k) {
        int len1 = end1 - start1 + 1;
        int len2 = end2 - start2 + 1;
        
        // 确保 nums1[] 一定比 nums2[] 短
        // 当有空的数组出现时，一定是 nums1[]
        if(len1 > len2) {
            return getKth(nums2, start2, end2, nums1, start1, end1, k);
        }
        if(len1 == 0) {
            return nums2[start2 + k - 1];
        }

        if (k == 1) return Math.min(nums1[start1], nums2[start2]);

        int i = start1 + Math.min(len1, k / 2) - 1;
        int j = start2 + Math.min(len2, k / 2) - 1;

        if(nums1[i] < nums2[j]) {
            return getKth(nums1, i + 1, end1, nums2, start2, end2, k - (i - start1 + 1));
        } else {
            return getKth(nums1, start1, end1, nums2, j + 1, end2, k - (j - start2 + 1));
        }
    }
}
```

