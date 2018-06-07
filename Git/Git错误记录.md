（1）git cherry-pick错误

error: Commit 5c8040488b5f587f8579599b7f1a18e3ddc5c9db is a merge but no -m option was given. fatal: cherry-pick failed

现象描述：从A分支merge到B分支，产生冲突，这时对B分支的冲突进行修改后，使用git commit -am ""形式提交，切换到C分支使用 git cherry-pick该commitid时报错

解决方法： git reset --soft A git commit -m ""

（2）warning: unable to unlink Operation not permitted

执行 git gc即可