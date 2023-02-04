简单记录一下 KMP 算法的一些基本要点，免得自己总是一遍一遍忘记再找半天。

## 核心思想
KMP 最核心的想法就是在匹配到不一致的字符后，并不是移一位，从头开始重新比较，而是通过构建一个 **“部分匹配表”**，记录 匹配到当前字符后，前缀匹配到的长度（可以省略不比的长度），然后跳过匹配的前缀，以加快比较。  

![部分匹配表实例](https://www.ruanyifeng.com/blogimg/asset/201305/bg2013050109.png)

这里放一张阮一峰老师[博客](https://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html)中的部分匹配表图片以帮助理解，假如匹配到最后一个字母 D 时匹配失败，最后一个匹配成功的字母 B 的部分匹配值为 2，也就是说前两个字母我们可以不用比较（因为被匹配的串开头前两个字母必为 `AB`），所以我们只需要移动 `6-2=4` 个字符即刻。

## 算法
### 部分匹配表的构建
```cpp
void cal_next(char *str, int *next, int len)
{
    next[0] = -1;//next[0]初始化为-1，-1表示不存在相同的最大前缀和最大后缀
    int k = -1;//k初始化为-1
    for (int q = 1; q <= len-1; q++)
    {
        while (k > -1 && str[k + 1] != str[q])//如果下一个不同，那么k就变成next[k]，注意next[k]是小于k的，无论k取任何值。
        {
            k = next[k];//往前回溯
        }
        if (str[k + 1] == str[q])//如果相同，k++
        {
            k = k + 1;
        }
        next[q] = k;//这个是把算的k的值（就是相同的最大前缀和最大后缀长）赋给next[q]
    }
}
```
### 匹配流程
```cpp
int KMP(char *str, int slen, char *ptr, int plen)
{
    int *next = new int[plen];
    cal_next(ptr, next, plen);//计算next数组
    int k = -1;
    for (int i = 0; i < slen; i++)
    {
        while (k >-1&& ptr[k + 1] != str[i])//ptr和str不匹配，且k>-1（表示ptr和str有部分匹配）
            k = next[k];//往前回溯
        if (ptr[k + 1] == str[i])
            k = k + 1;
        if (k == plen-1)//说明k移动到ptr的最末端
        {
            //cout << "在位置" << i-plen+1<< endl;
            //k = -1;//重新初始化，寻找下一个
            //i = i - plen + 1;//i定位到该位置，外层for循环i++可以继续找下一个（这里默认存在两个匹配字符串可以部分重叠），感谢评论中同学指出错误。
            return i-plen+1;//返回相应的位置
        }
    }
    return -1;  
}
```