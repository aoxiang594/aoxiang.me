# 记录一下常用Git命令

项目中SVN用的比较多，最近用Git也只是一直停留在`commit`,`push`。记录一下用过的命令。

**回滚整个版本库到指定版本**

```
#先查看指定版本的版本号
git log
git reset --head commit-id
git reset --header 82bb80c9ea80b35377ac018f86dfc220670e9699
```

**回滚部分文件到指定版本**

```
git checkout commit-id  files
git checkout 82bb80c9ea80b35377ac018f86dfc220670e9699 app/Http/Controllers/Controller.php
```
# 测试合并
> 我是一个远程库
