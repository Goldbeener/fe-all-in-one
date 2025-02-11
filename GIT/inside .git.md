
![[Pasted image 20240903103554.png]]



HEAD

branch

commit

trees

blobs



文件状态
Untracked
Unmodified
Modified
Staged


UnTracked ---  git add . --- Staged
Modified --- git add .  --- Staged
Modified --- git restore .  --- Unmodified
Unmodified --- edit  --- Modified
Staged --- git commit. --- Unmodified
Staged --- git restore --staged --- Modified


工作区
暂存区
本地仓库
远端仓库


git restore --staged
仅将文件从暂存区移除，不影响HEAD，不改变最近的提交
文件修改仍然存在，工作区还有

git reset --soft  <commit>
HEAD移动到指定提交
暂存区保持原样
工作区保持原样

用于撤销最近的提交，可以重新提交


git reset --mixed <commit>
默认行为，也就是不带参数时的行为
HEAD移动到指定提交
暂存区也被重置，暂存区受到影响，跟HEAD一样回退到指定提交
工作区不受影响，还有相关改动

用于回退提交，并重新git a







