---
tags: git, cherry-pick, rebase, --onto
---
# Git cherry-pick的坑

## 前言
之前在開發的時候，遇過一次把A branch merge回B branch之後，發現在A branch上的修改沒有正確被merge，和預期的結果不一樣

當時的狀況是A branch cherry-pick了B branch的一個commit，但是在A branch上又用了git revert，revert了這個commit的修改

```
A1 ---A2 ---------------- A3 --------------------A4
       \  cherry-pick                           /  merge B3
        \                                      /
B1 ---  A2' --- git revert A2' --- B2 ------- B3
```

git revert不會被正確apply，也就是A2這個commit的修正在A branch還是存在
> 我們在B branch上revert A2'，當B branch merge回A branch時，預期A2的修正應該要
> 被revert

## cherry-pick不要亂用
[1]這篇已經把cherry-pick會遇到的坑都解釋得非常清楚

不當使用cherry-pick有可能會造成
1. merge conflict
2. merge非預期結果 (很可怕，因為只能靠事後比對才能發現)

## 如何避免

只要把握兩個原則就可以避開cherry-pick的坑。

1. 兩個branch的內容需要雙向同步的話，應該要用merge
2. 兩個branch的內容只需要單向同步的話可以用cherry-pick或merge


## 有很多commit需要cherry-pick的時候

cherry-pick可以將commit移到別的branch，但如果commit很多的時候要一個一個pick過去
如果沒有特別的理由，例如，你想要改變commit的順序，那可以用git rebase來達到一樣的結果。

用個實際的例子

```
A1 --- A2 --- A3          A

B1 --- B2 --- B3 --- B4   B
        \            /
        \           /
        C1 ------- C2     C
```

C branch merge回B branch，但同時也想要merge到A branch

通常我會這樣做

```
$ git checkout C
$ git rebase --onto A HEAD~2
```

git rebase --onto後面接的參數第一個是新的base，第二個是現在的base
執行完之後就會把現在base的之後的commit，移到新的base上，就會變成這樣

```
A1 --- A2 --- A3          A
               \
               C1 --- C2  C
```

之後就可以merge了

## 很懶得數到底有幾個commit要搬到新的branch

git rebase --onto 後面可以接三個參數

```
$ git rebase --onto {new base} {after this commit} {to this commit}
```

你可以給一個區間，讓git自己去找到底有幾個commit需要merge，以上面的例子來說的話

```
$ git rebase --onto A B2 C2
```

這樣就可以達到一樣的結果了

再給個簡單一點的例子，來自於git的說明

```
                                       H---I---J topicB
                                      /
                             E---F---G  topicA
                            /
               A---B---C---D  master


       then the command

           git rebase --onto master topicA topicB

       would result in:

                            H'--I'--J'  topicB
                           /
                           | E---F---G  topicA
                           |/
               A---B---C---D  master

```

## 觀念釐清
如果還看不懂上面的例子，根據我自己學git的經驗，可能是對git觀念不清楚造成的
在git裡面所有commit都有一個SHA-1 hash ID，所有能指定到特定commit的操作
不管是改變HEAD、切換branch、還是tag，都只是特定commit的reference，也可以想成是alias

## Reference
[[1] Stop cherry-picking, start merging](https://devblogs.microsoft.com/oldnewthing/20180312-00/?p=98215)