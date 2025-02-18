# Forking Epsilon as a non-committer with Git

This article demonstrates how you can extend Epsilon and continue to receive source updates from the main development branch whilst also being able to commit changes to your own repository using the git version control system. The idea is to clone the Epsilon repository, make a branch and then set the remote for that branch to the master branch of your private repository. This allows you to maintain the history of Epsilon and so later on your changes can be merged into the main development branch if and when you gain committer priviliges. For the rest of this article, we shall refer to the main Epsilon project as "origin". Specifically, we are referring to the "master" branch of Epsilon. For the extension project, we shall call it "fork". Specifically, "forkbranch" refers to the branch name of the extension project, whilst "forkrepo" refers to the respository name of the extension project. We will assume that the "master" branch of forkrepo is used to host the forkbranch for simplicity.

If you have already set up your repository and have some content on it, you should back up all of your work before proceeding, as the following steps involve resetting your repository. The steps are as follows:

- Create a new folder and cd to it: `git init`
- Create a blank text file in that folder

```
git add .
git commit -m "Reset repo"
git remote add origin <forkrepo url>
git push -u --force origin master
```
- Delete the folder you created.

Your repository should now be completely clean. Here are the main steps:

- Clone the main project repository: `
git clone git://git.eclipse.org/gitroot/epsilon/org.eclipse.epsilon.git`
- Create and switch to a new branch: `git checkout -b forkbranch`
- Add the remote repository for the branch: `git remote add forkrepo <fork url>`
- Set the remote repository you're going to upload to: `git branch --set-upstream-to=forkrepo/master`
- Confirm that the following outputs "forkrepo/master": `git rev-parse --abbrev-ref --symbolic-full-name @{u}`
- Set the default push branch to be the same as the tracking (in this case, it will be master): `git config push.default upstream`
- If you have already initialized your fork repository, you need to get the files first.: `git fetch forkrepo master`
- This is the crucial step, as it allows you to merge your fork repository with the main project's repository: `git pull --allow-unrelated-histories`
- Add the files from the main project to your commit: `git commit -m "Original, unmodified files"`
- This will upload all the files to your fork repository: `git push --repo=forkrepo`
- Confirm that the following outputs "On branch forkbranch Your branch is up-to-date with 'forkrepo/master'. nothing to commit, working tree clean": `git status`
- Set the default push to your fork repository: `git config remote.pushDefault forkrepo`

For more information on using Git, please refer to the
[documentation](https://git-scm.com/documentation).
