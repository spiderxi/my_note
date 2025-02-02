# 1. 基本算法

## 1 位运算

_int 32 是如何存储的, 取值范围是多少?_

```cpp
存储: 1 * 符号位,  31 * 数值位
取值范围 -2^31 ~ 2^31-1
```

**_xor 运算的用法有哪些?_**

```
A xor A = 0
A xor 0 = A
A xor 1 = ~A
```

**_`x >> 1` 与 `x / 2` 的运算结果相同吗 ?_**

```
当x >= 0时相同, 否则运算结果可能不同
```

**_如何快速求出 `a^b` 的结果?_**

```
使用快速幂算法:

所以从低到高遍历b的比特位, 维护权重1 -> a -> a^2 -> a^4, 如果bi==1, 则ans += (bi*权重)
```

## 2 递归

**_递归问题的通用模板是怎样的?_**

```JAVA
        public void solve(当前状态) {
            if (满足结束条件) {
		相关处理操作
                return;
            }
            solve(当前状态后的A分支状态); // 递归搜索
            如果有必要, 进行回溯操作
            solve(当前状态后的B分支状态); // 递归搜索
        }

```

## 3 前缀和&差分数组

**_前缀和数组和差分数组的作用?_**

```
* 前缀和用来快速求出子数组的和
sum[i, j]  = preSum[j] - preSum[i-1]

* 差分数组用来快速记录子数组的变化
nums[i, j] += k 等价于 diff[i] += k, diff[j+1] -= k
```

```java
        public int[] getPreSum(int[] nums) {
            int n = nums.length;
            int[] preSum = new int[n + 1];
            for (int i = 0; i < n; i++) {
                preSum[i+1] = preSum[i] + nums[i];
            }
            return preSum;
        }

        public int[] getDiff(int[] nums) {
            int n = nums.length;
            int[] diff = new int[n];
            diff[0] = nums[0];
            for (int i = 1; i < n; i++) {
                diff[i] = nums[i] - nums[i-1];
            }
            return diff;
        }
```

## 4 二分法

**_整数的二分查找两种模板怎么写?_**

```java
    public int findMax(int[] arr, int condition) {
        int l = 初始值, r = 初始值;
        while (l < r) {
            计算中点m(考虑m==l或m==r的临界情况)
            if (中点不符合想要查找的条件) 移动端点
            else 移动端点
        }
        return 返回端点;
    }
```

| 查找类型         | l 初始值 | r 初始值     | 中点计算方式 | 返回端点 |
| ---------------- | -------- | ------------ | ------------ | -------- |
| 符合条件的最大值 | 0        | arr.length   | (l+r+1) >> 1 | r        |
| 符合条件的最小值 | -1       | arr.length-1 | (l+r) >> 1   | l        |

**_实数的二分查找模板怎么写?_**

```java
    public double find(int target) {
        double pricisoin = 1E-5;
        double l = 初始值, r = 初始值;
        while (l + pricisoin < r) {
            double m = (l+r) / 2;
            if (符合l左移条件) l = m;
            else r = m;
        }
        return l;
    }
```

**_解域二分法模板怎么写, 适合解决什么问题?_**

```
解域二分法使用场景:
* 要求解空间内求符合条件的最值时
* 解空间内呈两段单调性([0, m] 符合条件,  [m+1, r]不符合条件)
```

```java
    public int find(int target) {
        int l = 初始值, r = 初始值;
        while (l < r) {
            int m = (l + r) >> 1;
            if (!judge(m)) l = m + 1;
            else r = m;
        }
        return l;
    }
```

## 5 排序

**_常见的排序算法的时间复杂度是多少?_**

| 算法     | 时间复杂度                     | 最优时间复杂度 | 最差时间复杂度 |
| -------- | ------------------------------ | -------------- | -------------- |
| 冒泡排序 | `N^2`                          | `N`            | `N^2`          |
| 选择排序 | `N^2`                          | `N^2`          | `N^2`          |
| 归并排序 | `Nlog(N)`                      | `Nlog(N)`      | `Nlog(N)`      |
| 快速排序 | `Nlog(N)`                      | `Nlog(N)`      | `N^2`          |
| 基数排序 | `n*m, m为最大值的权重位的个数` | `n*m`          | `n*m`          |

**_如何动态维护中位数?_**

```
使用对顶堆(一个最大堆和一个最小堆), 两个堆的size相差不超过1

* 新增一个元素时, 根据两个堆的堆顶元素, 往某个堆中add元素
* 如果两个堆size差>1, 将size较大的堆的堆顶元素取出放到另一个堆中
```

**_如何求出序列逆序对的数量?_**

```
使用归并排序算法, 当合并两个有序区间时, 可以计算出逆序对个数
```

**_快速排序/快速查找模板怎么写?_**

