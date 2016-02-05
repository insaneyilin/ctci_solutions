
## 题目 2.1

> Write code to remove duplicates from an unsorted linked list.

> FOLLOW UP

> How would you solve this problem if a temporary buffer is not allowed?

## 解答

解法1：用 Hash 表，时间复杂度 O(n)，空间复杂度 O(n)。

```cpp
void remove_duplicates_1(ListNode *head)
{
    if (!head)
        return;

    std::unordered_map<int, int> h;
    ListNode *p = head;
    h[p->val] = 1;
    while (p && p->next)
    {
        int val = p->next->val;
        if (h.find(val) == h.end())
        {
            h[val] = 1;
            p = p->next;
        }
        else
        {
            ListNode *tmp = p->next;
            p->next = tmp->next;
            delete tmp;
        }
    }
}
```

解法2：两个指针，两层循环，时间复杂度 O(n^2)，空间复杂度 O(1)

```cpp
void remove_duplicates_2(ListNode *head)
{
    if (!head)
        return;

    for (ListNode *p1 = head; p1 != NULL; p1 = p1->next)
    {
        for (ListNode *p2 = p1; p2 && p2->next; )
        {
            if (p2->next->val == p1->val)
            {
                ListNode *tmp = p2->next;
                p2->next = tmp->next;
                delete tmp;
            }
            else
            {
                p2 = p2->next;
            }
        }
    }
}
```

---

## 题目 2.2

> 输入一个(单)链表，输出该链表中倒数第k个结点。

## 解答

设置两个指针，第一个指针先走 k 步，然后两个指针一起走。第一个指针走到尾部的 NULL 时，第二个指针就指向倒数第 k 个节点。

```cpp
class Solution {
public:
    ListNode* FindKthToTail(ListNode* pListHead, unsigned int k) {
        if (!pListHead)
            return NULL;

        ListNode *p1 = pListHead, *p2 = pListHead;
        for (int i = 0; i < k; ++i)
        {
            if (p1 == NULL)
                return NULL;
            p1 = p1->next;
        }

        while (p1 != NULL)
        {
            p1 = p1->next;
            p2 = p2->next;
        }

        return p2;
    }
};
```

---

## 题目 2.3

> 实现一个算法，删除单向链表中间的某个结点，假定你只能访问该结点（即不能访问到其前驱结点）。给定带删除的节点，请执行删除操作，若该节点为尾节点，返回false，否则返回true

## 解答

不能访问前驱结点怎么删除当前节点。可以先交换当前节点和其后继节点的值，然后删除后继节点即可。但这种方法对尾节点不适用（尾节点的后继节点是 NULL）。关于尾节点的情况在面试中一定要主动指出来。

```cpp
class Remove {
public:
    bool removeNode(ListNode* pNode) {
        if (pNode->next == NULL)
            return false;

        ListNode *tmp = pNode->next;
        pNode->val = tmp->val;
        pNode->next = tmp->next;

        delete tmp;

        return true;
    }
};
```

---

## 题目 2.4

> 编写代码，以给定值x为基准将链表分割成两部分，所有小于x的结点排在大于或等于x的结点之前
给定一个链表的头指针 ListNode* pHead，请返回重新排列后的链表的头指针。注意：分割以后保持原来的数据顺序不变。

## 解答

向面试官提问：

- 是否有重复元素？
- 链表中是否一定会出现 x ？
- 如果链表中有 x ，是分两段还是三段？

```
3 1 4 5 7 5 2

取 5 为pivot，如果分两段，结果是：
3 1 4 2 | 5 7 5

分三段，结果是：
3 1 4 2 | 5 5 | 7

```

分两段的代码：

```cpp
class Partition {
public:
    ListNode* partition(ListNode* pHead, int x) {
        if (!pHead)
            return NULL;

        ListNode dummy_small(-1), dummy_big(-1);
        ListNode *p_small = &dummy_small, *p_big = &dummy_big;

        while (pHead != NULL)
        {
            if (pHead->val < x)
            {
                p_small->next = pHead;
                p_small = p_small->next;
            }
            else
            {
                p_big->next = pHead;
                p_big = p_big->next;
            }
            pHead = pHead->next;
        }
        
        p_small->next = dummy_big.next;
        p_big->next = NULL;

        return dummy_small.next;
    }
};
```

分三段：

```cpp
class Partition {
public:
    ListNode* partition(ListNode* pHead, int x) {
        if (!pHead)
            return NULL;

        ListNode dummy_small(-1), dummy_big(-1), dummy_x(-1);
        ListNode *p_small = &dummy_small, *p_big = &dummy_big, *p_x = &dummy_x;

        while (pHead != NULL)
        {
            if (pHead->val < x)
            {
                p_small->next = pHead;
                p_small = p_small->next;
            }
            else if (pHead->val > x)
            {
                p_big->next = pHead;
                p_big = p_big->next;
            }
            else
            {
                p_x->next = pHead;
                p_x = p_x->next;
            }
            pHead = pHead->next;
        }

        // x 不在原链表中出现的情况
        if (dummy_x.next == NULL)
        {
            p_small->next = dummy_big.next;
            p_big->next = NULL;
        }
        else
        {
            p_small->next = dummy_x.next;
            p_x->next = dummy_big.next;
            p_big->next = NULL;
        }

        return dummy_small.next;
    }
};
```

