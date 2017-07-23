---
layout: post
title: "git-rebase-squash"
date: 2017-07-24 00:28:35 +0300
comments: true
categories:
  - git
---
В моей работе мне часто приходится делать пулл(мердж) реквесты в основную ветку (например, в master).  
Чтобы людям, которые мержат в мастер было удобно, я следую следующему воркфлоу:  

* Отвожу ветку;  
* Работаю некоторое время над ней;  
* Пушу в удаленный репозиторий свою ветку;  
* Проверяю на серверах, что происходит то, что я задумывал;  
* Подливаю из мастера коммиты, которые успели смержить наши мастера, чтобы мой коммит можно было вмержить автоматически;  
* Соединяю все свои коммиты в один для удобста чтения и последующего разбора;  
* Делаю пулл (мердж) реквест.  

Сейчас я покажу как это сделать с помощью git.  
<!--more-->
Отводим ветку:  
```
$ git checkout -b new_branch
Switched to a new branch 'new_branch'
```
Что-то делаем:
```
$ git add . && git commit -m "first commit"
[new_branch 23a60a2] first commit
 4 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 file1
 create mode 100644 file2
 create mode 100644 file3
 create mode 100644 file4
```
Пушим в свою ветку:  
```
$ git push -u origin new_branch
Counting objects: 3, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 291 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
To https://github.com/azalio/test.git
 * [new branch]      new_branch -> new_branch
Branch new_branch set up to track remote branch new_branch from origin.
```
Предположим, что я процессе проверки выяснилось, что надо изменить некоторые файлы:  
```
$ echo "some changes" >> file1
```
И опять коммитим и пушим:  
```
$ git add . && git commit -m "second commit"
[new_branch 969d102] second commit
 1 file changed, 1 insertion(+)
 
$ git push -u origin new_branch
Counting objects: 3, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 273 bytes | 0 bytes/s, done.
Total 3 (delta 1), reused 0 (delta 0)
remote: Resolving deltas: 100% (1/1), completed with 1 local object.
To https://github.com/azalio/test.git
   23a60a2..969d102  new_branch -> new_branch
Branch new_branch set up to track remote branch new_branch from origin.
```
Во время работы в своей ветке, ваши коллеги тоже работали и в мастере оказались их коммиты. Их надо добавить к вашим, чтобы гарантировать, что ваша ветка вольется в мастер автоматически.
Переключаемся на мастер ветку:  
```
$ git checkout master
Switched to branch 'master'
```
Подтягиваем изменения:  
```
$ git pull
```
Теперь нам надо добавить коммиты из мастера в свою ветку.  
Переключаемся на нее:  
```
$ git checkout new_branch
```
Подливаем коммиты:  
```
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: first commit
Applying: second commit
```
Итого у нас есть бранч, который автоматически сможет влиться в мастер.  
Но для красоты, можно соединить ваши коммиты в один:  
```
git rebase HEAD~2
```
Где 2 - число коммитов, которые вы хотите соединить.  
Появится экран где будет написано что-то подобное:  
```
pick 82de0b1 first commit
pick dcd4606 second commit

# Rebase 7dc4c24..dcd4606 onto 7dc4c24 (2 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
```
Мне надо объединить второй коммит с первым, для этого я применяю `squash`.  
```
pick 82de0b1 first commit
squash dcd4606 second commit
```
И закрываю редактор сохраняя изменения.  
Появляется еще одно окно редактора, в котором надо написать сообщение, посвященное объединенному коммиту. Пишем, сохраняем, выходим.  
```
$ git rebase -i HEAD~2
[detached HEAD 999b456] first commit
 Date: Sat Jul 22 14:11:40 2017 +0300
 4 files changed, 1 insertion(+)
 create mode 100644 file1
 create mode 100644 file2
 create mode 100644 file3
 create mode 100644 file4
Successfully rebased and updated refs/heads/new_branch.
```
Теперь нам надо запушить с *force*, поскольку мы по сути дела переписываем историю коммитов:  
```
$ git push --force -u origin new_branch
Branch master set up to track remote branch master from origin.
Everything up-to-date
```
Итого, у наша ветка можеть быть смержена автоматически и имеет красивую историю коммитов:  
{% img /images/git-rebase-squash.png %}

Подробно про **rebase** вы можете прочитать в [этом](https://habrahabr.ru/post/161009/) посте.
