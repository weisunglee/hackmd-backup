---
tags: c++, c++11, unordered_map
---
# 為什麼unordered_map的key不能是const string

今天同事把一個map replace成unordered_map，因為沒有排序的需求，結果就遇到static assertion

```cpp
static_assert(__check_hash_requirements<_Key, _Hash>::value,
"the specified hash does not meet the Hash requirements");
```

---

我好奇點進去看這個 `__check_hash_requirements` 到底幹了什麼好事

```cpp
template <class _Key, class _Hash>
using __check_hash_requirements = integral_constant<bool,
    is_copy_constructible<_Hash>::value &&
    is_move_constructible<_Hash>::value &&
    __invokable_r<size_t, _Hash, _Key const&>::value
>;
```

感覺怪怪的不知道哪裡有問題，這個問題太難了，只好先問問stackoverflow，果然有找到一篇在問相同問題的

[stackoverflow : Using a const key for unordered_map](https://stackoverflow.com/questions/3998978/using-a-const-key-for-unordered-map)

裡面的回答提到


> The associative containers only expose the (key,value) pair as
std::pair<const key_type, mapped_type>, so the additional const
on the key type is superfluous.

不過看起來他也是亂回答的，因為std::map也是用一樣的pair啊，為什麼std::map就可以用const string當作key？

問題還是出在hash，在仔細看了cppreference後突然有點想法了

---

unordered_map長這樣

```cpp
template<
    class Key,
    class T,
    class Hash = std::hash<Key>,
    class KeyEqual = std::equal_to<Key>,
    class Allocator = std::allocator< std::pair<const Key, T> >
> class unordered_map;
```

有問題的是這裡的 `std::hash<Key>`，再去查一下 `std::hash`
看到一段是這樣寫著

![](https://i.imgur.com/W16faIS.png)

對比了一下實作，在`<string>`裡對hash的特化是寫成這樣

```cpp
template<class _CharT, class _Traits, class _Allocator>
struct _LIBCPP_TEMPLATE_VIS hash<basic_string<_CharT, _Traits, _Allocator> >
    : public unary_function<basic_string<_CharT, _Traits, _Allocator>, size_t>
{
    size_t
        operator()(const basic_string<_CharT, _Traits, _Allocator>& __val) const _NOEXCEPT;
};

template<class _CharT, class _Traits, class _Allocator>
size_t
hash<basic_string<_CharT, _Traits, _Allocator> >::operator()(
        const basic_string<_CharT, _Traits, _Allocator>& __val) const _NOEXCEPT
{
    return __do_string_hash(__val.data(), __val.data() + __val.size());
}
```

嗯，沒有const，那只好自己寫一個去呼叫沒有const的版本，結果是可正常執行

```cpp
namespace std {
    template <>
    struct hash<const std::string> {
        size_t operator()(const std::string& str) const noexcept {
            std::hash<string> hasher;
            return hasher(str);
        }
    };
}
```

```cpp
int main() {
    std::unordered_map<const std::string, int> map {{"a", 1}, {"b", 2}};
    std::cout << map["a"] << " " << map["b"] << std::endl;
    return 0;
}
```

雖然在一開始發現assertion的時候直接拿掉const就是最快的解決方式，但因為好奇到底為什麼，也意外發現一個stackoverflow上的解答是錯的，也沒有人糾正他。

## 最後我還是把const移除了。
![](https://i.imgur.com/bErqt8l.png)


