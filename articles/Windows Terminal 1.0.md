---
tags: windows, windows terminal, powershell, posh-git, oh-my-posh
---
# Windows Terminal 1.0

Windows今年感覺有很多不錯的新功能，今天看到Windows Terminal正式release的消息，就裝來玩看看。

## 安裝
https://www.microsoft.com/zh-tw/p/windows-terminal-preview/9n0dx20hk701
直接從Store裝就好

## 安裝新版Power shell
https://github.com/PowerShell/PowerShell/releases
我自己是安裝6.2.5，本來的版本是5，可以並存沒有問題

## 安裝 posh-git 和 oh-my-posh:
方便直接在畫面上顯示git branch和路徑，看了舒服

github:
https://github.com/JanDeDobbeleer/oh-my-posh

這邊有提到一個東西ConEmu，我有安裝，利用Chocolatey安裝的，如果還沒安裝Chocolatey的話參考 https://chocolatey.org/


這樣要安裝ConEmu只需要輸入

```
choco install ConEmu
```

接著再安裝
posh-git 和 oh-my-posh

```
Install-Module posh-git -Scope CurrentUser
Install-Module oh-my-posh -Scope CurrentUser
```

## 安裝 PSReadline
不知道做什麼的但我裝了

```
Install-Module -Name PSReadLine -AllowPrerelease -Scope CurrentUser -Force -SkipPublisherCheck
```

## 設定

輸入

```
notepad $PROFILE
```

如果沒有設定過會提示，建立一個新的就好，接著在記事本內加入這段

```
Import-Module posh-git
Import-Module oh-my-posh
Set-Theme Paradox
```

上面的Paradox只是其中一種theme，還有其他選擇，可以在這邊找
https://github.com/JanDeDobbeleer/oh-my-posh?WT.mc_id=-blog-scottha#themes

## 完成

### Before
![](https://i.imgur.com/aiSRu6M.png)

### After
![](https://i.imgur.com/rgHKiNe.png)
