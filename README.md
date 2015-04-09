# Git 风格指南
这是一个 Git 风格指南， 收到了来自 [*How to Get Your Change Into the Linux Kernel*](https://www.kernel.org/doc/Documentation/SubmittingPatches),
[git man pages](http://git-scm.com/doc)
和大量社区欢迎的实践的启发。

# 目录

1. [分支](#branches)
2. [提交](#commits)
  1. [消息](#messages)
3. [合并](#merging)
4. [杂项](#misc)

## Branches

* 选择*短的*和*描述性*的名字来命名分支

  ```shell
  # 好！
  $ git checkout -b oauth-migration

  # 错误，太模糊了！
  $ git checkout -b login_fix
  ```

* 来自外部的标识符也是很好的用作分支的名字，例如来自 Github 的 Issue 的
  标号。
  
  ```shell
  # GitHub issue #15
  $ git checkout -b issue-15
  ```

* 用破折号去分割单词

* When several people are working on the *same* feature, it might be convenient
  to have *personal* feature branches and a *team-wide* feature branch.
  In that case, suffix the name of branch with a slash, followed by the
  person's name for the personal branches and *"master"* for the team-wide
  branch:
* 当不同的人在同一个特性上工作的时候，这也许很方便去拥有个人的特性分支和一个面向全队的特性分支。
  在这种情况下，将斜划线作为分支名的后缀，并接着一个人的名字来代表这个的个人分支，并用 master 
  表示面向团队的分支。

  ```shell
  $ git checkout -b feature-a/master # team-wide branch
  $ git checkout -b feature-a/maria # Maria's branch
  $ git checkout -b feature-a/nick # Nick's branch
  ```

  在你变基后(rebase), 随意[合并](#merging) 你的特性分支到团队的特性分支。最终，
  整个团队的分支会被合并到 master 分支。

* 当这个分支不再需要的时候，通常也就是Merge过后，删除这个特性分支。除非你有必须的原因。
  Tip: 用下面的命令，清理干净分支。

  ```shell
  $ git branch --merged | grep -v "\*"
  ```

## Commits

* 每个提交应当只包含一个简单地的逻辑上的改变，不要在一个提交里改变多件事情。比如，如果一个 
  Patch 修复了一个Bug，又改变了一个功能的效率， 把它分开吧。

* 不要将一个改变分成多个提交。 例如一个功能的实现和他对应的测试应当在一个提交里提交。

* 快速和实时提交，小的，独立的提交更容易理解和撤销当出错的时候。

* 提交应当*逻辑上*排序的得当。例如，如果 X 提交依赖于 Y，那么 Y 提交应该在 X 前面。

### Messages

* 使用编辑器, 而不是终端去编写你的提交消息

  ```shell
  # good
  $ git commit

  # bad
  $ git commit -m "Quick fix"
  ```
  
  来自终端的提交造成了一种不好的模式： 我必须包含所有事情在一行的空间里，这通常让人不知道你到底
  在这个提交信息里到底想说什么，

  概要行，通常是第一行应当是描述性而简明的，理想情况下，他应当不超过50个字符，用大写字母开头并且
  有着迫切的表现欲。It should not end with a period since it is effectively the commit
  *title*:
  
  ```shell
  # good - imperative present tense, capitalized, fewer than 50 characters
  Mark huge records as obsolete when clearing hinting faults

  # bad
  fixed ActiveModel::Errors deprecation messages failing when AR was used outside of Rails.
  ```

* After that should come a blank line followed by a more thorough
  description. It should be wrapped to *72 characters* and explain *why*
  the change is needed, *how* it addresses the issue and what *side-effects*
  it might have.
  

  It should also provide any pointers to related resources (eg. link to the
  corresponding issue in a bug tracker):

  ```shell
  Short (50 chars or fewer) summary of changes

  More detailed explanatory text, if necessary. Wrap it to
  72 characters. In some contexts, the first
  line is treated as the subject of an email and the rest of
  the text as the body.  The blank line separating the
  summary from the body is critical (unless you omit the body
  entirely); tools like rebase can get confused if you run
  the two together.

  Further paragraphs come after blank lines.

  - Bullet points are okay, too

  - Use a hyphen or an asterisk for the bullet,
    followed by a single space, with blank lines in
    between

  Source http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html
  ```

  Ultimately, when writing a commit message, think about what you would need
  to know if you run across the commit in a year from now.

* If a *commit A* depends on another *commit B*, the dependency should be
  stated in the message of *commit A*. Use the commit's hash when referring to
  commits.

  Similarly, if *commit A* solves a bug introduced by *commit B*, it should
  be stated in the message of *commit A*.

* If a commit is going to be squashed to another commit use the `--squash` and
  `--fixup` flags respectively, in order to make the intention clear:

  ```shell
  $ git commit --squash f387cab2
  ```

  *(Tip: Use the `--autosquash` flag when rebasing. The marked commits will be
  squashed automatically.)*

## Merging

* **Do not rewrite published history.** The repository's history is valuable in
  its own right and it is very important to be able to tell *what actually
  happened*. Altering published history is a common source of problems for
  anyone working on the project.

* However, there are cases where rewriting history is legitimate. These are
  when:

  * You are the only one working on the branch and it is not being reviewed.

  * You want to tidy up your branch (eg. squash commits) and/or rebase it onto
    the "master" in order to merge it later.

  That said, *never rewrite the history of the "master" branch* or any other
  special branches (ie. used by production or CI servers).

* Keep the history *clean* and *simple*. *Just before you merge* your branch:

    1. Make sure it conforms to the style guide and perform any needed actions
       if it doesn't (squash/reorder commits, reword messages etc.)

    2. Rebase it onto the branch it's going to be merged to:

       ```shell
       [my-branch] $ git fetch
       [my-branch] $ git rebase origin/master
       # then merge
       ```

       This results in a branch that can be applied directly to the end of the
       "master" branch and results in a very simple history.

       *(Note: This strategy is better suited for projects with short-running
       branches. Otherwise it might be better to occassionally merge the
       "master" branch instead of rebasing onto it.)*

* If your branch includes more than one commit, do not merge with a
  fast-forward:

  ```shell
  # good - ensures that a merge commit is created
  $ git merge --no-ff my-branch

  # bad
  $ git merge my-branch
  ```

## Misc.

* There are various workflows and each one has its strengths and weaknesses.
  Whether a workflow fits your case, depends on the team, the project and your
  development procedures.

  That said, it is important to actually *choose* a workflow and stick with it.

* *Be consistent.* This is related to the workflow but also expands to things
  like commit messages, branch names and tags. Having a consistent style
  throughout the repository makes it easy to understand what is going on by
  looking at the log, a commit message etc.

* *Test before you push.* Do not push half-done work.

* Use [annotated tags](http://git-scm.com/book/en/v2/Git-Basics-Tagging#Annotated-Tags) for
  marking releases or other important points in the history.

  Prefer [lightweight tags](http://git-scm.com/book/en/v2/Git-Basics-Tagging#Lightweight-Tags) for personal use, such as to bookmark commits
  for future reference.

* Keep your repositories at a good shape by performing maintenance tasks
  occasionally, in your local *and* remote repositories:

  * [`git-gc(1)`](http://git-scm.com/docs/git-gc)
  * [`git-prune(1)`](http://git-scm.com/docs/git-prune)
  * [`git-fsck(1)`](http://git-scm.com/docs/git-fsck)

# License

![cc license](http://i.creativecommons.org/l/by-nc/3.0/88x31.png)

This work is licensed under a Creative Commons Attribution-NonCommercial 4.0
International license.

# Credits

Agis Anastasopoulos / [@agisanast](https://twitter.com/agisanast) / http://agis.io
