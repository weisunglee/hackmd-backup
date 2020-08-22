---
tags: c++, c++11, CRTP
---
# Curiously Recurring Template Pattern，從入門到放棄

<!-- .slide: data-transition="fade-out" -->

---

## Polymorphism
一個介面，不同行為

<!-- .slide: data-transition="fade-out" -->

---

## Dynamic Polymorphism
Virtual function
<!-- .slide: data-transition="fade-out" -->

----

```cpp=
class Shape {
public:
    virtual ~Shape() = default;
    virtual void Draw() = 0;
};
```

```cpp=+
class Circle : public Shape {
public:
    ~Circle() = default;
    void Draw() override {
        std::cout << "Circle::Draw" << std::endl;
    }
};
```

```cpp=+
class Square : public Shape {
public:
    ~Square() = default;
    void Draw() override {
        std::cout << "Square::Draw" << std::endl;
    }
};
```

<!-- .slide: data-transition="fade-out" -->

---

## Static Polymorphism
CRTP
<!-- .slide: data-transition="fade-out" -->

----

* [Wiki](https://en.wikipedia.org/wiki/Curiously_recurring_template_pattern)
* [C++ Now 2018](https://www.youtube.com/watch?v=0hy3DhE5pIo)



<!-- .slide: data-transition="fade-out" -->

----

> I'm really surprised this was allowed in a C++ conference. Literally the Wikipedia page for CRTP has this example in it. It has been known since C++98 and way before that even.

<!-- .slide: data-transition="fade-out" -->

----

```cpp=
template <typename Derived>
class Shape {
public:
    void Draw() {
        static_cast<Derived*>(this)->Draw();
    }
};
```

```cpp=+
class Circle : public Shape<Circle> {
public:
    void Draw() {
        std::cout << "Circle::Draw" << std::endl;
    }
};
```

```cpp=+
class Square : public Shape<Square> {
public:
    void Draw() {
        std::cout << "Square::Draw" << std::endl;
    }
};
```

<!-- .slide: data-transition="fade-out" -->

---

## Static VS. Dynamic
* 效能
* 可維護性
* 可讀性
<!-- .slide: data-transition="fade-out" -->

---

## Non-void function (Cont'd)

```cpp=
template <typename Derived>
struct Shape {
    bool Draw() {
        return static_cast<Derived*>(this)->Draw();
    }
};
```

```cpp=+
struct Circle : public Shape<Circle> {
    bool Draw();
};
```

```cpp=+
Circle c;
c.Draw();
```

<!-- .slide: data-transition="fade-out" -->

----

## Non-void function

```cpp=
template <typename Derived>
struct Shape {
    bool Draw() {
        return static_cast<Derived*>(this)->Draw();
    }
};
```

```cpp=+
struct Circle : public Shape<Circle> {
    //bool Draw();
};
```

```cpp=+
Circle c;
c.Draw();
```

<!-- .slide: data-transition="fade-out" -->

----

## Since C++14

```cpp=
template <typename Derived>
struct Shape {
    auto Draw() {
        return static_cast<Derived*>(this)->Draw();
    }
};
```

```cpp=+
struct Circle : public Shape<Circle> {
    //bool Draw();
};
```

```cpp=+

// complie error
Circle c;
c.Draw();
```

<!-- .slide: data-transition="fade-out" -->

----

```cpp=
template <typename Derived>
struct Shape {
    auto Draw() {
        return static_cast<Derived*>(this)->DoDraw();
    }
};
```

```cpp=+
struct Circle : public Shape<Circle> {
    bool DoDraw();
};
```

```cpp=+
struct Square : public Shape<Square> {
    void DoDraw();
};
```

```cpp=+
Circle c;
c.Draw();
Square s;
s.Draw();
```

<!-- .slide: data-transition="fade-out" -->

----

## Polymorphism or not

```cpp=
template <typename Derived>
struct Shape {
    template<typename ...Args>
    auto Draw(Args&& ...args)  {
        return self()->DoDraw(std::forward<Args>(args)...);
    }

    Derived* self() {
        return static_cast<Derived*>(this);
    }
};
```

```cpp=+
struct Circle : public Shape<Circle> {
    bool DoDraw(int radius);
};
```

```cpp=+
struct Square : public Shape<Square> {
    void DoDraw(int width, int height);
};
```

<!-- .slide: data-transition="fade-out" -->

---

## Modern CRTP
<!-- .slide: data-transition="fade-out" -->

----

```cpp=
template <typename Derived>
class Shape {
public:
    auto Draw() const {
      return self().DoDraw();
    }
```

```cpp=+
private:
    // isolate the static_cast
    inline Derived& self() {
      return *static_cast<Derived*>(this);
    }

    inline Derived const& self() const {
      return *static_cast<const Derived*>(this);
    }
};

```

<!-- .slide: data-transition="fade-out" -->

----

```cpp=+
class Circle final : public Shape<Circle> {
private:
    auto DoDraw() const;
    friend class Shape<Circle>;
};
```

<!-- .slide: data-transition="fade-out" -->

---

## Container

<!-- .slide: data-transition="fade-out" -->

----


## std::vector<????????>

1. Shape
2. Circle
3. Square
4. Shape<Circle>
5. Shape<Square>

<!-- .slide: data-transition="fade-out" -->

----

![](https://i.imgur.com/Yzuu5bw.png)

<!-- .slide: data-transition="fade-out" -->

---

## std::variant
> The class template std::variant represents a `type-safe union`.

<!-- .slide: data-transition="fade-out" -->

----

```cpp=
using var_t = std::variant<Circle, Square>;
std::vector<var_t> v{Circle{}, Square{}};

try {
    auto shape1 = std::get<Circle>(v[0]);
    auto shape2 = std::get<Square>(v[1]);
}
catch(std::bad_variant_access&) {

}
```

<!-- .slide: data-transition="fade-out" -->

----


```cpp=
auto* p = std::get_if<Circle>(&v[0]);

if (p != nullptr)
    ...
else
    ...

```

<!-- .slide: data-transition="fade-out" -->

----

## std::visit

```cpp=

struct Visitor {
    void operator()(const Circle& s) { s.Draw(); }
    void operator()(const Square& s) { s.Draw(); }
};

```

```cpp=+
int main() {
    using var_t = std::variant<Circle, Square>;
    std::vector<var_t> v{Circle{}, Square{}};

    Visitor visitor;
    std::for_each(std::cbegin(v), std::cend(v)
    , [&visitor](const auto& s){
        std::visit(visitor, s);
    });

    return 0;
}
```

<!-- .slide: data-transition="fade-out" -->

----

```cpp=
std::for_each(std::cbegin(v), std::cend(v)
, [](const auto& s) {
    std::visit([](const auto& shape){ shape.Draw();}, s);
});
```

<!-- .slide: data-transition="fade-out" -->

----

```cpp=
template<typename... Ts>
struct overloaded : Ts... { using Ts::operator()...; };
template<typename... Ts>
overloaded(Ts...) -> overloaded<Ts...>;
```

```cpp=+
std::for_each(std::cbegin(v), std::cend(v)
, [](const auto& s) {
    std::visit(overloaded {
        [](const Circle& shape){ shape.Draw(); },
        [](const Square& shape){ shape.Draw(); },
    } , s);
});
```

<!-- .slide: data-transition="fade-out" -->

---

## virtual function VS. CRTP
效能沒有很明顯的差異

* 換compiler (g++、clang++)
* Optimization level (O0~O3)
* inline

<!-- .slide: data-transition="fade-out" -->

---

## Conclusion

* 用virtual function就能完成絕大部分需要polymorphism的需求
* 設計library時可以考慮使用CRTP，但不一定是為了實現polymorphism


<!-- .slide: data-transition="fade-out" -->

---

# Q & A

<!-- .slide: data-transition="fade-out" -->