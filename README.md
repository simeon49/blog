# blog

## simeon's personal blog

## 本地开发测试
```bash
# install hugo(base on go)
brew install hugo

# run
hugo serve
```

## 部署
```bash
# 添加 upstream 远程节点
git remote add upstream git@github.com:simeon49/blog.git
# 拉取主机信息
git fetch --all

# 删除老的发布页面
rm -rf public
mkdir public
git worktree prune
rm -rf .git/worktrees/public

# checkout out gh-pages branch into public
git worktree add -B gh-pages public upstream/gh-pages

# 编译
hugo

# 将结果上传到gh-pages分支上
cd public && git add --all && git commit -am 'publish new' && git push && cd ..
```
