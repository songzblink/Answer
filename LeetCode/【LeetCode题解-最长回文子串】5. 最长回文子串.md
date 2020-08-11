**题目描述：**给定一个字符串 `s`，找到 `s` 中最长的回文子串。你可以假设 `s` 的最大长度为 1000。

**示例1：**

```c++
输入: "babad"
输出: "bab"
注意: "aba" 也是一个有效答案。
```

**示例2：**

```c++
输入: "cbbd"
输出: "bb"
```

**解析：**

【暴力】

- 根据回文子串的定义，枚举所有长度大于等于 2 的子串，依次判断它们是否是回文；
- 在具体实现时，可以只针对大于“当前得到的最长回文子串长度”的子串进行“回文验证”；
- 在记录最长回文子串的时候，可以只记录“当前子串的起始位置”和“子串长度”，不必做截取。这一步我们放在后面的方法中实现。



```java
class Solution {
    public String longestPalindrome(String s) {
        int len = s.length();
        if (len < 2) {
            return s;
        }

        int maxLen = 1;
        int begin = 0;
        // s.charAt(i) 每次都会检查数组下标越界，因此先转换成字符数组
        char[] charArray = s.toCharArray();

        // 枚举所有长度大于 1 的子串 charArray[i..j]
        for (int i = 0; i < len - 1; i++) {
            for (int j = i + 1; j < len; j++) {
                if (j - i + 1 > maxLen && validPalindromic(charArray, i, j)) {
                    maxLen = j - i + 1;
                    begin = i;
                }
            }
        }
        return s.substring(begin, begin + maxLen);
    }

    /**
    * 判断字符串str是否是回文串
    */
    static boolean validPalindromic(char[] str, int left, int right) {
        while (left < right && str[left] == str[right]){
            left++;
            right--;
        }
        if (left < right) {
            return false;
        } else {
            return true;
        }
    }   
}
```

【动态规划】

```java
class Solution {
    public String longestPalindrome(String s) {
        // 回文子串的首尾字符一定相同
        // 设dp[i][j] = true 表示以i开头和j结尾的子串是回文，false则表示不是回文
        // dp[i][j] = dp[i+1][j-1] and s[i] == s[j]
        // 边界条件 i+1和j-1不能构成区间，即长度小于2，(j-1)-(i+1)+1<2，整理得j-i<3
        // 初始条件dp[i][i] = true
        int len = s.length();
        if(len < 2) return s; // 特判
        boolean [][] dp = new boolean[len][len];
        char [] sc = s.toCharArray();
        int maxLen = 1;
        int start = 0;
        for(int i = 0; i < len; i++) {
            dp[i][i] = true;
        }
        // 填表，只需要填上半边表就行，因为i始终小于等于j的
        for(int j = 1; j < len; j++) {
            for(int i = 0; i < j; i++) {
                if(sc[i] == sc[j]) {
                    if(j - i < 3) {
                        dp[i][j] = true;
                    } else {
                        dp[i][j] = dp[i+1][j-1];
                    }
                } else {
                    dp[i][j] = false;
                }
                if(dp[i][j] == true && j - i + 1 > maxLen) { // 更新，记录开始位置和长度
                    maxLen = j - i + 1;
                    start = i;
                }
            }
        }
        return s.substring(start, start + maxLen);
    }
}
```

