---
title: "git submodule delete"
draft: false
---

[git submodule]({{< relref "20230618153744-git_submodule.md" >}})

In modern git (I'm writing this in 2022, with an updated git installation), this has become quite a bit simpler:

Run:

```bash
git rm <path-to-submodule>
```

and commit.

This removes the filetree at &lt;path-to-submodule&gt;, and the submodule's entry in the .gitmodules file. I.e. all traces of the submodule in your repository proper are removed.

As the [docs note](https://git-scm.com/docs/gitsubmodules) however, the .git dir of the submodule is kept around (in the modules/ directory of the main project's .git dir), "to make it possible to checkout past commits without requiring fetching from another repository".

If you nonetheless want to remove this info, manually delete the submodule's directory in .git/modules/, and remove the submodule's entry in the file .git/config. These steps can be automated using the commands

```bash
rm -rf .git/modules/<path-to-submodule>
git config --remove-section submodule.<path-to-submodule>
```


## Reference List {#reference-list}

1.  <https://stackoverflow.com/questions/1260748/how-do-i-remove-a-submodule>
