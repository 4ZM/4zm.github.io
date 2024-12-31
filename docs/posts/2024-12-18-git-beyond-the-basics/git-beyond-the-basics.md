---
date: 2024-12-18
slug: git-beyond-the-basics
categories:
  - Git
  - Beyond the Basics
---

# Git - Beyond the Basics

How well do you know Git? Many of us use it daily and pick up the basics as we go. After a few years, you might start to think that you know Git pretty well. But there are more things in the Git man pages than are dreamt of in your philosophy...

<!-- more -->

For instance, if you are not using `git maintenance` or `worktree`, you are missing out. This blog post is for those who would like to explore Git beyond the basics.

<pre><p style="text-align: center; margin-top: 0px; margin-bottom: 4pt;">•  •  •</p></pre>

Much of the content in this post is straight out of the 2024 FOSDEM talk "So You Think You Know Git?" by the amazing Scott Chacon. Here is that talk and the follow-up "Part 2" from DevWorld 2024. I have cherry-picked the parts that I found most useful from his presentations and mixed them up with some of my own favorite Git features.

<center>
<iframe width="380" height="252" src="https://www.youtube.com/embed/aolI_Rz0ZqY?si=w-fTNQIOv5cjEjoc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<iframe width="380" height="252" src="https://www.youtube.com/embed/Md44rcw13k4?si=aPogguUMXINligKV" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</center>

