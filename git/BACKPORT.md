As we know, for tp-qemu and avocado-vt framework, we have both downstream and upstream. The feature owner works upstream, and someone BACKPORT it to our downstream as the stable tree.
For example: tp-qemu project
```
downstream	https://gitlab.cee.redhat.com/kvm-qe/tp-qemu
upstream	https://github.com/autotest/tp-qemu
```
-------
For the acceptance test, we keep using a stable tree for stability. 
But if we need some patches upstream that fix important bugs.
# How to BACKPORT?

## Method One: The patches merged, but not BACKPORT to downstream
```
[root@dell-per750-13 io-github-autotest-qemu]# git remote add upstream https://github.com/autotest/tp-qemu
[root@dell-per750-13 io-github-autotest-qemu]# git remote -v
origin	https://gitlab.cee.redhat.com/kvm-qe/tp-qemu (fetch)
origin	https://gitlab.cee.redhat.com/kvm-qe/tp-qemu (push)
upstream	https://github.com/autotest/tp-qemu (fetch)
upstream	https://github.com/autotest/tp-qemu (push)
[root@dell-per750-13 io-github-autotest-qemu]# git fetch upstream
remote: Enumerating objects: 1776, done.
remote: Counting objects: 100% (1422/1422), done.
remote: Compressing objects: 100% (326/326), done.
remote: Total 1776 (delta 1254), reused 1148 (delta 1096), pack-reused 354
Receiving objects: 100% (1776/1776), 1.99 MiB | 726.00 KiB/s, done.
Resolving deltas: 100% (1288/1288), completed with 126 local objects.
From https://github.com/autotest/tp-qemu
 * [new branch]        dependabot/github_actions/dot-github/workflows/tj-actions/changed-files-41 -> upstream/dependabot/github_actions/dot-github/workflows/tj-actions/changed-files-41
 * [new branch]        master     -> upstream/master
[root@dell-per750-13 io-github-autotest-qemu]# git che
checkout      cherry        cherry-pick
[root@dell-per750-13 io-github-autotest-qemu]# git cherry-pick 278b246e646a0381662e60c35aab087d786e8283
Auto-merging provider/win_driver_installer_test.py
Auto-merging provider/win_driver_utils.py
Auto-merging qemu/tests/cfg/qemu_guest_agent.cfg
Auto-merging qemu/tests/cfg/win_virtio_driver_install_by_installer.cfg
Auto-merging qemu/tests/qemu_guest_agent.py
Auto-merging qemu/tests/win_virtio_driver_update_by_installer.py
[stable df35db6d] virtio-win-installer: some update of installer tests
 Author: Xiaoling Gao <xiagao@redhat.com>
 Date: Thu Mar 7 17:21:17 2024 +0800
 Committer: root <root@dell-per750-13.lab.eng.pek2.redhat.com>
Your name and email address were configured automatically based
on your username and hostname. Please check that they are accurate.
You can suppress this message by setting them explicitly. Run the
following command and follow the instructions in your editor to edit
your configuration file:

    git config --global --edit

After doing this, you may fix the identity used for this commit with:

    git commit --amend --reset-author

 9 files changed, 122 insertions(+), 72 deletions(-)
[root@dell-per750-13 io-github-autotest-qemu]#
```
## Method Two: The patches are still in someone's repository and have not been merged upstream
BACKPORT https://github.com/avocado-framework/avocado-vt/pull/3882/commits/c21fdae4a727df02f27ac15c5b4907028c5facd3
to https://gitlab.cee.redhat.com/kvm-qe/avocado-vt
```
[root@dell-per750-13 avocado-vt]# git remote -v
origin	https://gitlab.cee.redhat.com/kvm-qe/avocado-vt (fetch)
origin	https://gitlab.cee.redhat.com/kvm-qe/avocado-vt (push)
xiaoling	https://github.com/xiagao/avocado-vt/ (fetch)
xiaoling	https://github.com/xiagao/avocado-vt/ (push)
[root@dell-per750-13 avocado-vt]#  git fetch xiaoling
[root@dell-per750-13 avocado-vt]# git cherry-pick c21fdae4a727df02f27ac15c5b4907028c5facd3
```
## Method Three: The patches are still in someone's repository and have not been merged upstream, but we will use the upstream repo (not about acceptance test, just sharing)
BACKPORT https://github.com/autotest/tp-qemu/pull/4001/ to local
```
# git remote -v 
origin https://github.com/autotest/tp-qemu (fetch)
# git pull origin master pull/4001/head --no-rebase
```

## Method Four: Just a patch
BACKPORT  https://github.com/avocado-framework/avocado-vt/pull/3882/commits/c21fdae4a727df02f27ac15c5b4907028c5facd3 to anywhere, just typing:
```
# wget https://github.com/avocado-framework/avocado-vt/pull/3882/commits/c21fdae4a727df02f27ac15c5b4907028c5facd3.patch -O xxx.patch
# git apply xx.patch
```

Thanks ngu@ for reviewing this article.
Wecom
