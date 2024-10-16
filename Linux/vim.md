# Vim操作大全

## 0 项目积累代码

1. 自动登入ftp

```shell
#!/usr/bin/expect
spawn ftp 115.236.112.89
expect "*duankui*"
send "****\r"
expect "*Password*"
send "****\r"
expect "ftp*"
send "cd ****\r"
interact
expect eof
```

2. 自动登入

```
/usr/bin/expect /nas/dk/code/scripts/git_pull1.sh
/usr/bin/expect /nas/dk/code/scripts/git_pull2.sh
/usr/bin/expect /nas/dk/code/scripts/git_pull3.sh

#curdir=$(pwd)
#cd $curdir
#echo $curdir
spawn git fetch --all
expect "Username*" {send "duankui\r"}
expect "Password" {send "Xxdk0808\r"}
interact

#curdir=$(pwd)
#cd $curdir
#echo $curdir
spawn git reset --hard origin/master
interact

#curdir=$(pwd)
#cd $curdir
#echo $curdir
spawn git pull
expect "Username*" {send "duankui\r"}
expect "Password" {send "Xxdk0808\r"}
interact
```

## 1 Vim 实用大全



| x    | 删除光标处字符                     | u    | 撤销命令操作                   |
| ---- | ---------------------------------- | ---- | ------------------------------ |
| dd   | 删除单行（+ndd）                   | \>G  | 增加从当前行到文档末尾处的缩进 |
| f    | 查询当前行的某一字符               | ;    | 跳到当前下一个查询字符         |
| s    | 删除光标处字符<br />并进入插入模式 |      |                                |
| n    | 查询到下一个匹配项                 |      |                                |
|      |                                    |      |                                |
|      |                                    |      |                                |
|      |                                    |      |                                |
|      |                                    |      |                                |
|      |                                    |      |                                |



### 1.1 认识 . 命令

> . 命令可以让我们重复上次的修改，它是Vim中最为强大的多面手。

