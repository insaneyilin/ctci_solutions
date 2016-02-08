# 栈和队列

基本要求，能够手写栈和队列的实现（基于数组/链表）。

---

## 题目 3.1

> Describe how you could use a single array to implement three stacks.

## 解答

> 参考：
> http://www.hawstein.com/posts/3.1.html

> 我们可以很容易地用一个数组来实现一个栈，压栈就往数组里插入值，栈顶指针加1； 出栈就直接将栈顶指针减1；取栈顶值就把栈顶指针指向的单元的值返回； 判断是否为空就直接看栈顶指针是否为-1。

> 如果要在一个数组里实现3个栈，可以将该数组分为3个部分。如果我们并不知道哪个栈将装 入更多的数据，就直接将这个数组平均分为3个部分，每个部分维护一个栈顶指针， 根据具体是对哪个栈进行操作，用栈顶指针去加上相应的偏移量即可。

```cpp
class stack3
{
public:
    stack3(int size = 300) {
        buf = new int[size * 3];
        ptop[0] = ptop[1] = ptop[2] = -1;
        this->size = size;
    }

    ~stack3() {
        delete[] buf;
    }

    void push(int stack_num, int val) {
        int idx = stack_num * size + ptop[stack_num] + 1;
        buf[idx] = val;
        ++(ptop[stack_num]);
    }

    void pop(int stack_num) {
        --(ptop[stack_num]);
    }

    int top(int stack_num) {
        int idx = stack_num * size + ptop[stack_num];
        return buf[idx];
    }

    bool empty(int stack_num) {
        return ptop[stack_num] == -1;
    }

private:
    int *buf;     // 数组
    int ptop[3];  // 三个栈的栈顶指针
    int size;     // 每个栈的大小
};
```

> 如果不要求三个栈的大小相等，则数组不分段，无论是哪个栈入栈，都依次地往这个数组里存放。这样一来，我们除了维护每个栈的栈顶指针外，我们还需要维护每个栈中， 每个元素指向前一个元素的指针。这样一来，某个栈栈顶元素出栈后，它就能正确地找到下一个栈顶元素。

```cpp
struct node {
    int val;
    int pre_idx;  // 当前节点所在的栈中上一个节点的索引
};

class stack3
{
public:
    stack3(int total_size = 900) {
        buf = new node[total_size];
        ptop[0] = ptop[1] = ptop[2] = -1;
        this->total_size = total_size;
        cur = 0;
    }

    ~stack3() {
        delete[] buf;
    }

    void push(int stack_num, int val) {
        if (cur > this->total_size)
        {
            cout << "stack is full\n";
            return;
        }

        buf[cur].val = val;
        buf[cur].pre_idx = ptop[stack_num];
        ptop[stack_num] = cur;
        ++cur;
    }

    void pop(int stack_num) {
        if (cur <= 0)
        {
            cout << "stack is full\n";
            return;
        }

        ptop[stack_num] = buf[ptop[stack_num]].pre_idx;
    }

    int top(int stack_num) {
        return buf[ptop[stack_num]].val;
    }

    bool empty(int stack_num) {
        return ptop[stack_num] == -1;
    }

private:
    node *buf;     // 数组
    int ptop[3];  // 三个栈的栈顶指针
    int total_size;  // 数组总大小
    int cur;
};
```

---

## 题目 3.2

> How would you design a stack which, in addition to push and pop, also has a function min which returns the minimum element? Push, pop and min should all operate in O(1) time.

## 解答

用一个数据栈保存数据，一个辅助栈保存阶段性最小值。

push 时，如果入栈元素不大于(<=)辅助栈的栈顶元素，则同时 push 进辅助栈；pop 时，如果数据栈栈顶元素等于辅助栈栈顶元素，辅助栈的栈顶元素也出栈。这样保证辅助栈的栈顶一定是当前数据栈中的最小元素值。

注意上面入栈时用 <= 进行判断，这是考虑到可能有多个相等的最小元素存在。

```cpp
class MinStack {
public:
    MinStack() {
        // do initialization if necessary
    }

    void push(int number) {
        // write your code here
        data_.push(number);
        
        if (min_vals_.empty() || number <= min_vals_.top()) {
            min_vals_.push(number);
        }
    }

    int pop() {
        // write your code here
        int ret = data_.top();
        data_.pop();
        if (ret == min_vals_.top()) {
            min_vals_.pop();
        }
        
        return ret;
    }

    int min() {
        // write your code here
        return min_vals_.top();
    }
    
private:
    stack<int> data_;
    stack<int> min_vals_;
};

```

