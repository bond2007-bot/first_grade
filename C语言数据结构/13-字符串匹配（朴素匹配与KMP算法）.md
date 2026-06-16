# 字符串匹配（朴素匹配与KMP算法）

## 📖 内容来源
- [[KMP.c]]

---

## 一、朴素字符串匹配（Naive String Matching）

**思想**：从主串的每个位置开始，与模式串逐字符比较。

```c
#include<stdio.h>
#include<string.h>

// 字符串的朴素匹配
int strMatch(char* str, char* pattern)
{
    int n = strlen(str);
    int m = strlen(pattern);

    for(int i = 0; i <= (n - m); i++)
    {
        int j = 0;
        while(j < m)
        {
            if(str[i] == pattern[j])
            {
                i++;
                j++;
            }
            else
            {
                i = i - j;  // 回溯到本轮起始位置的下一个
                break;
            }
        }
        if(j == m)
        {
            return i - j;  // 返回匹配的起始位置
        }
    }
    return -1;
}

int main()
{
    char* str = "abcabaabcabc";
    char* pattern = "abaa";
    int pos = strMatch(str, pattern);
    printf("%d\n", pos);  // 输出：3
    return 0;
}
```

### 朴素匹配过程示例
```
主串：    a b c a b a a b c a b c
模式串：  a b a a

第1轮：a b c ≠ a → 后移
第2轮：a b a a → 在第4个字符a=c不匹配 → 后移
第3轮：a b a a → 匹配成功！返回位置3
```

### 特性
| 项目 | 值 |
|------|-----|
| 时间复杂度 | O(n×m) |
| 最坏情况 | O(n×m)（如主串"aaaaaaaaab"，模式串"aaab"） |
| 空间复杂度 | O(1) |
| 缺点 | 大量回溯，效率低 |

---

## 二、KMP匹配算法

**KMP核心思想**：利用next数组，当匹配失败时，**主串指针不回溯**，模式串滑动到合适位置继续匹配。

```
朴素匹配失败时：i回溯到i-j+1，j归0
KMP匹配失败时：i不变，j = next[j]
```

### 1. KMP算法框架

```c
// KMP匹配
// 基于模式串确定next数组
// 利用next数组完成字符串匹配

int KMP(char* str, char* pattern, int next[])
{
    int n = strlen(str);
    int m = strlen(pattern);
    int i = 0, j = 0;

    while(i < n && j < m)
    {
        if(j == -1 || str[i] == pattern[j])
        {
            i++;
            j++;
        }
        else
        {
            j = next[j];  // 匹配失败时，模式串滑动到next[j]位置
        }
    }

    if(j == m)
        return i - j;  // 匹配成功
    else
        return -1;     // 匹配失败
}
```

### 2. next数组的计算

**next数组定义**：
- next[j] 表示当模式串中第j个字符与主串匹配失败时，模式串中新的匹配位置
- 即：**模式串前j-1个字符中，最长相等前后缀的长度**

**手工计算方法**：
```
设模式串 P = "abaabcac"

j   1  2  3  4  5  6  7  8
P   a  b  a  a  b  c  a  c

求next[j]：
next[1] = 0    // 规定
next[2] = 1    // 规定

next[3]: P[1..2]="ab"，无相等前后缀 → next[3]=1
next[4]: P[1..3]="aba"，前缀"a"=后缀"a" → 长度1 → next[4]=2
next[5]: P[1..4]="abaa"，前缀"a"=后缀"a" → 长度1 → next[5]=2
         但"ab"≠"aa"，所以最长=1 → next[5]=2
next[6]: P[1..5]="abaab"，前缀"ab"=后缀"ab" → 长度2 → next[6]=3
next[7]: P[1..6]="abaabc"，无相等前后缀 → next[7]=1
next[8]: P[1..7]="abaabca"，前缀"a"=后缀"a" → 长度1 → next[8]=2

最终：next = {0, 1, 1, 2, 2, 3, 1, 2}
        索引  1  2  3  4  5  6  7  8
```

### 3. nextval数组（优化版）

**nextval改进**：当 `P[j] == P[next[j]]` 时，进一步递归优化，避免无效匹配。

```
对上述 next 数组优化为 nextval：

next[1]=0 → nextval[1]=0
next[2]=1, P[2]=b, P[1]=a, b≠a → nextval[2]=1
next[3]=1, P[3]=a, P[1]=a, a=a → nextval[3]=nextval[1]=0
next[4]=2, P[4]=a, P[2]=b, a≠b → nextval[4]=2
next[5]=2, P[5]=b, P[2]=b, b=b → nextval[5]=nextval[2]=1
next[6]=3, P[6]=c, P[3]=a, c≠a → nextval[6]=3
next[7]=1, P[7]=a, P[1]=a, a=a → nextval[7]=nextval[1]=0
next[8]=2, P[8]=c, P[2]=b, c≠b → nextval[8]=2

nextval = {0, 1, 0, 2, 1, 3, 0, 2}
```

---

## 三、朴素匹配 vs KMP 对比

| 特性 | 朴素匹配 | KMP算法 |
|------|---------|---------|
| 时间复杂度 | **O(n×m)** | **O(n+m)** |
| 空间复杂度 | O(1) | O(m)（存储next数组） |
| 主串指针回溯 | ❌ 需要回溯 | ✅ **不回溯** |
| 实现难度 | 简单 | 较复杂 |
| 适用场景 | 短字符串 | 长字符串、重复匹配 |

---

## 四、期末考试考点

### 📌 核心考点
1. **next数组的计算**（必考大题）
2. **nextval数组的计算**（优化版，常考）
3. **KMP匹配过程**（手动模拟）
4. **朴素匹配的回溯过程**

### 📌 常见题型
- **选择题**：KMP时间复杂度、next数组含义
- **填空题**：给定模式串求next数组
- **简答题**：
  - 写出next数组和nextval数组
  - 描述KMP匹配过程
  - 比较朴素匹配和KMP的异同
- **算法题**：KMP算法的next数组计算

### 📌 KMP计算规则总结

```
next数组：
1. next[1] = 0, next[2] = 1（固定值，编号从1开始）
2. 对于 j > 2：
   - 找 P[1..j-1] 中最长的相等前后缀长度 k
   - next[j] = k + 1
   - 如果没有相等前后缀，next[j] = 1

nextval数组（优化）：
1. nextval[1] = 0
2. 对于 j > 1：
   - 若 P[j] ≠ P[next[j]] → nextval[j] = next[j]
   - 若 P[j] = P[next[j]] → nextval[j] = nextval[next[j]]
```

### 📌 易错点
- next数组编号从1开始，不是0
- 前后缀比较时，前缀不包含最后一个字符，后缀不包含第一个字符
- KMP主串指针i**永不回溯**，这是其高效的关键
- 注意区分next和nextval的计算规则