```java
    public void quickSort(int[] arr, int l, int r) {
        if (l >= r) return;
        int lp = l, rp = r, pivot = arr[l];
        while (lp < rp) {
            while (lp < rp && arr[rp] >= pivot) rp--;
            while (lp < rp && arr[lp] <= pivot) lp++;
            if (lp < rp) {
                int t = arr[lp];
                arr[lp] = arr[rp];
                arr[rp] = t;
            }
        }
        arr[l] =  arr[lp];
        arr[lp] = pivot;
        quickSort(arr, l, lp-1);
        quickSort(arr, lp+1, r);
    }
```

# 2. 数据结构扩展

## 1. 栈和队列

**_单调队列/单调栈的核心思想是什么?_**

```
* 在从左往右遍历数组的过程中, 元素依次入队/入栈
* 元素入队/栈时, 弹出队列/栈中不可能的解, 避免不必要的搜索
```

**_如何实现优先队列(最小/大堆)?_**

**_什么是 Catlan 数?_**

## 2 字符串

**KMP 算法**

什么是 KMP 算法 ?

```
KMP算法可以求出字符串text[0~N]中是否包含子字符串pat[0~M], 以及子字符串pat的位置, 时间复杂度为O(M+N)
```

KMP 算法的步骤 ?

```
1. 根据pat字符串求出next数组
2. 使用双指针进行匹配, 由于使用next数组, 指向text当前匹配位置的指针在匹配过程中不会回溯
```

KMP 中 next 数组的含义 ?

```
next(i) = k 表示pat[0~i]这个字符串最大"对称度"为k, 如abcab中 前缀ab = 后缀ab, "对称度" = 2
```

KMP 算法伪代码

```java
    public int kmp(String text, String pat) {
        int[] next = getNext(pat);
        for (int i = 0, j = 0; i < text.length() && j < pat.length();) {
            if (text.charAt(i) == pat.charAt(j)) { // 匹配成功
                i++;
                j++;
                if (j == pat.length()) return i - pat.length();
            } else {
                if (j != 0) j = next[j - 1];
                if (j == 0) i++;
            }
        }
        return -1;
    }
```

next 数组的求法(KMP 算法的精华所在) ?

```
定义: j是next(i)的候选值 等价于 pat[0~j-1] = pat[i-j+1~i]
可以通过反证法证明: 如果j时next(i)的侯选值, 那么next(i)的候选值中小于j的最大候选值为next(j)

要求next(i), 即从大到小判断 (next(i-1)的候选值+1) 是否为next(i)
```

```java
    int[] getNext(String pat) {
        int[] next = new int[pat.length()];
        for (int i = 1; i < pat.length(); i++) {
            // next[i-1]为初始候选值
            int k = i - 1;
            // 如果pat.charAt(next[k]) != pat.charAt(i), next[k]更新为下一个更小的候选值
            while (k != 0 && pat.charAt(next[k]) != pat.charAt(i)) k = next[k];
            if (pat.charAt(next[k]) == pat.charAt(i)) next[i] = next[k] + 1;
        }
        return next;
    }
```

## 3 树

字典树的有什么用 ?

```
用于快速检索字符串集合中的某个字符串
```

字典树上基本的操作有哪些?

Huffman 树可以用来做什么?

$$
\sum W_i * L_i 最小的树, 其中W_i = 叶节点权重, L_i为根到叶路径的长度
$$

k 阶 Huffman 树构造方法 ?

```
1. 每次从森林中选择两个根节点值最小的树(假设根节点的值为x, y)合并为根节点值为x+y的新树
2. 初始森林中叶节点的数量n需要满足
```

$$
(n - 1) mod (k-1) == 0
$$

```
不满足时添加权重为负无穷的节点
```

什么是红黑树?

## 4 并查集

什么是并查集 ?

```
并查集动态维护一个森林, 森林中每个树的所有节点属于一个集合(根节点为这个集合的代表元素), 并且任意两个集合之间的交集为空.

并查集支持以近似常量的复杂度对两个集合进行合并(merge)/查询某个元素属于那个集合(get)
并查集适合处理传递性关系(等于运算/无向图中两个节点是否连通)
```

并查集代码实现

```java
public class UnionSet {
    int[] father; // 记录每个节点的父节点
    int[] rank; // 记录树的大小
    public UnionSet(int size) {
        father = new int[size];
        rank = new int[size];
        Arrays.fill(rank, 1);
        for (int i = 0; i < size; i++) {
            father[i] = i;
        }
    }
    public int get(int id) {
        if (id == father[id]) return id;
        return father[id] = get(father[id]); // 路径压缩
    }
    public void merge(int id0, int id1) {
        int tree0 = get(id0);
        int tree1 = get(id1);
        if (tree1 == tree0) return;
        if (rank[tree0] > rank[tree1]) { // 按秩合并
            rank[tree0] += rank[tree1];
            father[tree1] = tree0;
        } else {
            rank[tree1] += rank[tree0];
            father[tree0] = tree1;
        }
    }
}
```

