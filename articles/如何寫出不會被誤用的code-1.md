---
tags: c++, c++11
---

[![hackmd-github-sync-badge](https://hackmd.io/kfOxqe-JRs6xihKRzIUjBQ/badge)](https://hackmd.io/RpUhk9JnR6u4aMYqTevEaA)

# 如何寫出不會被誤用的code #1

良好的API設計下，如果user寫錯，compile應該會失敗。
最近剛好要改寫一段code，大概是像這樣：

```cpp=
#include <iostream>
#include <string>

using namespace std; // 好孩子不要亂加這行，我只是寫範例

class Encryptor {
public:
    string encrypt(const string& key, const string& plainText) const {
        return "CypherText";
    }
};

int main() {
    Encryptor e;
    cout << e.encrypt("key", "PlainText") << endl;
    return 0;
}
```

## 問題
```cpp
string encrypt(const string& key, const string& text)
```
這邊參數(parameter)都是用string，一個是代表加密演算法所需要的key，一個是代表欲加密的plain text，如果順序寫反了，結果會不一樣。(範例都是回傳"CypherText"所以沒差，但這只是範例！)

看來我們需要型別檢查，讓compiler禁止參數順序擺錯。

## 用typedef或using可不可以

```cpp=
using CypherText = string;
using PlainText = string;
using Key = string;

class Encryptor {
public:
    CypherText encrypt(const Key& key, const PlainText& plainText) const {
        return CypherText{"CypherText"};
    }
};

int main() {
    Encryptor e;
    cout << e.encrypt(Key{"key"}, PlainText{"PlainText"}) << endl;

    return 0;
}
```

不幸的是，不管是typedef還是using，都只是alias而已，對事情沒有幫助，因為本質上type沒有改變，所以我改變引數(argument)的順序compiler也不會告訴我寫錯了。

## 好，直接寫新的type

```cpp=
struct CypherText {
    CypherText(string s)
        : text(s) {
    }
    string text;
};

struct PlainText {
    PlainText(string s)
        : text(s) {
    }
    string text;
};

struct Key {
    Key(string s)
        : key(s) {
    }
    string key;
};

ostream& operator<<(ostream& os, CypherText c) {
    return os << c.text;
}

class Encryptor {
public:
    CypherText encrypt(const Key& key, const PlainText& plainText) const {
        return CypherText{"CypherText"};
    }
};

int main() {
    Encryptor e;
    cout << e.encrypt(Key{"key"}, PlainText{"PlainText"}) << endl;
    return 0;
}
```

看起來解決了引數(argument)順序寫錯的問題，但如果有人這樣寫：
```cpp
e.encrypt(string{"PlainText"}, string{"key"});
```
因為string可以轉型成Key，也可以轉型成PlainText (implicit conversion)，所以語法上這樣寫沒有問題。

## 禁用implicit conversion

```cpp=
struct CypherText {
    explicit CypherText(string s)
        : text(s) {
    }
    string text;
};

struct PlainText {
    explicit PlainText(string s)
        : text(s) {
    }
    string text;
};

struct Key {
    explicit Key(string s)
        : key(s) {
    }
    string key;
};

ostream& operator<<(ostream& os, CypherText c) {
    return os << c.text;
}

class Encryptor {
public:
    CypherText encrypt(const Key& key, const PlainText& plainText) const {
        return CypherText{"CypherText"};
    }
};

int main() {
    Encryptor e;
    // compile error
    cout << e.encrypt(string{"key"}, string{"PlainText"}) << endl;
    return 0;
}
```
compiler會告訴你不能從string轉型成Key，PlainText同理。(待續)

