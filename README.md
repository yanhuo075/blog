#Yanhuo075' blog
echo "# blog" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/yanhuo075/blog.git
git push -u origin main

附录
更新文章命令：
hexo clean 或简写 hexo c
hexo generate 或简写hexo g

1.添加wiki文档步骤：

①添加项目id配置文件source/_data/wiki/xx.yml
②在source/_data/wiki.yml文件中添加项目id
③创建文档目录source/wiki/xx
④编写md文章source/wiki/xx/xxx.md


2.仓库github仓库命令:
git add .
git commit -m "commit update at $(date '+%Y-%m-%d %H:%M:%S')"
git push -u origin main