## 5 树状数组

树状数组使用一个树维护一个序列的前缀和, 可以在 log(N)时间内查询前缀 / 进行单点增加

![1687773702270](image/算法/1687773702270.png)

树状数组中 tree[x]表示什么?

```
tree[x]表示arr[1~x]之间的一个小区间arr[x-lowbit(x)+1 ~ x]的和
```

如何查询前缀和?

```
arr[1~x]的和 = tree[x] + tree[x-lowbit(x)] + ...
相当于求节点tree[x]与左侧兄弟节点的和

tree[x]左侧兄弟节点为tree[x - lowbit(x)]
```

```java
    // 查询前缀和
    public int ask(int x) {
        if (x >= n) x = n - 1;
        x++;
        int sum = 0;
        for (; x > 0; x -= lowBit(x)) {
            sum += tree[x];
        }
        return sum;
    }
```

如何进行单点增加?

```
要想使arr[x]的值增加increment, 将从叶节点tree[x]到根节点的路径中所有节点的值增加increment

tree[x]的父节点为tree[x + lowbit(x)]
```

## 6 线段树

# 3. 搜索

## 1 DFS 的剪枝

```
1. 记忆化剪枝: 在对图使用DFS搜索时, 可以标记一个节点是否访问过, 如果访问过则在搜索时跳过

2. 优化搜索顺序: 先搜索规模小的子树, 再搜索规模大的子树

3. 可行性剪枝: 如果当前分支一定搜索不到答案, 立刻返回

4. 最优化剪枝: 如果确认当前分支以后可以搜索到的答案一定不会优于当前搜索到的最优答案, 立刻返回
```

## 2 BFS 算法的变形

什么是优先队列 BFS?

```
一般的BFS问题等价于在边长均为1的图中搜索两点之间的最短路径
优先队列BFS等价于在边长不定的图中搜索两点之间的最短距离, 时间复杂度为N.log(N)
```

什么是 A\*算法?

```
在优先队列BFS的基础上, 使用未来代价预估函数costFuture(state) + 当前状态的代价costNow(state) 的值作为优先队列BFS排序的依据

必须满足条件: 未来代价预估函数costFuture(state) <= 当前状态到目标状态真实代价realCost(state), 才能保证: 到目标状态的最小代价路径先于其他路径出队
```

# 4. 数学相关

## 1 数论

**_如何获取 2~N 之间的所有质数?_**

**_如何判断一个数是否是质数?_**

**_最大公因数和最小公倍数怎么求?_**

```
最大公约数采用欧几里得算法

最小公倍数可以通过最大公约数计算
```

```Java
    public int gcb(int a, int b) {
        int remainder = a % b;
        while (remainder != 0) {
            a = b;
            b = remainder;
            remainder = a % b;
        }
        return b;
    }

    public int lcm(int a, int b) {
        return (a * b) / gcb(a, b);
    }
```

## 2 排列组合

## 3 其他经典数学问题

## 4 复杂计算的实现

**_如何使用矩阵的快速幂算法加速递推过程?_**

```
1. 对于初始状态, 记作一个一维向量 A = {a1, a2}

2. 把递推的过程抽象为一个转移矩阵S, 每次递推后的状态为 A*S

3. 如果递推的次数为n, 那么最终状态=A*(S^n), n非常大时使用快速幂算法
```

**_如何实现高斯消元求解方程?_**

# 5. 背包问题

dp[i][j]表示前 i 个物品在容积不超过 j 时的最大价值

```
01背包问题状态转移方程
dp[i][j] = max(dp[i-1][j], dp[i-1][j-weight[i]] + value[i])

完全背包问题状态转移方程
dp[i][j] = max(dp[i-1][j], dp[i][j-weight[i]] + value[i])
```

# 6. 图算法

如何求图中单点到其他点的最短路径?

Dijstra 算法的步骤是什么?

如何求图中任意两点之间的最短路径?

什么是最小生成树?

如何求最小生成树?

# 7. 奇淫技巧

_如何不使用中间变量交换两个 int 变量?_

```java
// 使用xor运算
    private void swapDemo() {
        int a = 985, b = 211;
        a ^= b;
        b ^= a;
        a ^= b;
        System.out.printf("a = %d, b = %d\n", a, b);
    }

```

_如何不使用加法运算符计算两数之和?_

```java
// 使用&模拟进位, ^模拟模2加
    private int add(int a, int b) {
        if (a == 0) return b;
        return add((a&b) << 1, a^b);
    }
```
