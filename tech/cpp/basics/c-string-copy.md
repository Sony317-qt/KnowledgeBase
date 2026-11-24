---
id: cpp/basics/c-string-copy
title: C 字符串遍历与拷贝
level: null
level_status: 待用户确认
reviewed: 2026-10-30
next_review: 2026-11-01
projects: []
sources: [本次聊天整理]
---

## 一句话

字符指针逐个读取和写入字符，遇到零字符结束，目标也必须补结束标记。

## 机制
```c
char* my_strcpy(char* dest, const char* src) {
    char* start = dest;
    while (*src != '\0') {
        *dest = *src;
        ++dest;
        ++src;
    }
    *dest = '\0';
    return start;
}
```
- *dest=*src 复制一个字符；dest=src 才是复制地址。char 指针递增移动一个字符。
- src[i] 等价于 *(src+i)。函数内递增指针副本，不改变调用者的指针，但写入 *dest 会改变外部数组。
- 调用前必须保证源有效且有结束符、目标足够大、区域不重叠。此函数没有容量检查。
- char str[]="hello" 的 sizeof 为 6，含结束符；char* p=str 的 sizeof 是指针大小。数组名不能像指针变量一样递增。

## 追问链
1. 为何源用 const？→ 只读字符，不允许通过该路径写入。
2. 漏掉结束符？→ 后续按字符串读取可能越界。
3. 返回 start 而非 dest？→ dest 已移动，start 保留起始地址。

## 项目证据 / 反例

- 未提供项目证据；保留对话中的易错点供复习。
