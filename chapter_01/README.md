
## 题目 1.1

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

---

## 题目 1.2

> Write code to reverse a C-Style String. (C-String means that “abcd” is represented as five characters, including the null character.)

## 解答

对于 C 的字符串：

```cpp
class Reverse {
public:
    void reverseString(char str[]) {
        if (!str)
            return;

        char tmp;
        char *p1 = str, *p2 = str;
        while (*p2)
            ++p2;
        --p2;
        while (p1 < p2)
        {
            tmp = *p1;
            *p1 = *p2;
            *p2 = tmp;

            ++p1, --p2;
        }
    }
};
```

对于 C++ 的 `string`：

```cpp
class Reverse {
public:
    string reverseString(string iniString) {
        // write code here
        int p1 = 0, p2 = iniString.length() - 1;
        while (p1 < p2)
        {
            std::swap(iniString[p1], iniString[p2]);
            ++p1, --p2;
        }

        return iniString;
    }
};
```

---

## 题目 1.3

> Given two strings, write a method to decide if one is a permutation of the other.

## 解答

### 解法1：排序

时间复杂度 O(n log(n))。

```cpp
class Same {
public:
    bool checkSam(string stringA, string stringB) {
        std::sort(stringA.begin(), stringA.end());
        std::sort(stringB.begin(), stringB.end());

        return stringA == stringB;
    }
};
```

### 解法2：Hashing

时间复杂度 O(n)，空间复杂度 O(n)。

```cpp
class Same {
public:
    bool checkSam(string stringA, string stringB) {
        if (stringA.length() != stringB.length())
            return false;
        std::map<char, int> h;

        for (const char &ch : stringA)
        {
            ++(h[ch]);
        }

        for (const char &ch : stringB)
        {
            if (h.find(ch) == h.end())
                return false;
            --(h[ch]);
        }

        for (const auto &pr : h)
        {
            if (pr.second != 0)
                return false;
        }

        return true;
    }
};
```

---

## 题目 1.4

> 请编写一个方法，将字符串中的空格全部替换为“%20”。假定该字符串有足够的空间存放新增的字符，并且知道字符串的真实长度(小于等于1000)，同时保证字符串由大小写的英文字母组成。

## 解答

先遍历一次字符串，得到空格个数 `num_spaces`，然后从后向前进行替换。（从前向后替换有很多字符会重复移动）

```cpp
class Replacement {
public:
    string replaceSpace(string iniString, int length) {
        int num_space = 0;
        for (int i = 0; i < length; ++i)
        {
            if (iniString[i] == ' ')
                ++num_space;
        }

        string res_string;
        res_string.resize(length + 2 * num_space);

        for (int i = length - 1; i >= 0; --i)
        {
            int j = i + 2 * num_space;
            res_string[j] = iniString[i];
            if (res_string[j] == ' ')
            {
                --num_space;
                res_string[j - 2] = '%';
                res_string[j - 1] = '2';
                res_string[j] = '0';
            }
        }

        return res_string;
    }
};
```

---

## 题目 1.5

> 利用字符重复出现的次数，编写一个方法，实现基本的字符串压缩功能。比如，字符串“aabcccccaaa”经压缩会变成“a2b1c5a3”。若压缩后的字符串没有变短，则返回原先的字符串。

## 解答

```cpp
class Zipper {
public:
    string zipString(string iniString) {
        if (iniString.empty())
            return string("");

        string res_string;
        res_string.reserve(3000);

        res_string += iniString[0];
        int cnt = 1;
        for (int i = 1; i < iniString.length(); ++i)
        {
            if (iniString[i] == iniString[i - 1])
            {
                ++cnt;
            }
            else
            {
                res_string += ('0' + cnt);
                res_string += iniString[i];
                cnt = 1;
            }
        }
        res_string += ('0' + cnt);

        return res_string.length() < iniString.length() ? 
            res_string : iniString;
    }
};
```

---

## 题目 1.6