---

## 题目 2.5

> You have two numbers represented by a linked list, where each node contains a single digit. The digits are stored in reverse order, such that the 1’s digit is at the head of the list. Write a function that adds the two numbers and returns the sum as a linked list.

> EXAMPLE

> Input: (3 -> 1 -> 5), (5 -> 9 -> 2)

> Output: 8 -> 0 -> 8

> FOLLOW UP

> Suppose the digits are stored in forward order. Repeat the problem.

## 解答

从最低位逐位相加，保留进位，即可。

```cpp
/*
struct ListNode {
    int val;
    struct ListNode *next;
    ListNode(int x) : val(x), next(NULL) {}
};*/
class Plus {
public:
    ListNode* plusAB(ListNode* a, ListNode* b) {
        // write code here
        if (!a) return b;
        if (!b) return a;
        
        ListNode sum_head(-1);
        ListNode *p_sum = &sum_head;
        
        int carry = 0;
        while (a && b) {
            int sum = a->val + b->val + carry;
            carry = sum / 10;
            sum %= 10;
            ListNode *tmp = new ListNode(sum);
            p_sum->next = tmp;
            p_sum = p_sum->next;
            
            a = a->next;
            b = b->next;
        }
        
        if (a) {
            p_sum->next = a;
        }
        
        if (b) {
            p_sum->next = b;
        }
        
        while (carry) {
            if (!p_sum->next) {
                p_sum->next = new ListNode(carry);
                break;
            }
            
            int s = p_sum->next->val + carry;
            carry = s / 10;
            s %= 10;
            p_sum->next->val = s;
            p_sum = p_sum->next;
        }
        
        return sum_head.next;
    }
};
```

容易写错的地方：

- 循环条件是 `while (a && b)`，我经常脑残写成 `while (!a && !b)`
- 最后检查是否还有进位的 `while` 循环里，注意要处理的是 `p_sum->next`，这样才能够在链表尾部新加节点。

FOLLOW UP ，现在数字正序排列，怎么处理进位？不需要想太多，先给出最直接的解法，将链表逆序（O(n) 方法），再使用上面的方法即可。

---

## 题目 2.6

> 链表中有环，实现一个算法返回这个环的开始节点。

> EXAMPLE

> Input: A -> B -> C -> D -> E -> C [the same C as earlier]

> Output: C

## 解答


考察了 Floyd 判环算法，该算法可以判断链表中是否有环、确定环的起点和环的长度。

参考：

> http://blog.csdn.net/thestoryofsnow/article/details/6822576

本题解法：设置两个指针，都从表头出发，慢指针每次走一步，快指针每次走两步。若链表中无环，则快指针会先走到空指针。若有环，当两指针相遇时，将快指针移到表头，与慢指针一起每次走一步，两指针再次相遇的位置即是环的起点。

```cpp
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        ListNode *p1 = head, *p2 = head;
        bool has_cycle = false;
        while (p2 && p2->next)
        {
            p1 = p1->next;
            p2 = p2->next->next;
            if (p1 == p2) {
                has_cycle = true;
                break;
            }
        }
        if (!has_cycle)
            return NULL;

        p2 = head;
        while (p1 && p2)
        {
            if (p1 == p2)
                return p1;
            p1 = p1->next;
            p2 = p2->next;
        }

        return NULL;
    }
};
```

---

## 题目 2.7

> 请编写一个函数，检查链表是否为回文。
给定一个链表ListNode* pHead，请返回一个bool，代表链表是否为回文。

## 解答

链表逆序，然后与原链表进行逐节点比较。时间复杂度 O(n)，空间复杂度 O(n)。

```cpp
/*
struct ListNode {
    int val;
    struct ListNode *next;
    ListNode(int x) : val(x), next(NULL) {}
};*/
class Palindrome {
public:
    bool isPalindrome(ListNode* pHead) {
        if (!pHead)
            return true;

        ListNode r_dummy(-1);
        ListNode *p = &r_dummy;
        
        ListNode *head = pHead;

        while (pHead)
        {
            ListNode *tmp = new ListNode(pHead->val);
            tmp->next = pHead->next;
            p->next = tmp;
            p = p->next;

            pHead = pHead->next;
        }
        
        ListNode *r_head = reverse(r_dummy.next);

        while (head && r_head)
        {
            if (head->val != r_head->val)
                return false;

            head = head->next;
            r_head = r_head->next;
        }

        return true;
    }

private:
    ListNode* reverse(ListNode *head) {
        if (!head)
            return NULL;

        ListNode *new_head = NULL;

        while (head)
        {
            ListNode *next = head->next;

            head->next = new_head;
            new_head = head;

            head = next;
        }

        return new_head;
    }
};
```



