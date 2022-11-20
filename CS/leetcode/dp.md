# 1、最长回文子串

#### [5. 最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring/)

给你一个字符串 `s`，找到 `s` 中最长的回文子串。

```java
class Solution {
    public String longestPalindrome(String s) {
        int len = s.length();
        if(len == 0 || len == 1)
            return s;
        boolean[][] dp = new boolean[len] [len];
        char[] arr = s.toCharArray();
        int begin = 0;
        int end = 0;
        int t = 0;
        for(int i = 0;i < len;i++)
        {
            dp[i] [i] = true;
        }
        for(int l = 2;l < len;l++)
        {
            for(int i = 0;i < len-l+1;i++)
            {
                int j = i+l-1;
                if(l == 2)
                    dp[i] [j] = arr[i] == arr[j]? true:false;
                else
                    dp[i] [j] = (dp[i+1] [j-1] && arr[i] == arr[j])? true:false;
                if(dp[i] [j] && j-i+1 >t)
                {
                    begin = i;
                    end = j;
                    t = j-i+1;
                }
            }
        }
        return s.substring(begin,end+1);
    }
```

dp[i] [j] = (dp[i+1] [j-1] && arr[i] == arr[j])? true:false;

1、由于dp矩阵是由短到长判断，因此最外层循环必须以长度做变量

2、i+1 <= j-1 且 j=i+l-1，推导出当l >= 3时才符合一般方程，因此需要对l = 2做特殊处理

# 2、接雨水

#### [42. 接雨水](https://leetcode-cn.com/problems/trapping-rain-water/)

给定 *n* 个非负整数表示每个宽度为 1 的柱子的高度图，计算按此排列的柱子，下雨之后能接多少雨水。

```java
class Solution {
    public int trap(int[] height) {
        int len = height.length;
        int[] leftMax = new int[len];
        int[] rightMax = new int[len];
        int sum = 0;
        leftMax[0] = 0;
        rightMax[len-1] = 0;
        int temp = 0;
        for(int i=1;i<len;i++)
        {
            leftMax[i] = Math.max(leftMax[i-1],height[i-1]);
            rightMax[len-1-i] = Math.max(rightMax[len-i],height[len-i]);
        }
        for(int i=0;i<len;i++)
        {
            temp = Math.min(leftMax[i],rightMax[i])-height[i];
            //必须判断非负数才可累加
            if(temp > 0)
            	sum += temp;
        }
        return sum;
    }
```

1、每一处接到的雨水量等于此处向左最高值和向右最大值，两者取最小值，乘以此处宽度1，减去自身高度。

temp = Math.min(leftMax[i],rightMax[i])-height[i];

2、每一处的雨水量必须判断非负数，方可累加

3、左右两侧最大值数列采用动态规划计算。