> 有一副由NxN矩阵表示的图像，这里每个像素用一个int表示，请编写一个算法，在不占用额外内存空间的情况下(即不使用缓存矩阵)，将图像顺时针旋转90度。
给定一个NxN的矩阵，和矩阵的阶数N,请返回旋转后的NxN矩阵,保证N小于等于500，图像元素小于等于256。

## 解答

举例子：

```
1 2 3
4 5 6
7 8 9
```

顺时针旋转 90 度后：

```
7 4 1
8 5 2
9 6 3
```

原矩阵先沿主对角线翻转：

```
1 4 7
2 5 8
3 6 9
```

再沿竖直中线翻转：

```
3 6 9
2 5 8
1 4 7
```

代码：

```cpp
class Transform {
public:
    vector<vector<int> > transformImage(vector<vector<int> > mat, int n) {
        for (int i = 0; i < n; ++i)
        {
            for (int j = i + 1; j < n; ++j)
            {
                std::swap(mat[i][j], mat[j][i]);
            }
        }

        for (int i = 0; i < n; ++i)
        {
            for (int j = 0; j < n / 2; ++j)
            {
                std::swap(mat[i][j], mat[i][n - 1 - j]);
            }
        }

        return mat;
    }
};
```

---

## 题目 1.7

> 请编写一个算法，若MxN矩阵中某个元素为0，则将其所在的行与列清零。


## 解答

遍历一次矩阵，当遇到元素等于0时，记录下这个元素对应的行和列。 可以开一个行数组row和列数组col，当元素a[i][j]等于0时， 就把row[i]和col[j]置为true。第二次遍历矩阵时，当某个元素对应的行row[i] 或列col[j]被设置为true，说明该元素在需要被置0的行或列上，因此将它置0。

```cpp
class Clearer {
public:
    vector<vector<int> > clearZero(vector<vector<int> > mat, int n) {
        return clearZero(mat, n, n);
    }

private:
    vector<vector<int> > clearZero(vector<vector<int> > mat, int m, int n) {
        vector<int> row(m, 0), col(n, 0);

        for (int i = 0; i < m; ++i)
        {
            for (int j = 0; j < n; ++j)
            {
                if (mat[i][j] == 0)
                {
                    row[i] = 1;
                    col[j] = 1;
                }
            }
        }

        for (int i = 0; i < m; ++i)
        {
            for (int j = 0; j < n; ++j)
            {
                if (row[i] || col[j])
                {
                    mat[i][j] = 0;
                }
            }
        }

        return mat;
    }
};
```

---

## 题目 1.8

> 假设你有一个isSubstring函数，可以检测一个字符串是否是另一个字符串的子串。 给出字符串s1和s2，只使用一次isSubstring就能判断s2是否是s1的旋转字符串， 请写出代码。旋转字符串："waterbottle"是"erbottlewat"的旋转字符串。

## 解答

参考：http://www.hawstein.com/posts/1.8.html

暴力解法，直接对 s1 进行不断旋转，判断是否和 s2 相等。

s1 和 s2 长度相等，一般要利用 `isSubstring` 的情形是有一个较长的串和一个较短的串。如何得到一个较长的串？s1 + s2 不行，因为 s1 和 s2 都是 s1 + s2 的子串。那么 s1 + s1 呢。

```
s1 + s1 = waterbottlewaterbottle
```

> 很容易可以发现，s1+s1其实是把s1中每个字符都旋转了一遍，而同时保持原字符不动。 比如waterbottle向右旋转2个字条应该是：terbottlewa，但如果同时保持原字符不动， 我们得到的就是waterbottlewa，而terbottlewa一定是waterbottlewa的子串， 因为waterbottlewa只是在terbottlewa的基础上再加上一条原字符不动的限制。 因此s1+s1将包含s1的所有旋转字符串，如果s2是s1+s1的子串，自然也就是s1 的旋转字符串了。


C++ 中可以用 `string::find()` 作为 `isSubtring` 。

```cpp
class ReverseEqual {
public:
    bool checkReverseEqual(string s1, string s2) {
        return (s1 + s1).find(s2) != string::npos;
    }
};
```
