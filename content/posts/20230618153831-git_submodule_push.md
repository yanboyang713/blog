---
title: "git submodule push"
draft: false
---

[git submodule]({{< relref "20230618153744-git_submodule.md" >}})

```bash
$ cd your_submodule
$ git checkout master
git add *
$ git commit -a -m "commit in submodule"
$ git push
$ cd ..
$ git add your_submodule
$ git commit -m "Updated submodule"
```


## Reference List {#reference-list}

1.  <https://stackoverflow.com/questions/5814319/git-submodule-push>
