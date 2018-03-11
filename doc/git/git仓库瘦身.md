#git 仓库瘦身


首先查找仓库记录中的大文件,只输出前5个
git rev-list --objects --all | grep "$(git verify-pack -v .git/objects/pack/*.idx | sort -k 3 -n | tail -5 | awk '{print$1}')"

然后通过filter-branch来重写这些大文件涉及到的所有提交（重写历史记录）：   
git filter-branch -f --prune-empty --index-filter 'git rm -rf --cached --ignore-unmatch {#要删除的大文件}' --tag-name-filter cat -- --all


删除完毕后   
rm -rf .git/refs/original/   
git reflog expire --expire=now --all   
git fsck --full --unreachable   
git gc --aggressive --prune=now  
git push --force 对应远程分支 现在分支 (master分支需要在gitlab上保护去除)




