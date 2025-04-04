---
title: "A walk through the Magit interface"
draft: false
---

## clone original repository {#clone-original-repository}

```bash
git clone git@github.com:yanboyang713/personalblog.git
```


## Add upstream repository {#add-upstream-repository}

1.  Open Emacs and use Magit to navigate to your local repository.
    You can open Magit status by typing:
    ```bash
    M-x magit-status
    ```
2.  Once you're in the magit-status buffer of your repository:
3.  Press M (Shift + m) to open the Remotes popup.
4.  Press a to add a new remote.
5.  You'll be prompted to enter the name of the remote, typically upstream.
6.  Next, you'll be asked for the URL of the upstream repository.
7.  Set 'remote.pushDefault' to "upsream"? (y or n): I choice n
8.  Finally, press RET (return/enter) to add the remote.


## Merge upstream repository {#merge-upstream-repository}

Ensure Upstream is Added
Before merging, ensure that the upstream repository is added as a remote, as explained in the previous [Add upstream repository](#add-upstream-repository). If it's not, add it using Magit as described earlier.


### Fetching Upstream Changes {#fetching-upstream-changes}

Open your repository's magit-status by typing M-x magit-status and navigating to your repository if needed.

Press F (Shift + f) to open the Fetch popup.

Press e to fetch from a specific remote.

```file
hint: You have divergent branches and need to specify how to reconcile them.
hint: You can do so by running one of the following commands sometime before
hint: your next pull:
hint:
hint:   git config pull.rebase false  # merge
hint:   git config pull.rebase true   # rebase
hint:   git config pull.ff true       # fast-forward only
hint:
hint: You can replace "git config" with "git config --global" to set a default
hint: preference for all repositories. You can also pass --rebase, --no-rebase,
hint: or --ff-only on the command line to override the configured default per
hint: invocation.
fatal: Need to specify how to reconcile divergent branches.
```

Type upstream (or whatever you named the upstream remote) and press RET (return/enter) to fetch the changes from the upstream repository.


### Merging Upstream Changes {#merging-upstream-changes}

After fetching the upstream changes, merge them into your current branch.

Ensure you're on the branch you want to merge into (usually master or main). You can switch to the desired branch by pressing b b and selecting your branch from the list.

Press m to open the Merge popup.

Press m again to merge.

You'll be prompted to specify the branch to merge into the current branch. To merge changes from the upstream repository, type upstream/master or upstream/main (or whatever branch from upstream you wish to merge) and press RET (return/enter).

If there have been updates in the upstream repository that you fetched, this will merge them into your local branch.


### Merge Conflicts {#merge-conflicts}

If there are merge conflicts, Magit will notify you, and you’ll need to resolve these conflicts before completing the merge.

Press e on a file in the "Unmerged" section to open the ediff merging tool, or RET to open it in the regular buffer.

Commit the resolved changes.


## Reference List {#reference-list}

1.  <https://emacsair.me/2017/09/01/magit-walk-through/>