---

## 题目 3.3

> Imagine a (literal) stack of plates. If the stack gets too high, it might topple. Therefore, in real life, we would likely start a new stack when the previous stack exceeds some threshold. Implement a data structure SetOfStacks that mimics this. SetOfStacks should be composed of several stacks, and should create a new stack once the previous one exceeds capacity. SetOfStacks.push() and SetOfStacks.pop() should behave identically to a single stack (that is, pop() should return the same values as it would if there were just a single stack).

> **FOLLOW UP**

> Implement a function popAt(int index) which performs a pop operation on a specific sub-stack.

## 解答

参考：

> http://www.hawstein.com/posts/3.3.html

首先不考虑 `popAt`，`SetOfStacks` 由栈的数组构成，我们需要一个指向当前栈的变量cur， 每当执行push操作时，我们需要检查一下当前栈是否已经达到其容量了， 如果是的话，就要将cur加1，指向下一个栈。而执行pop操作时， 需要先检查当前栈是否为空，如果是，则cur减1，移向上一个栈。top操作同理。 这时候，SetOfStacks可以想象成把一个本来可以叠得很高的栈，分成了好几个子栈。 push和pop操作其实都只是在“最后”一个子栈上操作。

```cpp
class SetOfStacks  // without popAt()
{
public:
    SetOfStacks(int num_stacks = 5, int capacity = 20)
    {
        this->capacity = capacity;
        this->num_stacks = num_stacks;
        cur = 0;
        stacks = new stack<int>[num_stacks];
    }

    ~SetOfStacks()
    {
        delete[] stacks;
    }

    void push(int val)
    {
        if (full())
        {
            cout << "No extra space, cannot push.\n";
            return;
        }

        if (stacks[cur].size() == capacity)
            ++cur;
        stacks[cur].push(val);
    }

    void pop()
    {
        if (empty())
        {
            cout << "Stack is empty, cannot pop.\n";
            return;
        }

        if (stacks[cur].empty())
            --cur;
        stacks[cur].pop();
    }

    int top()
    {
        if (empty())
        {
            cout << "Stack is empty.\n";
            return;
        }

        if (stacks[cur].empty())
            --cur;
        return stacks[cur].top();
    }

    bool empty()
    {
        if (cur == 0)
            return stacks[0].empty();

        return false;
    }

    bool full()
    {
        if (cur == num_stacks - 1)
            return stacks[cur].size() == capacity;

        return false;
    }

private:
    stack<int> *stacks;  // 栈的数组
    int cur;    // 指向当前操作的栈
    int capacity;  // 每个栈的容量
    int num_stacks;  // 栈的数量
};
```

当加入popAt函数，情况就变得复杂了。因为这时候的数据分布可能出现中间的某些子栈使 用popAt把它们清空了，而后面的子栈却有数据。 **为了实现方便，我们不考虑因为popAt 带来的空间浪费。** 即如果我用popAt把中间某些子栈清空了，并不把后面子栈的数据往前移动。这样一来，cur指向操作的“最后”一个栈，它后面的子栈一定都是空的， 而它本身及前面的子栈由于popAt函数的缘故都有可能是空的。如果没有popAt函数， cur前面的子栈一定都是满的(见上面的例子)。这样一来，push仍然只需要判断一次当前子 栈是否为满。但是，pop函数则要从cur向前一直寻找，直到找到一个非空的子栈， 才能进行pop操作。同理，popAt，top，empty也是一样的。

（这种实现浪费了空间，即虽然中间有些栈是空的，但无法再向栈中 push 元素。看了下 ctci 在 github 上的解答，也没有考虑这个问题。）

---

## 题目 3.4

经典的汉诺塔问题。用栈来解决（不要用递归）。

## 解答

看了下 ctci 官方题解，这题并没有说不能用递归，只是用栈来模拟，这就太简单了：

```cpp
void tower_of_hanoi(stack<int> &src, stack<int> &buf, stack<int> &dst, int n) {
    if (n == 0)
        return;

    tower_of_hanoi(src, dst, buf, n - 1);
    dst.push(src.top());
    src.pop();
    tower_of_hanoi(buf, src, dst, n - 1);
}
```