Before we get started, make sure you have the latest version of git installed: [https://git-scm.com/](https://git-scm.com/) Some of the things covered in this post are relatively new. If you haven't updated your Git installation for a few years, now is the time.

Staying true to Scott Chacon's "shotgun buffet" presentation style, here is a bunch of useful Git tips for you, in no particular order. Let's go!

## git log

Let's start by setting up some nice log formatting and default arguments with a `git l` alias:

```
git config --global alias.l "log --pretty=format:'%C(#cccc00)%h %Cred%ad %Creset%<(60,trunc)%s%C(auto)%d %C(magenta)%<(15,trunc)%an' --date=format:'%y%m%d'"
```

<center>![zlib gitlog color](git-log.png){ width=100% }</center>

With formatting out of the way, let's look at some other nice `git log` options.

To figure out not just how, but when a function changed, you can use the `-L` option with git log:

```
git log -L :compress2:compress.c
```

This will list all commits (and display diffs) for when the `compress2` function in the `compress.c` file changed. Git uses a heuristic to figure out where the function starts and ends, and it might not work 100%. Sometimes you will see a few "extra" revs listed in the log, but it's still quite useful.

Another useful way to search your history is with the `-S` option. It looks for additions or deletions of a specific string:

```
git log -S API_KEY      # Show matching commits
git log -p -S API_KEY   # Show diff for matching commits
```

This will show all the commits that add or remove the "API_KEY" string. If you are horrified by what you find after running that command, perhaps consider [GitHub Secret Scanning](https://docs.github.com/en/code-security/secret-scanning/introduction/about-secret-scanning)?

## git blame

The `git blame` command shows you when and who last modified each line of a file. This is useful to understand who to ask about some piece of code or what the context was for a change.

But there is a problem. Sometimes a line changes in not so interesting ways. Perhaps someone removed a trailing whitespace (you should be using [pre-commit](https://pre-commit.com/) to avoid that by the way). Or a line might have been moved around. In these cases, the plain `git blame` will only show the most recent, i.e. the not so interesting change. The origin of the line is obscured.

There are a few things you can do to improve the situation. For starters, use: `git blame -w -C`. This ignores whitespaces (`-w`) and looks for lines that were moved or copied from other files in the same commit (`-C`).


You can additionally provide the `-C` option two or three times to detect even "deeper" moving of lines by looking further back into history. In my experience though, using more than one `-C` makes the command frustratingly slow for anything but small repos with a short history.

Here is the deep-blame alias I use:

```
git config --global alias.dblame 'blame -w -C'
```

Another useful blame feature is the ability to ignore specific revisions by adding a `.git-blame-ignore-revs` file. Without this, major formatting changes will make blame mostly useless since all blame commands will only show the formatting commit. Great if you are adopting [ruff](https://docs.astral.sh/ruff/) for your Python code.

Start using the file in your repository with:

```
git config --global blame.ignoreRevsFile .git-blame-ignore-revs
```

And this is some sample `.git-blame-ignore-revs` content:

```
d01ecec5367d4475d4a8ff08ac74088cc6423ba6 # Global clang-format with new style
68af25be0dc7d77e2cb99f3c9cabe1c96cf71149 # Switching from black to ruff
63b5fa235c5f4ec01fcb329972d2fa51682e7d75 # Consistent "west const"
```

While we are on the subject of `git blame`, there is also the `--color-by-age` option that highlights recent changes with an accent color. Useful to see recent changes "at a glance".

<pre><p style="text-align: center; margin-top: 0px; margin-bottom: 4pt;">•  •  •</p></pre>

## git branch

The git branch command will list local branches in alphabetical order. That's not helpful, at least if you have many local branches. What you want is to sort them by recent activity. Change the default:

```
git config --global branch.sort -committerdate
```

While we're at it, let's add some color and a bit more context to the branch output. This is my `git b` alias and the rich output it produces:

```
git config --global alias.b "branch --format='%(color:#cccc00)%(objectname:short) %(color:red)%(committerdate:short) %(color:bold white)%(refname:short)'"
```

<center>![branch alias](git-branch.png){ width=60% }</center>

## git push --force-with-lease

If you have worked long enough with Git, you will have discovered the tempting foot-gun `git push --force`. If you have discovered it, you might also have successfully discharged it into your foot and lost some work.

It usually goes like this: You do your work, commit and push it. Then you realize you forgot to rebase on top of main, so you do that and try to push again. This time, Git will complain since you are rewriting history on the remote. But you think that this is exactly what you wanted to do, so you try again with `git push --force`. This time, your rebased commits are up and everything is great. Or is it?

While you were rebasing, your coworker pushed a commit on top of your original change. Since you didn't fetch before rebasing, your force push obliterated their work and it is lost, like tears in the rain. Actually, the change is still locally with your coworker, so you can probably fix this pretty easily. But it's not nice to obliterate commits from other people. So let's try to avoid that.

Enter `git push --force-with-lease`. This does what you want but doesn't let you shoot yourself in the foot. It only allows the force push if the remote history is the same as the local history, i.e., it works like a regular force push as long as there are no coworker commits in the way.

## git maintenance

To keep Git performing well, there is some maintenance required. The internal data representation must be pruned and compressed.

Instead of letting these tasks piggyback on other Git commands (making them take longer), we can enable background repo maintenance:

```
git maintenance start
```

This is one of those "set and forget" type of things. Enable this in your active repos and Git will just be faster. Just do it.

## git rerere

Resolving merge conflicts is nobody's favorite pastime. So when you get the exact same merge conflict again, you wish that Git would have learned how it should be resolved and get on with it. If you enable rerere (reuse recorded resolution), Git will do just that:

```
git config --global rerere.enabled true
```

This problem is more common if you have long-lived feature branches or workflows where you do a lot of rebasing and cherry-picking. But since there is almost no cost to enable it, just do it.


## git and bash

If you are using Git from the terminal, you might consider adding a bit of ornamentation to your prompt. This is my favorite and it works well for bash.

<center>![zlib gitlog color](git-prompt.png){ width=50% }</center>

```
git clone https://github.com/magicmonty/bash-git-prompt.git ~/.bash-git-prompt --depth=1
```
Then and add this to your `~/.bashrc`:

```
if [ -f "$HOME/.bash-git-prompt/gitprompt.sh" ]; then
    GIT_PROMPT_ONLY_IN_REPO=1
    GIT_PROMPT_THEME=Single_line_Solarized
    source "$HOME/.bash-git-prompt/gitprompt.sh"
fi
```

## git add/reset/checkout -p

Sometimes (or all the time?) we find ourselves multitasking. While working on one thing, we find some minor unrelated "wart" and fix it. That's great, but it's really not part of the original task. Ideally, it should end up in a different commit.

If the "bonus" changes are made to different files, we can just stage them separately. But more often than not, the "bonus" and the original task changes are to the same file.

This is where `git add -p` comes in handy. You can stage parts of a file and leave other parts unstaged. You can also unstage with `git reset -p` and even revert changes with `git checkout -p`.

Git will step through all changes and ask you what to do with them. The caveat is that when you run the unit tests locally, they will run on both staged and unstaged changes, so you might not notice if your partial add broke something until tests are running in CI.

## git worktrees

If the casual multitasking of `git add -p` is not enough, you should consider *worktrees*. This is how you check out several branches side by side in different directories without cloning your repo multiple times. All the blobs and refs are then shared between the two directories. Here is how to check out a new branch in a new-repo-dir directory:

```
git worktree add -b new-branch ../new-repo-dir
```

## git rebase --autosquash

GitHub's pull request concept (which isn't really a Git concept) is very nice for collaborating in a repository, especially if you want to do code reviews. The squash merge strategy is the way to go in this situation.

But sometimes I find myself working alone in a Git repo and the pull request workflow is just a lot of overhead. Instead, I tend to prefer a rebase-centered workflow. Create a few commits on a branch as you work, then tidy them up with `git rebase --interactive` before pushing. This lets you combine, split, or reword commits as appropriate to make them coherent and cohesive.

Nice! But if you make a change that you know should have been part of an earlier commit, you can save yourself some work in the interactive rebase phase by committing it as a fixup directly.

Consider this situation with two commits:


```
echo "foo" > foo
git add foo
git commit -m "Adding a foo file"

"bar" > bar
git add bar
git commit -m "Adding a bar file"
```

You then proceed to work and realize that what you are doing should be part of the first commit. You could try to remember this and fix it when you do your interactive rebase later. But you don't have to. You can do a fixup and stay in your development flow. Later, you autosquash the fixup commits and do any further interactive rebasing (if there is still something to do).

<center>![fixup commit](fixup.png){ width=60% }</center>

```
echo "fubar" >> foo
git add -u
git commit --fixup=<commit hash> # 9f5e01c

git rebase --autosquash
```



## github-linguist

Maybe the least useful advice in this blog post is also not a git native topic, but a GitHub specific one. But since I know this really trigger some people's OCD, I consider it a public health service. I'm referring to the GitHub language stats for you repo...

<center>![github-linguist](linguist.png){ width=80% }</center>

Some times, the tool that GitHub uses `github-linguist` (you can run it locally) need a little help. You can provide hints in a `.gitattributes` file.

```
camera/test-images/*.h   linguist-generated
partner-libs/*           linguist-vendored
*.h                      linguist-language=C
```

This example will prevent generated header's with binary image data from being included. And it will ignore "vendored in" thirdparty libraries. It will also fix the [common issues miss-classifying C as Objective-C](https://github.com/github-linguist/linguist/blob/main/docs/troubleshooting.md#my-ccobjective-c-h-header-file-is-detected-as-the-wrong-language)

While you are editing your `.gitattributes` file, go ahead and add `* text=auto` to it to make git take care of line ending differences between Linux and Windows so you don't have to hope that all devs setup their `core.autocrlf` config in the right way.

<pre><p style="text-align: center; margin-top: 0px; margin-bottom: 4pt;">•  •  •</p></pre>
