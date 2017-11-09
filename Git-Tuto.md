# Overview

Différentes stratégies de permissions:

## Central

- tout le monde commite sur le même repo, sur une seule machine.
- Git peut être utilisé de cette manière: un commit local et un push vers trunck.

## Patch

- Seul un nombre restreint de personnes ont accès au repo central
- Les autres collaborent en envoyant des patchs (demo)

## Fork

- C'est le modèle open source
- Plusieurs copies du repo existent
- L'une d'entre elles est identifiée comme étant la source primaire
- Là aussi, on limite la possibilité de modifier la source primaire à certaines personnes, en ouvrant la possibilité à tous d'avoir l'exacte copie de la source primaire. L'altération est controlée, l'accès est ouvert.

## Branch

- Stratégie permissive car tout le monde a accès au repo en ecriture (votre cas)
- Typiquement utilisé pour des teams en interne, sans contribution externe
- Il n'y a aucune restriction, ce sont les conventions établies par la team qui définissent les modes de travail


## Initialisation d'un repo

`git init`

## Cloner un repo existant

`git clone repo`

## Add files

`git add file *.txt`

## Commit

- interactive message: `git commit`
- inline message: `git commit -m "message"`
- inline message and adding changes in tracked files: `git commit -am "message"`

DO NOT AMEND PUBLIC COMMITS

- amend a commit: say you forgot to add a file in your commit, don't reset, amend: `git add missig file && git commit --amend` (can be used to just change the commit message as well) 

# Before we start

Here is a git log command that I find very usefull and I recommend setting up as an alias in your .gitconfig.

```
[alias]
  lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
  lga = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --all
```

# Merge vs Rebase

## Merge

