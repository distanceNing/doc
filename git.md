
git submodule update --init --recursive

设置默认密码

````
git config --global credential.helper store
````



https://git-scm.com/book/zh/v1/Git-%E5%88%86%E6%94%AF-%E4%BD%95%E8%B0%93%E5%88%86%E6%94%AF

合并dev分支到当前分支上：
      
```
git merge dev
```


```
取消当前分支修改
```
修改分支名称：

```
git branch -m oldName newName
```


git clean -d -fx

删除远程分支：
 git push origin --delete zhihong

删除本地分支：
git branch -d 

token登录：

````
git remote set-url origin https://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@github.com/distanceNing/testapp.git
````

取消add
 git rm -r --cached
取消commit
 git reset --soft HEAD~1

创建并切换到分支：

    git checkout -b iss53
切换分支：

    git checkout dev


拉去远程分支：

````    
git checkout -t origin/dev 该命令等同于：git checkout -b dev origin/dev
````


本地分支push到远程仓库：


### merge 合并多个commit

```
git merge --squash

git commit -m "合并信息"
```


### 回退版本：

```
    git reset --hard c47742c98dce390fea0bca44f21d4e70fa7b9eab
    git push origin HEAD --force
```
### 相对引用

* 使用 ^ 向上移动 1 个提交记录
* 使用 ~<num> 向上移动多个提交记录，如 ~3

# 本地栈式提交



git cherry-pick <提交号>...