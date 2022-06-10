---
title: Collaborating - Branching and Pull Requests
teaching: 25
exercises: 0
questions:
- "How can I use version control to collaborate with other people more effectively?"
objectives:
- "Create a branch"
- "Push the branch to a repository"
- "Make a pull request to the repository"
keypoints:
- "`git checkout -b branchname` creates a branch that you can work on a set of changes"
- "`git checkout branchname` switches between branches"
- "Creating pull requests is the way most people work with Git and is good practice"
- "Before pull requests can be merged, there must be no merge conflicts"
---

Often, we want to work on a set of changes that are more complicated than what was shown in the last lesson, and without affecting other people's work. Think for example, what would happen if I made a change to code that someone else was using, but left it in a broken state. They would probably not be very happy with me! To this effect, we can use a concept called 'branches' to separate works-in-progress from the known 'good' copy of the code. In general, it's considered good practice to create a branch for every piece of work that you do, and to merge these into the 'good' version regularly.

To create a new branch, run the following command in your repository:
 
~~~
$ git checkout -b add-square-array-method
~~~
{: .bash}
~~~
Switched to a new branch 'add-square-array-method'
~~~
{: .output}

This creates a separate area for us to work in and add changes. It's a bit like how we cloned the repository in a separate place in the last lesson. However, we can also push our branch to the remote repository and keep it backed up.

We'll add a function that takes in a list, and squares all of the elements in it, returning a new array:
~~~
$ nano functions.py
$ tail -n 2 functions.py
~~~
{: .bash}

~~~
def square_array(list):
    return [item*item for item in list]
~~~
{: .output}

Then we'll commit it as normal:
~~~
$ git add functions.py
$ git commit -m "Add a method that squares a list"
~~~
{: .bash}

~~~
[add-square-array-method ea4141e] Add a method that squares a list
 1 file changed, 4 insertions(+)
~~~
{: .output}

Notice now that instead of saying 'master' here, it says 'add-square-array-method', showing us that our commit is on our branch. We've sort of glossed over it previously, but 'master' is the "default branch" in Git. In some newer git versions this has been renamed to "main", so you may see this instead.

Our commit is now saved in our local repository. If you want to, you can switch back to the "master" branch by doing:

~~~
git checkout master
~~~
{: .bash}
~~~
Switched to branch 'master'
~~~
{: .output}

If you now print the file, you'll see that our new method isn't there:

~~~
cat functions.py
~~~
{: .bash}
~~~
def sum_function(list):
    """
    A function which takes a list as an argument and
    returns the sum

    Parameters
    ----------
    list: list
        Must be floats or ints

    Returns
    -------
    float:
        The sum of the elements in list
    """
    sum = 0.0
    for item in list:
        sum += item
    return sum


def sum_product(list):
    product = 1.0
    for item in list:
        product *= item
    return product
~~~
{: .output}

# Pushing a new branch

We'll switch back to our branch again:
~~~
$ git checkout add-square-array-method
~~~
{: .bash}
~~~
Switched to branch 'add-square-array-method'
~~~
{: .output}

We can put our changes onto GitLab by pushing it. However, if you run `git push`, it won't immediately work:

~~~
$ git push
~~~
{: .bash}
~~~
fatal: The current branch add-square-array-method has no upstream branch.
To push the current branch and set the remote as upstream, use

    git push --set-upstream origin add-square-array-method
~~~
{: .output}

This message just means that the remote doesn't have a branch *called* add-square-array-method to push our work to. We can create one in our push just by running the command it gives us:

~~~
$ git push --set-upstream origin add-square-array-method
~~~
{: .bash}
~~~
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 4 threads
Compressing objects: 100% (2/2), done.
Writing objects: 100% (3/3), 346 bytes | 173.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0)
remote:
remote: To create a merge request for add-square-array-method, visit:
remote:   https://gitlab.bham.ac.uk/elvidgsm-dasp/planets-pepperr/-/merge_requests/new?merge_request%5Bsource_branch%5D=add-square-array-method
remote:
To https://gitlab.bham.ac.uk/elvidgsm-dasp/planets-pepperr.git
 * [new branch]      add-square-array-method -> add-square-array-method
Branch 'add-square-array-method' set up to track remote branch 'add-square-array-method' from 'origin'.
~~~
{: .output}

This slightly convoluted message tells us that:
 
* A new branch was created on the remote GitLab version of the repository
* Our local copy is associated with the remote branch
* We pushed that commit

New changes can be added and then pushed to the branch just by running the standard commit and push commands now. It's worth noting that `git push` only applies to the branch that you are currently working on - if you make changes on "master", then switch to the "add-square-array-method" and run `git push`, the master changes will not be uploaded to GitLab.

On GitLab, you can switch branches by usin the little drop down menu:
![Switch branch](../fig/gitlab-switch-branch.png)

# Pull (Merge) Requests

Pull requests can be used at this point to put the changes on the 'master' copy of the repository.

Go back to your repository on GitLab and click on the Merge Request bit of the menu (on a narrow screen it will hide the text, so you may need to widen your browser):
![Merge request menu](../fig/gitlab-mr-menu.png)

Click the button to create a new Merge Request:
![Gitlab new Merge Request](../fig/gitlab-new-mr.png)

Select the branch you did the extra work on, and then click 'Compare branches and continue'
![Gitlab select source branch](../fig/gitlab-select-source-branch.png)

You'll now see a page with quite a few options. You should generally write a description that tells you what the changes are. If you are working on a project with other people, 'Assignees' are people who will be implementing changes (i.e. you) and reviewers are people who will check your work for any mistakes, code that could be written more elegantly, etc. - it is very good practice to get your code reviewed before merging. 
![Gitlab merge request form](../fig/gitlab-create-mr-form.png)

After submitting the form, the merge request will be created:
![Submitted MR](../fig/gitlab-submitted-mr.png) 

You can click 'Commits' on the top for a list of the commits, but more useful is the 'Changes' button, which shows you any changes you have made:
![Changeset](../fig/gitlab-mr-changeset.png) 


Going back to the 'overview' section. We'll talk about the 'pipelines' part in the next lesson. You can press the 'merge' button to put your changes into the 'master' branch. Normall,y, when work a branch is merged, we'd consider it finished, and we can safely delete the branch because our changes are incorporated, so leave the box 'Delete source branch' ticked. The status of the merge request will change to 'Merged':
![Merged](../fig/gitlab-merged.png)

Clicking the file in the repository on the Master branch, we can see that the change has been incorporated:

![Change incorporated](../fig/gitlab-change-incorporated.png)