如果不用递归，怎么实现？显式地利用栈来模拟递归过程中状态参数的压栈、出栈过程。

定义一个数据结构保存操作过程中的参数：

```cpp
struct op 
{
    int begin, end;
    stack<int> *src, *buf, *dst;
    op() 
    {
        begin = 0;
        end = 0;
        src = NULL;
        buf = NULL;
        dst = NULL;
    }
    op(int begin, int end, 
        stack<int> *src, stack<int> *buf, stack<int> *dst) 
    {
        this->begin = begin;
        this->end = end;
        this->src = src;
        this->buf = buf;
        this->dst = dst;
    }
};
```

这五个参数表示，柱子 src 上有编号为 begin 到 end 的圆盘，借助柱子 buf 移动到 柱子 dst 上。当操作参数栈不为空时，不断出栈，判断操作数的 begin 和 end 是否相等。若相等则表示剩下当前问题规模下的“最后”一个圆盘，进行移动；若不相等，向操作参数栈中 push 三个操作数（顺序与执行顺序相反，原理同递归实现，这样出栈时执行顺序才会正确）。

非递归求解汉诺塔的完整代码：

```cpp
struct op 
{
    int begin, end;
    stack<int> *src, *buf, *dst;
    op() 
    {
        begin = 0;
        end = 0;
        src = NULL;
        buf = NULL;
        dst = NULL;
    }
    op(int begin, int end, 
        stack<int> *src, stack<int> *buf, stack<int> *dst) 
    {
        this->begin = begin;
        this->end = end;
        this->src = src;
        this->buf = buf;
        this->dst = dst;
    }
};

void tower_of_hanoi_iter(stack<int> &src, stack<int> &buf, stack<int> &dst, int n) {
    if (n == 0)
        return;

    stack<op> op_stack;
    op tmp;
    op_stack.push(op(1, n, &src, &buf, &dst));
    
    while (!op_stack.empty())
    {
        tmp = op_stack.top();
        op_stack.pop();
        if (tmp.begin != tmp.end)
        {
            op_stack.push(op(tmp.begin, tmp.end - 1, tmp.buf, tmp.src, tmp.dst));
            op_stack.push(op(tmp.end, tmp.end, tmp.src, tmp.buf, tmp.dst));
            op_stack.push(op(tmp.begin, tmp.end - 1, tmp.src, tmp.dst, tmp.buf));
        }
        else
        {
            tmp.dst->push(tmp.src->top());
            tmp.src->pop();
        }
    }
}

void tower_of_hanoi(stack<int> &src, stack<int> &buf, stack<int> &dst, int n) {
    if (n == 0)
        return;

    tower_of_hanoi(src, dst, buf, n - 1);
    dst.push(src.top());
    src.pop();
    tower_of_hanoi(buf, src, dst, n - 1);
}

void print_hanoi(stack<int> src, stack<int> buf, stack<int> dst)
{
    cout << "src: ";
    while (!src.empty())
    {
        cout << src.top() << " ";
        src.pop();
    }
    cout << endl;

    cout << "buf: ";
    while (!buf.empty())
    {
        cout << buf.top() << " ";
        buf.pop();
    }
    cout << endl;

    cout << "dst: ";
    while (!dst.empty())
    {
        cout << dst.top() << " ";
        dst.pop();
    }
    cout << endl << endl;
}

int main()
{
    int n = 3;
    stack<int> src, buf, dst;

    for (int i = n; i >= 1; --i)
    {
        src.push(i);
    }

    print_hanoi(src, buf, dst);

    tower_of_hanoi_iter(src, buf, dst, n);
    print_hanoi(src, buf, dst);

    system("pause");
    return 0;
}
```

---

## 题目 3.5 

> Implement a MyQueue class which implements a queue using two stacks.

## 解答

用两个栈实现一个队列。

第一个栈用于入队，第二个栈用于出队。

入队操作：压入 stack1。

出队操作：判断 stack2 是否为空。非空，则返回 stack2 栈顶；为空，则将 stack1 中元素依次出队压入 stack2 中，返回 stack2 栈顶。

