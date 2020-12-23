#git
## 初始化配置 

```
//全局global 本地 local
 git config --global  --list 查看配置项
 git config --global user.name  'xxx'
 git config --global user.email 'xxx'
```
##  免密码 
ssh-keygen -t rsa -C "这里换上你的邮箱"
在指定的保存路径下会生成2个名为id_rsa和id_rsa.pub的文件：
github 选择SSH and GPG keys 配置即可

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

## 恢复单个文件
    git log '文件路径'
    git checkout 'logid' '文件路径'
