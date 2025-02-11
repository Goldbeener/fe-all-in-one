
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







