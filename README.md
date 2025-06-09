#Yanhuo075' blog
echo "# blog" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/yanhuo075/blog.git
git push -u origin main

附仓库github仓库命令:
git add .
git commit -m "commit update at $(date '+%Y-%m-%d %H:%M:%S')"
git push -u origin main
