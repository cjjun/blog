---
title: Git commands
date: 2021-02-21 16:46:16
tags: ["git"]

---

Git commands
===

This is a brief recap of frequently used git commands and some important design ideas in git.

# 1. Initialization and status
```git
git init directory
```
or 

```git
cd directory
git init
```
Initialize a git-controlled directory. You can then use 

```git
git status
git log
```
 to check tracked status.

# 2. Working directory, stage, repository
In principle, the following pseudocode can help to explain git behavior:
```cpp
class GitNode{
    string hashkey;
    GitNode *prev;
    string CommitLog;
    map<string,snapshot> TrackFilesSnapshot;
    
    GitNode(){
        // Blank initialization
        hashkey = Generate();
        prev = NULL;
        CommitLog = "";
    }

    GitNode(GitNode &prev){
        hashkey = Generate();
        prev = NULL;
        CommitLog = "";
        copy(TrackFilesSnapshot, prev.TrackFilesSnapshot);
    }
    bool CheckStatus(){
        // Compare current files with snaposhot recorded
    }

    bool Add(string filename){
        /* Check if filename exists in TrackFilesSnapshot.
        if so, update snapshot with current file, 
        otherwise insert the snapshot.
        */
        TrackFilesSnapshot[filename] = f[filename]

    }
    bool commit(mode m, string s){
        if(m == 'm')
            CommitLog = s;
        
        else if(m == 'am'){
            map<string,snapshot>::iterator *it;
            for(it = TrackFilesSnapshot.begin(); it != TrackFilesSnapshot.end(); ++it){
                if( diff(f[it->first], it->second) )
                    Add(it->first)
                CommitLog = s
            }
        }
        else
            ...
        // Next, BranchHead = this;
        return true;
    }
}
class Branch{
    string BranchName;
    GitNode *HEAD = NULL;

}

class GIT{
    GitNode *HEAD;
    Branch *branch;
}
```
The following codes can track, add to cache, and update
```git
git add filename            // Track and update cache
git commit -m 'message'     // Add cache to working directory
```
**Note:** ```git add``` only create the snapshot when it's called. If further updates are made, git will only adopt previous snapshot and stash it. Updates will be marked as revised.

To update snapshot of tracked (previously or currently) files, use 
```git
git commit -am 'message'
```

 # 3. Branch and checkout
 Personally, I would rather to consider node before considering branch. In fact, branch is a later developed trick to make chains more interpretable.

 Suppose you begin with a branch head pointer ``BR_HEAD`` and global head pointer ``HEAD`` in the same GitNode. At the same time, the system has already beginned editing a temporary uncollected GitNode. ``Add`` operation will change snaposhot of it, and commit operation will make it finished and corresponding branch linked-list will be extended.

 To create a branch (default branch is main) since current location, use
 ```git
git branch branch_name
git checkout branch_name
 ```
or simply
 ```git
git checkout -b branch_name
 ```
If you want to create branch for some branches, use
```git
git branch branch_name some_branch
```

Besides, ```some_branch``` can be any pointer, of branch, remote branch, etc. A useful usage is:
```git
git checkout -b dev origin/dev
```
This will grab ```dev``` branch of remote space and create a branch ```dev``` at local workspace.

**The simple rule is, branch is just a ```reference```, i.e. a terminal-pointer of linked-list. It's introduced to manage complexed DAG with clarity.**

Checkout can not only jump to any branch, but also jump to any gitnode given hash reference. This concurs with previous assertion that branch is merely a pointer. To mange this detached node, git will create temporary detached branch to manage it as branch.

Whenever ``checkout`` happens that ``HEAD `` leaves current branch, the temporary detached branch will be deleted. 

# 4. Reset, revert, rebase, restore
## 4.1. reset
``reset`` is operation regarding ``BR_HEAD``. Tracking backwardly from ``BR_HEAD``, a complete linked-list can be found. ``reset`` is manipulating this linked-list, plus the temporarily processing GitNode, which has not yet been attached to branch.

There are three modes of ``reset``, which are:
```
--soft 
--mixed (default) 
--hard
```

``--soft`` moves ``BR_HEAD`` without changing temporary GitNode.

``--mixed`` moves ``BR_HEAD`` and copies ``TrackFilesSnapshot`` to temporary GitNode from ``BR_HEAD``. This leads to discard of currently uncommited operations.

``--hard`` moves ``BR_HEAD``, resets all files from snapshot of ``BR_HEAD``, then create a new temporary GitNode from ``BR_HEAD``.

There are several use syntax:
```
HEAD^       (Move back 1 step)
HEAD~n      (Move n steps back)
HEAD@{n}    (Move n step(s) forward)
```
## 4.2. revert
```
revert -n [target]
```

``revert`` is different from  ``reset``, it basically computes change between target and its predecessor
(i.e. ``target^`` - ``target``), the difference will be applied to current cache.

Suppose the difference (inversely) includes three parts: ``add``, ``remove``, and ``revision``. ``add`` and ``revision`` will be copied to cache, and conflicts will be needed for solutions. The interesting part is ``remove``. 

If the files have not been changed since target version (we mean temporary gitnode), they will be deleted without doubt. If somethings happens, it will not be removed, and revert will fail. In this case, we have to delete those files manually.

## 4.3. rebase
``rebase`` can update previous commits in more complex ways. A typical usage is merging commits.

```
pick xxxx
fixup xxxx
squash xxxx
```
## 4.4. restore
```git
git restore [target]    // Restore files to snapshot in TrackFilesSnapshot
git restore --staged    // Remove file from tracked directory but not change its content)
```

**To understand how git recognize difference, check the link [Myer algorithm](https://cjting.me/2017/05/13/how-git-generate-diff/)**.
# 5. Remote warehouse
Add remote source ``origin``
```git
git remote add origin git@github.com:/xxx/xxx.git
```

Push current branch and set associate it with upstream
```git
git push -u <remote> <branch>
git push --set-upstream <remote> <branch>
```
correspongdingly, pull remote branch to current branch
```git
git pull <remote> <branch> 
git pull --set-upstream <remote> <branch>
```
``fetch`` serves the same function:
```git
git pull <remote> <branch> 
git fetch --set-upstream <remote> <branch>
```
The difference is that ``fetch`` returns pointer ``FETCH_HEAD`` for youself to dermine, while ``pull`` automatically merge ``FETCH_HEAD`` with current branch.

Once you set upstream, all operations can be simplified as  ``git push``, ``git pull``. Please always save your commit before executing ``pull`` or ``merge`` operation.

In local remote management, you can use
```git
git remote add <nickname> <remote-url>
git remote rename <old_name> <new_name>
git remote remove <nickname>
```

Furthermore, if you want to operate on remote warehouse directly, here are some commands:
```git
git push <remote> --delete <branch> // delete remove branch
git remote prune <remote>  // prune a local branch whose upstrea has been deleted
git fetch -p    //equivalent to commands above
```
To rename a remote branch, tracked by local branch with same name, use
```git
git push <remote> --delete <old_name> 
git branch -m <old_name> <new_name>
git checkout <new_name>
git push -u <remote>  <new_name> 
```
If the ``<old_name>`` is the default branch, you have to set another branch as default.

<!-- # 5. To be continued... -->
