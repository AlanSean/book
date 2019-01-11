#git

## 删除某文件历史记录
```
    删除文件在本地的提交历史:
    git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch 文件地址' --prune-empty --tag-name-filter cat -- --all
    将修改提交到远程仓库，远程仓库的commit历史将被修改
    git push origin --force --all
    如果tag中也需要删除敏感数据，则执行
    git push origin --force --tags

```
## 取消合并
```
    git merge --abort #如果Git版本 >= 1.7.4
    git reset --merge #如果Git版本 >= 1.6.1
```
## 撤销commit
```
    git status
    git reset HEAD  文件名
```
## 取消文件的修改
`git checkout -- 文件名`
