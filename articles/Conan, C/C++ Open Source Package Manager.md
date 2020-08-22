---
tags: c++, conan, package manager, visual studio
---
# Conan, C/C++ Open Source Package Manager
### https://conan.io/
<!-- .slide: data-transition="fade-out" -->

---

![](https://i.imgur.com/SsnatbR.png)

<!-- .slide: data-transition="fade-out" -->

---

![](https://i.imgur.com/GTzPr8S.png)

<!-- .slide: data-transition="fade-out" -->

---

## 先講古
<!-- .slide: data-transition="fade-out" -->

---


## C/C++ libraries

* header files
* static/dynamic link binaries

<!-- .slide: data-transition="fade-out" -->

---

## C/C++ libraries

```bash
├── include
├── src
└── lib
```

<!-- .slide: data-transition="fade-out" -->

---

## 還好網路上充斥 XXX lib 設定教學

<!-- .slide: data-transition="fade-out" -->

---

## 萬一沒設定好...

* 找不到include -> compile error
* 找不到lib -> link error

<!-- .slide: data-transition="fade-out" -->

---

## Windows更慘

* 用哪一版vc build出來的exe就必須link到哪一版的build的lib
* Visual C++ ABI compatibility到2015年後才比較好 (vc140、vc141、vc142彼此兼容)


<!-- .slide: data-transition="fade-out" -->

---

## 許多library只提供src

<!-- .slide: data-transition="fade-out" -->

---

## 根據自己的需求build
```
├── 1.0
├── 1.1
└── 1.2
     ├───x64
     └───x86
          ├───static
          └───dynamic
               ├───debug
               └───release
```

<!-- .slide: data-transition="fade-out" -->

---

## 時間成本
* 研究怎麼build
* 等待漫長的build lib過程
* 修link error
<!-- .slide: data-transition="fade-out" -->

---

## 共同開發時, 如何分享library
1. 自行安裝library, 必須安裝在相同的路徑下
2. 把需要用到的library放在git上再用submodule管理

<!-- .slide: data-transition="fade-out" -->

---

## 缺點
1. 自行安裝容易出錯, build不過
2. 需要多版本共存時不易管理
3. git對於大檔案管理的缺陷
4. 無法只下載特定版本的library, 除非你一個版本一個repo

<!-- .slide: data-transition="fade-out" -->

---

![](https://i.imgur.com/u9szyar.png)

<!-- .slide: data-transition="fade-out" -->

---

![](https://i.imgur.com/nRLR44P.png)

<!-- .slide: data-transition="fade-out" -->

---

### https://conan.io/
![](https://i.imgur.com/PWxR2w0.png)

<!-- .slide: data-transition="fade-out" -->

---

## 安裝

### python  2.7、3.5~3.7
`$ pip install conan`

### OSX
`$ brew install conan`

<!-- .slide: data-transition="fade-out" -->

---

## 以Windows Visual Studio為例

<!-- .slide: data-transition="fade-out" -->

---

```
$ conan search boost --remote=conan-center

Existing package recipes:

boost/1.66.0@conan/stable
boost/1.67.0@conan/stable
boost/1.68.0@conan/stable
boost/1.69.0@conan/stable
boost/1.70.0
boost/1.70.0@conan/stable
boost/1.71.0
boost/1.71.0@conan/stable
boost/1.72.0
```

<!-- .slide: data-transition="fade-out" -->

---

## search的陷阱
* 目前只有 case-sensitive search, 沒辦法改成case-insensitive
* 需要fuzzy search的時候需要自己加上 *, 例如boo\*
<!-- .slide: data-transition="fade-out" -->

---

conanfile.txt
```
 [requires]
 boost/1.66.0@conan/stable

 [generators]
 visual_studio
```

<!-- .slide: data-transition="fade-out" -->

---

![](https://i.imgur.com/B75Rqdm.png)

<!-- .slide: data-transition="fade-out" -->

---

![](https://i.imgur.com/5aAKRNT.png)

<!-- .slide: data-transition="fade-out" -->

---

## Build !

<!-- .slide: data-transition="fade-out" -->

---

![](https://i.imgur.com/ssU7tWh.jpg)

<!-- .slide: data-transition="fade-out" -->

---

## 參考文件

![](https://i.imgur.com/6C8rWRH.png)

<!-- .slide: data-transition="fade-out" -->

---

remotes.json

```
{
 "remotes": [
  {
   "name": "conan-center",
   "url": "https://conan.bintray.com",
   "verify_ssl": true
  }
 ]
}
```

<!-- .slide: data-transition="fade-out" -->

---

## 新增conan server
```
$ conan_server
$ conan remote add myremote https://localhost:9300
```

<!-- .slide: data-transition="fade-out" -->

---

## 比較
* conan, 根據conanfile.txt的內容, 只下載header和lib, 安裝到home目錄下
* vcpkg, 只下載source code, 需要自己build library
* NuGet, 只能搭配Visual Studio, library會安裝到project目錄下, 不同project會有重複下載的問題

<!-- .slide: data-transition="fade-out" -->

---

## Q & A
