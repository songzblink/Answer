**题目描述：**将一个给定字符串根据给定的行数，以从上往下、从左到右进行 Z 字形排列。

比如输入字符串为 `"LEETCODEISHIRING"` 行数为 3 时，排列如下：

```
L   C   I   R
E T O E S I I G
E   D   H   N
```

之后，你的输出需要从左往右逐行读取，产生出一个新的字符串，比如：`"LCIRETOESIIGEDHN"`。

请你实现这个将字符串进行指定行数变换的函数：

```
string convert(string s, int numRows);
```

**示例1：**

```c++
输入: s = "LEETCODEISHIRING", numRows = 3
输出: "LCIRETOESIIGEDHN"
```

**示例2：**

```c++
输入: s = "LEETCODEISHIRING", numRows = 4
输出: "LDREOEIIECIHNTSG"
解释:

L     D     R
E   O E   I I
E C   I H   N
T     S     G
```

**解析：**

这种题没什么好说的，大多在考察编码能力，模拟这个行索引的变化，在遍历中把每个字符填到正确的行即可。

于是需要定义一个大小为 `numRows` 的 list 集合来存放每行的元素，利用 `flag` 来判断当前元素应该放入哪个 list 集合之中，代码如下：

```java
class Solution {
    public String convert(String s, int numRows) {
        if (numRows < 2) {
            return s;
        }
        ArrayList<Character>[] arrayLists = new ArrayList[numRows];
        for (int i = 0; i < numRows; i++) {
            arrayLists[i] = new ArrayList<Character>();
        }
        int flag = -1, row = 0;
        for (Character c : s.toCharArray()) {
            arrayLists[row].add(c);
            if (row == 0 || row == numRows - 1) {
                flag = -flag;
            }
            row += flag;
        }
        StringBuffer sb = new StringBuffer();
        for (int i = 0; i < numRows; i++) {
            Character[] str = new Character[arrayLists[i].size()];
            arrayLists[i].toArray(str);
            for (Character temp : str) {
                sb.append(temp);
            }
        }
        return sb.toString();
    }
}
```







