---
tags: c++, c++11
---

[![hackmd-github-sync-badge](https://hackmd.io/kfOxqe-JRs6xihKRzIUjBQ/badge)](https://hackmd.io/kfOxqe-JRs6xihKRzIUjBQ)

# 如何寫出不會被誤用的code #2


```cpp=
class Rectangle {
public:
    Rectangle(unsigned int w, unsigned int h)
            : width(w), height(h) {
    }

private:
    unsigned int width;
    unsigned int height;
};
```

一個常見的矩形class，長和寬是unsighed int，看起來很美好。caller可以這樣寫，完全沒問題。
```cpp
Rectangle(100, 200);
```

## 兩個問題
1. width和height寫反也合法，上一集有講解，不多贅述
3. 引數是負數也完全合法
```cpp
Rectangle(-100, -200);
```


## 將width和height變成新的type再加上explicit呢?
利用上一集寫過的例子來改，會變這樣

```cpp=
class Width {
public:
    explicit Width(unsigned int w)
            : width(w) {
    }

private:
    unsigned int width;
};

class Height {
public:
    explicit Height(unsigned int h)
            : height(h) {
    }

private:
    unsigned int height;
};


class Rectangle {
public:
    Rectangle(const Width &w, const Height &h)
            : width(w), height(h) {
    }

private:
    Width width;
    Height height;
};
```
現在解決了第一個問題，至少width和height的順序固定了。
但第二個問題沒有解決，將負數assign給unsigned int是合法的，width和height會變成一個很大的正整數，通常我們不會希望這樣的結果。

## 利用type traits來檢查
type traits提供了一系列型別檢查用的template，其中一個std::unsigned剛好可以派上用場。
改寫一下上面的code：

```cpp=
template <typename T>
class Width {
public:
    explicit Width(T w)
            : width(w) {
        static_assert(std::is_unsigned_v<T>, "signed types is not allow.");
    }

private:
    T width;
};

template <typename T>
class Height {
public:
    explicit Height(T h)
            : height(h) {
        static_assert(std::is_unsigned_v<T>, "signed types is not allow.");
    }

private:
    T height;
};

template <typename T1, typename T2>
class Rectangle {
public:
    Rectangle(const Width<T1> &w, const Height<T2> &h)
            : width(w), height(h) {
    }

private:
    Width<T1> width;
    Height<T2> height;
};
```
現在caller只能用unsigned types來宣告width和height了

```cpp
Rectangle(Width(100u), Height(200u)); // 注意要加上unsigned-suffix 'u'
```

(待續)