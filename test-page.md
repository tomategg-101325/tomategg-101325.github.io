---
layout: page
title: "Test Page"
permalink: /test/
---

# This is a Test Page!

## Section 1

[Here](https://shuiyuan.sjtu.edu.cn/t/topic/369637) is my diary on SJTU BBS. Unfortunately, only those with a JAccount can visit.

- 📢 让<u>我们</u>**说***中文*！
- ~~これは私のサイトです。~~

---

## Section 2

Below is the implementation of `find_min`:

```cpp
template <typename _T>
std::optional<_T> find_min(std::vector<_T>::const_iterator begin, std::vector<_T>::const_iterator end) {
    if (begin == end) return std::nullopt;

    _T result = *begin;
    for (auto it = begin + 1; it != end; ++it) {
        if (*it < result) result = *it;
    }
    return result;
}
```