[Demo](https://github.com/olibob/tutogit)

To get changes into your branch ()


- in the feature branch, merge in the changes from master: `git merge master`
- add a new commit to master: `touch AA.txt && git add AA.txt && git commit -m "Add AA"` and `touch BB.txt && git add BB.txt && git commit -m "Add BB"`
- add a new commit to feature: `touch G.txt && git add G.txt && git commit -m "Add G"`
- merge master in the feature branch again
- show that the feature branch has master content, but the opposite is not true, master does not have the feature content
- merge the feature into master: because it is a fast forward, there is no merge commit
- nothing shows on the tree view (fast forward), but all files are there
- `git reflog` to see where we want to reset back to (before the fast forward merge)
- `git reset --hard HEAD@{1}`
- now do a no fast forward and get a clearer tree view: `git merge --no-ff f/Add-E`
- now the tree shows all stages

Merging is a way to get the changes from the main branch in your feature branch.

## Rebase

Pros (if merging is fast forward):
- removes unecessary merge commits
- clearer project history

Cons:
- never use it on a public branch or your team collaboration will suffer!
- you loose traceability: when upstream changes were included into the feature branch (less important)

[Demo](https://github.com/olibob/tutogit)

- go to feature branch and rebase on master: `git rebase master`
- view the tree and observe that the feature branch is now located at the tip of the master branch 
- let's go back and reset the rebase: `git reset --hard HEAD@{4}`
- now we're back were we started
- let's rebase again, but this time, interactively: `git rebase -i master`
- all commits show in the editor, nice!
- now we can re-arrange the history as we see fit
  - re-order commits (demo)
  - squash commits that belong together (demo squash)

What happens if you rebase a public branch on a feature branch?
- You change the history of the public branch.
- It will diverge from origin and errors on a push.
- if you pull from origin, you'll create a merge commit
- now you could push, efectively changing things for the team: the public branch history
- congratulations, you made everything super confusing for the rest of the team ;-)
- oh and do not force push if it's not your private branch!
- never force push on a public branch!

Tip: Cleanup rebasing

You can cleanup your commits in your private branch by rebasing on a commit (demo)

- add a new commit: `touch G.txt && git add G.txt && git commit -m "Add G"`
- now we have 3 commits in the private branch
- let's pretend F is just a fix for what was wrong in E and we want to not show this fix to the rest of the world when we contribute our code
- rebase interactively on the last commits, choose the one you want to start the rebase whith, for me it's going to be the first one, so 3 back: `git rebase -i HEAD~3`
- I'll say that F fixes up E and be done

Remark: don't go beyond what's in your private branch!

Tip:
- If you want to re-write the feature entirely, you can get the sha of the start of the feature with `git merge-base f/add-E master` and feed this sha to the rebase command `git rebase -i $(git merge-base f/add-E master)`
- Rebase and Pull Request: if you do a pull request, the branch is public! You know what this means (no more rebase).
- if you merge (fast forward after a rebase, you'll have a linear history without merge commits) (demo: )

## Checkout & Reset & Revert

This re-writes the history, use with caution!

[Demo](https://github.com/olibob/tutogit2)

We will work on master, we manage, lol :D

### Checkout

- edit the C.txt file and add garbage to it, save
- git registers the change: `git status`
- this is not what we wanted to do, so let's go back to the original state: `git checkout -- C.txt`
- now we know how to reset a change that was not yet added to the stage (file scope).

Tip:

- checkout can be used to move through commits (and branches)
  - when moving to a commit, you'll be in detached mode
  - before adding commit in detached mode, create a branch. If you don't, you'll loose your work!

### Reset

DO NOT USE RESET ON PUBLIC BRANCHES

- add garbage to the C.txt file again, save and now: `git add C.txt`
- to remove the file from stage: `git reset C.txt` (Stage scope: --soft, --mixed and --hard have no effect when reset is used with a file path)
- now let's look at the commit scope
- first go back to the original file with the right git command ...
- now, add garbage to it, save, add and commit the file: `git add C.txt && git commit -m "Change C"`
- notice the new commit: `git lg`
- Let's roll back: we want to remove the commit, keep the previous stage and file modifications: `git reset --soft HEAD~1`
- check the log and status, we did just that
- commit again: `git commit -m "Change C"`
- now let's roll back: we want to remove the commit, reset the stage, but keep the file changes: `git reset HEAD~1` (identical to `git reset --mixed HEAD~1`)
- check the log and status
- add the file and commit it again: `git add C.txt && git commit -m "Change C"`
- to remove the commit, stage and get rid of the changes: `git reset --hard HEAD~1`
- we're back to the previous commit, stage is empty, changes are gone

### Revert

When you want to remove commits on a public branch, revert the commit(s).

[Demo](https://github.com/olibob/tutogi3)

On any public branch, you will want to revert changes. This will add a new commit revert the changes from the selected commit(s). This way you do not disrupt the team workflow.

- Revert the last commit: `git revert HEAD`
- Notice how the file is empty again
- Revert the revert: `git revert HEAD` (your back with the file's content)

[Demo](https://github.com/olibob/tutogit4)

- This time, we'll revert a feature with a twist
- clone the repo

Case 1: straight merge without rebase

- in master branch: `git merge f/killer-feature`
- Add a commit to master jsut to simulate work as continued `touch F.txt && git add F.txt && git commit -m "F" F.txt`
- if you can fix it by moving forward, do so. If you do not want the code to be shared, you need to make it disappear (CI broken, making sure it does not land in a release, ...) 
- we need to re-work the feature because it's bugged. What do?
- revert the feature (if no one depends on it yet and you don't want it in the code base): `git revert <merge commit> -m 1` (m 1 because the parent we want to rebase on is master - look at the git log to view the 2 parents)
- all changes and/or files disapear, the history stays! Notice that the history stays! (bisect)
- to re-work the feature, we can go back into it: `git checkout f/killer-feature`
- edit end.txt for instance, input what ever you want to generate a change.
- add end.txt
- commit the change
- now that we changed things in the branch, how do we get it all back in our main branch?
- first revert the revert: this will re-add the borked feature to the main branch: `git revert <revert commit>`
- you can now merge the branch to integrate the fix: `git merge f/killer-feature`
- all good, back to normal workflow

Case 2: straight merge, with rebase

- clone again
- in master branch: `git merge f/killer-feature`
- Add a commit to master jsut to simulate work as continued `touch F.txt && git add F.txt && git commit -m "F" F.txt`
- revert the feature (if no one depends on it yet and you don't want it in the code base): `git revert <merge commit> -m 1`
- this time, we'll create a entirely new branch, based on the feature branch
- checkout the feature branch: `git checkout f/killer-feature`
- we are going to rebase the branch in a brand new one:
  - find the branching SHA-1: in our case B (4058709)
  - `git rebase --no-ff 4058709`
  - look at your tree: a brand new branch appeared with the same name
  - (personal) I'll rename the branch for clarity: `git branch -m f/killer-feature-rework`
  - edit end.txt, add the file, commit but this time, we will not show the fix, it'll be integrated into the previous commit: `git commit --amend`
  - switch to master: `git checkout master`
  - merge the new feature: `git merge f/killer-feature-rework`

Case 3: rebase, merge fast forward, fix
- clone again
- in feature branch: `git rebase f/killer-feature`
- switch to the feature branch
- rebase the feature branch on master (notice the new commit after HEAD on master)
- switch to master
- merge the feature branch
- `git log --oneline` : notice there is no merge commit this time
- Add a commit to master jsut to simulate work as continued `touch F.txt && git add F.txt && git commit -m "F" F.txt`
- if you need to revert now, you have to know which commits need to be reverted!!!
- `git revert 07f9de1..9d6d51e` (I know the exact commits, so let's revert them)
- now you need to include the faulty code in a new branch, to do so, we'll checkout the SHA-1 of the faulty code in a new branch
- `git co -b f/killer-feature-rework 9d6d51e`
- edit end.txt
- add end.txt
- commit it
- switch to master and revert the reverts: `git revert 4e338da..HEAD`
- switch to the feature re-work and rebase on master: `git rebase master`
- switch to master (we're back at faulty code)
- now we'll integrate the fix: `git merge f/killer-feature-rework`
- and we're done

## Log

- See stats: `git log --stat`
- See patches: `git log -p`
- See contribution overview: `git shortlog`
- One line overview: `git log --oneline`
- Graph overview: `git log --graph --oneline`
- Limit commits show by number: `git log --oneline -5`

More interesting filters:

- Limit commits show by date: `git log --graph --oneline --since "2017-10-27"` or `git log --graph --oneline --since "9 days ago" --until --"8 days ago"`
- Limit by author: `git log --oneline --author "Olivier"
- Limit by message: `git log --oneline --grep "JIRA-123:"`
- Limit by file: `git log --oneline --  controllers/users.go`
- Limit by searching for a string: `git log --oneline -S "LayoutDirectory"`
- Limit by searching for a regular expression: `git log -G ".*Dir"`
- Compare branches: `git log --oneline master..feature/X"`
- Don't show merge commits: `git log --oneline --no-merges`
- Show only merge commits: `git log --oneline --merges`