```cpp
class MyQueue 
{
public:
    void push(int node) {
        stack1.push(node);
    }

    int pop() {
        if (stack2.empty()) {
            while (!stack1.empty()) {
                stack2.push(stack1.top());
                stack1.pop();
            }
        }
        
        int top = stack2.top();
        stack2.pop();
        
        return top;
    }

private:
    stack<int> stack1;
    stack<int> stack2;
};
```

---

## 题目 3.6

> Write a program to sort a stack in ascending order. You should not make any assumptions about how the stack is implemented. You may use additional stacks to hold items, but you may not copy the elements into any other data structure(such as an array). The following are the only functions that should be used to write this program: push | pop | peek | isEmpty.

> 请编写一个程序，按升序对栈进行排序（即最大元素位于栈顶），要求最多只能使用一个额外的栈存放临时数据，但不得将元素复制到别的数据结构中。

## 解答

借助另外一个有序栈来实现，保证该有序栈栈顶为最小元素。把下一个待插入的元素存放在一个临时变量 tmp 里，然后在有序栈里，把所有比 tmp 大的元素都 push 到无序栈中，保证当 tmp 压入有序栈时，所有在它下面的元素都比它小。当无序栈中所有元素都被转移到有序栈时，有序栈中的元素是从栈顶由小到大排列的。

```cpp
void sort_stack(stack<int> &st) {
    stack<int> sorted;  // top contains smallest element
    while (!st.empty())
    {
        int tmp = st.top();
        st.pop();
        while (!sorted.empty() && tmp > sorted.top())
        {
            st.push(sorted.top());
            sorted.pop();
        }

        sorted.push(tmp);
    }

    while (!sorted.empty())
    {
        st.push(sorted.top());
        sorted.pop();
    }
}
```

用递归也可以实现（看起来没有用另一个栈，实际上递归中使用了栈）。先看“栈转置”问题，用递归颠倒一个栈。例如输入栈{1, 2, 3, 4, 5}，1在栈顶。颠倒之后的栈为{5, 4, 3, 2, 1}，5处在栈顶。

递归定义：将当前栈的最底元素移到栈顶，其他元素顺次下移一位，然后继续处理不包含该栈顶元素的子栈。终止条件：递归下去，直到栈为空。

```cpp
// 将栈底元素移动到栈顶
void move_bottom_to_top(stack<int> &s) {
    if (s.empty())
        return;

    int top1 = s.top();
    s.pop();

    if (s.empty())
    {
        s.push(top1);
        return;
    }

    move_bottom_to_top(s);
    int top2 = s.top();
    s.pop();
    s.push(top1);
    s.push(top2);
}

// 转置整个栈
void reverse_stack(stack<int> &s) {
    if (s.empty())
        return;
    move_bottom_to_top(s);
    int top = s.top();
    s.pop();
    reverse_stack(s);  // 递归处理子栈
    s.push(top);
}
```

现在稍作修改，每次将最小（或最大）元素移动到栈顶：

```cpp
void move_min_to_top(stack<int> &s) {
    if (s.empty())
        return;

    int top1 = s.top();
    s.pop();

    if (!s.empty())
    {
        move_min_to_top(s);
        int top2 = s.top();
        if (top1 > top2)
        {
            s.pop();
            s.push(top1);
            s.push(top2);
            return;
        }
    }

    s.push(top1);
}

void sort_stack(stack<int> &s) {
    if (s.empty())
        return;

    move_min_to_top(s);
    int top = s.top();
    s.pop();

    sort_stack(s);
    s.push(top);
}
```

---

## 题目 3.7

> An animal shelter holds only dogs and cats, and operates ona strictly “first in, first out” basis. People must adopt either the “oldest”(based on arrival time) of all animals at the shelter, or they can select whether they could prefer a dog or a cat (and will receive the oldest animal ofthat type). They cannot select which specific animal they would like. Create the data structures to maintain this system and implement operations such a senqueue, dequeueAny, dequeueDog and dequeueCat. You may use the built-in LinkedList data structure.

> 有一个动物收养站，猫或狗都不停地被收养进来，同时也有人想领养。规则是领养人可以挑选猫或狗，但不能挑具体哪一只，只能挑选早被收养的那只动物。

## 解答

猫狗各自建一个队列，保存时间信息即可。

dequeueDog 和 dequeueCat 分别从各自队列出队；dequeueAny 判断两队队首哪个时间早再出队。

代码略。


