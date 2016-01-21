
## 题目

> Implement an algorithm to determine if a string has all unique characters. What if you can not use additional data structures?

> 判断字符串中的字符是否唯一，不用额外的数据结构（只用基本数据结构）。

## 解答

可以向面试官提的问题：

- 哪些字符？只有英文（26 个字母）还是所有 ACSII 码（256 个字符）；
- 不用额外数据结构什么意思？

假设字符集是ASCII字符，用一个 256 容量的 hash 表可以解决。或者利用位运算压缩空间，256 位可用 8 个 32 位的 `int` 保存。

```cpp
class Different {
public:
    bool checkDifferent(string iniString) {
        // write code here
        return is_unique_2(iniString);
    }

private:
    bool is_unique_1(string s)
    {
        int h[256] = { 0 };
        for (const char &ch : s)
        {
            unsigned int v = static_cast<unsigned int>(ch);
            if (h[v] == 1)
                return false;
            h[v] = 1;
        }

        return true;
    }

    bool is_unique_2(string s)
    {
        int h[8] = { 0 };
        for (const char &ch : s)
        {
            int v = (int)ch;
            int idx = v / 32, shift = v % 32;
            if (h[idx] & (1 << shift))
                return false;
            h[idx] |= (1 << shift);
        }

        return true;
    }
};
```