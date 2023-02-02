```
git tag -a v0.0.1 -m "v0.0.1发布" # 打tag

git push origin v0.0.1

1、在控制台打印出当前仓库的所有标签：git tag

2、搜索符合模式的标签：git tag -l 'v0.0.*'

3、创建附注标签：git tag -a v0.0.1 -m "v0.0.1发布"

4、删除标签：git tag -d v0.0.1

5、查看标签的版本信息：git show v0.0.1

6、指向打v0.0.2标签时的代码状态：git checkout v0.0.2

7、将v0.0.1标签提交到git服务器：git push origin v0.0.1

8、将本地所有标签一次性提交到git服务器：git push origin –tags
```

