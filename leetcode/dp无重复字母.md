给定一个只由小写字母(a~z)组成的字符串str，返回其中最长无重复字符串的子串长度。

思路：有两种情况
1、当前的字母位置i和上次出现此字母时位置的差
2、上一个位置i-1最长的位置
```java
class Solution {
    public static int  longestUnrepeatedString(String s) {
        char[] str = s.toCharArray();
        int[] last = new int[26];
        for(int i = 0; i < 26; i++) {
          last[i] = -1;
        }
        last[str[0] - 'a'] = 0;
        int maxLocation = 1; //因为第一个已经算了
        int maxCurrent = 1;
        for(int i = 1; i < str.length; i++) {
          // i - last[str[i] - 'a'] 这个为当前的字母位置i和上次出现此字母时位置的差
          // maxCurrent + 1 这个为上一个位置i-1最长的位置
          maxCurrent = Math.min(i - last[str[i] - 'a'], maxCurrent + 1);
          maxLocation = Math.max(maxLocation,maxCurrent);
          last[str[i] - 'a'] = i;
        }
        return maxLocation;
    }
}
```