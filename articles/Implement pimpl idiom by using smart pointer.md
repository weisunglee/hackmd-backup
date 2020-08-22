---
tags: c++, c++11, pimpl
---
# Implement pimpl idiom by using smart pointer

<!-- .slide: data-transition="fade-out" -->

---

autotime.h

```cpp=
#ifdef _WIN32
#include <windows.h>
#else
#include <sys/time.h>
#endif
#include <string>
```

```cpp=+
class AutoTimer {
public:
    explicit AutoTimer(const std::string& name);
    ~AutoTimer(); //解構時顯示出經過了多久時間

private:
    double GetElapsed() const;
    std::string m_Name;
#ifdef _WIN32
    DWORD m_StartTime;
#else
    struct timeval m_StartTime;
#endif
};
```

<!-- .slide: data-transition="fade-out" -->

---

autotime.h, Pimpl idiom

```cpp=
#include <string>
```

```cpp=+
class AutoTimer {
public:
    explicit AutoTimer(const std::string& name);
    ~AutoTimer();

private:
    class Impl; // forward declaration
    Impl *m_Impl; // opaque pointer
};
```

<!-- .slide: data-transition="fade-out" -->

---

autotime.cpp

```cpp=
#include "autotime.h"
#ifdef _WIN32
#include <windows.h>
#else
#include <sys/time.h>
#endif
```

```cpp=+
class AutoTimer::Impl {
public:
    AutoTimer::Impl(const std::string& name)
        : m_Name(name) {
        ...
    }

    AutoTime::~Impl() {
        ...
    }

private:
    double GetElapsed() const {
        ...
    }

    std::string m_Name;
#ifdef _WIN32
    DWORD m_StartTime;
#else
    struct timeval m_StartTime;
#endif
};

AutoTimer::AutoTimer(const std::string& name)
    : m_Impl(new AutoTimer::Impl(name)) {
    ...
}

// 印出時間, delete m_Impl
AutoTimer::~AutoTimer() {
    ...
}
```

<!-- .slide: data-transition="fade-out" -->

---

## PIMPL Idiom (Pointer to IMPLementation)

* 避免在header file公開private data的一種方式
    * C++上的一種解決辦法，嚴格來說不算Design Pattern
    * 特殊情況的Bridge Pattern

<!-- .slide: data-transition="fade-out" -->

---

## Compilation Firewall

![](https://i.imgur.com/4hUl2wg.png)

---

## Pros
* 資訊隱藏
* 解耦
* 編譯變快
* ABI compatible
* Lazy allocation

<!-- .slide: data-transition="fade-out" -->

---

## Cons
* Runtime overhead
* Maintenance overhead

<!-- .slide: data-transition="fade-out" -->

---

## 前面範例的問題
* raw pointer
* 需要記得實作copy ctor，免得有人誤用

<!-- .slide: data-transition="fade-out" -->

---

## Demo code
[Pimpl Demo](https://github.com/weisunglee/PimplDemo)

<!-- .slide: data-transition="fade-out" -->

---

## Using smart pointer

Widget.h

```cpp=
#include <memory>

class Widget {
public:
    Widget();
    void Foo();

private:
    class Impl;
    std::unique_ptr<Impl> m_Impl;
};
```

<!-- .slide: data-transition="fade-out" -->

---

Widget.cpp

```cpp=
#include "Widget.h"
#include <iostream>
```

```cpp=+
class Widget::Impl {
public:
    void Foo() {
        std::cout << "Foo" << std::endl;
    }
};

Widget::Widget()
    : m_Impl(std::make_unique<Impl>()) {}

void Widget::Foo() {
    m_Impl->Foo();
}

```

<!-- .slide: data-transition="fade-out" -->

---

# But...

<!-- .slide: data-transition="fade-out" -->

---

## Compile Error

```bash
error: invalid application of 'sizeof'
to an incomplete type 'Widget::Impl'
static_assert(sizeof(_Tp) > 0,
              ^~~~~~~~~~~
```

<!-- .slide: data-transition="fade-out" -->

---

std::unique_ptr, defined in header \<memory\>

```cpp=
template<
    class T,
    class Deleter = std::default_delete<T>
> class unique_ptr;
```

* std::unique_ptr may be constructed for an incomplete type T

<!-- .slide: data-transition="fade-out" -->

---

* complier 自動產生的 dtor 為inline
* 需要自己定義widget的 dtor
<!-- .slide: data-transition="fade-out" -->

---

## Rule of three/five

* destructor
* copy constructor
* copy assignment operator
* move constructor (C++11)
* move assignment operator (C++11)

<!-- .slide: data-transition="fade-out" -->

---

std::shared_ptr, defined in header \<memory\>

```cpp=
template< class T > class shared_ptr;
```

* 用shared_ptr就不需要自己定義dtor
* 但通常pimpl idiom用unique_ptr比較符合現況

<!-- .slide: data-transition="fade-out" -->

---

## Pimpl template，讓實作pimpl時更方便

<!-- .slide: data-transition="fade-out" -->

---

當Widget::Impl需要帶有參數的ctor時該怎麼辦

<!-- .slide: data-transition="fade-out" -->

---

設計API時要注意const member function在pimpl下的合理性

> std::experimental::propagate_const

<!-- .slide: data-transition="fade-out" -->

---

## 結論

* 設計API時，使用pimpl idiom對用API的人來說是方便的
* 大型專案中，努力讓編譯更快是值得的

<!-- .slide: data-transition="fade-out" -->

---

## Reference

* Martin Reddy, API Design for C++
* Scott Meyers, Effective Modern C++
* https://en.cppreference.com
* https://www.fluentcpp.com
* Sutter’s Mill, https://herbsutter.com/gotw/_100/
* Sutter’s Mill, https://herbsutter.com/gotw/_101/

<!-- .slide: data-transition="fade-out" -->

---

## Q&A

<!-- .slide: data-transition="fade-out" -->

---

# C++20 : Modules

<!-- .slide: data-transition="zoom" data-transition-speed="slow" -->
