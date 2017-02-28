#移除GitHub上的错误提交

假设，你在GitHub上托管了项目，但是发现你最近的一次提交出了错。你想要撤销这个提交，那么应该怎么做呢？

假设`5c24e50`是这次错误提交的SHA，`58c4e50`是离这次提交最近的一次提交，在命令行下输入如下命令即可从`5c24e50`回退到
`58c4e50`:
    
    git push origin +58c4e50:master
    
原文链接见[这里](http://stackoverflow.com/questions/448919/how-can-i-remove-a-commit-on-github)。
