fgci-ansible contribution guidelines
-------------------------

Git Branches
--------

Configure upstream:

<pre>
cd fgci-ansible 
git pull
git checkout devel
git branch --set-upstream-to=origin/devel devel
git pull
git checkout master
</pre>

Git Workflow:
---------

 - devel: After the work is commited to devel, run testing. Merge devel into master when testing is done.
 - master: The stable branch

Syncing changes done in devel:
<pre>
git checkout master; git pull
git merge --ff-only devel

#If the merge was successful then make:

git push
git checkout devel
</pre>

Git push where appropriate

Releases / Tags
--------

After syncing the changes from devel to master we should make a release on github.

Every time a new change is made to the devel branch - a CSC person should mention that change in a draft 
release note. We anyway should describe the changes in the release, so documenting the release in the
release note when making the change is better. If the change is not tested (yet) this should also be
mentioned in the draft release note.

Release Naming Convention: vx.y.z Adjective Snakename 

https://en.wikipedia.org/wiki/List_of_snakes_by_common_name


Pull Requests
-------------

 - Make a fork of this repo (or the ansible-role repo if it's for a role)
 - Create a new branch in your fork called "new_feature"
  - This is important, don't make the updates in the master branch of your local repo.
 - Commit and push to this new_feature branch in your repo
 - Create a pull request with the fix

Adding new roles for ansible-pull
---------------------------------

As of writing (20160906) if we add a new role to the ansible-pull playbook (local.yml) - it will cause ansible-pull to fail on the compute nodes _until_ the sites have updated their local git mirror. For this reason the process for adding a new role to ansible-pull is:

 1. Add the new role to requirements.yml, preferably on Monday/Tuesday
 2. Inform the sites via e-mail/social media that in a week we'll add a new role and that they should update their local git-mirror (run fgci-install tag/role on install node, see the tools/pullReqs.sh helper script)
 3. On the following Monday add the role to local.yml

We have an open issue about this: https://github.com/CSC-IT-Center-for-Science/ansible-role-fgci-install/issues/11

Testing the changes:
----------------

In general we make updates in the devel branch, improve them and test them there.

List of scenarios we aim to test/consider:
 - applies to ansible-pull
  - test the changes to ansible-pull by making the change to the devel branch on github
 - applies to ansible-push
  - test this by making local changes in the fgci-ansible working directory and running the role, first with --check --diff
 - applies to reinstalls / brand new clusters
  - test this by reinstalling the cluster
  - think of the order (available in site.yml) in which nodes are installed, install node before login node for example
 - applies to partial reinstalls, like keeping a filesystem or sql database
  - test this by reinstalling the cluster
 - applies to sites using hardware that's not only from the initial FGCI Dell Harware?
  - ask the sites for assistance


Ansible Roles
-------------

These are suggestions for requirements of roles that the fgci-ansible orchestrator playbook uses.

 - **name** must start with: ansible-role-* - if it's an external role, change the name in requirements.yml
 - **meta:** there must be a meta/main.yml and should be filled in as much as possible
 - **independence:** the role should work by itself - it shouldn't depend on other roles. For example if a role installs software from a repo that is not installed by default, the repo and software should be added in that role.
 - **defaults:** if the role uses any variables, they should be defined and have defaults in defaults/main.yml
 - **idempotence:** if you run the same role twice it should not change anything on the second run
 - **readme:**  README.MD should contain description
 - **variables names:**  the variable namespace is global, so try to not use variables called "packages" or "installed". role_name_variable_name is better.
 - **automatic testing:** add the role to travis. Some files needed in the repo - For example see https://github.com/CSC-IT-Center-for-Science/ansible-role-cvmfs
 - if it's an optional role, the role should not do anything by default. Suggestion is to force users that want to use the role to set something like "autofs_enabled: True". This is how we add site-specific roles to the playbooks.

CSC tasks after adding a new role/repo:
 - **enable automatic testing:** enable the repo in https://travis-ci.org/CSC-IT-Center-for-Science/
 - **notifications:** add the new github repo to the #fgci channel in flowdock
 - **backlog management:** add the new github repo to https://waffle.io/CSC-IT-Center-for-Science/fgci-ansible/

Ansible role suggestions (not mandatory):

Make it work on also non-EL systems. Don't use yum module if possible, use something like:

<pre>
- name: install rdma
  action: 
    module: "{{ ansible_pkg_mgr }}" 
    name: rdma 
    state: present
 </pre>

If the role is only for EL, have this in ansible-role-name/tasks/main.yml:

<pre>
- include: redhat.yml
  when: ansible_os_family == "RedHat"
</pre> 

Then do the tasks in redhat.yml 

command/shell tasks should be avoided, but if they are used and they are only used to write the output into a variable - they should have "always_run: True" so that the role works when running the playbook it with --check. They should also have a "changed_when: False" so that they aren't listed as changed. 

When removing tasks, try to set the state to "absent" rather than just removing the task - like this https://github.com/CSC-IT-Center-for-Science/ansible-role-cuda/commit/076296dc24c3f4fbc85e15a81432a7d2bd3fa714


