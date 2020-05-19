# Welcome

Our aim is to run the [BadgerTrap][badgergithub] tool on the latest kernel version and get some information out of it. The officially supported version is `3.21.3`. This version fails to boot with the latest Qemu system. We are following the instructions mentioned [here][qemu_tut] to run a custom kernel on the Qemu system.


The oldest kernel that successfully runs is `3.16.35`. However, there are couple of changes in the code since the last BadgerTrap supported kernel, and a copy-paste job will not do. We need to understand the working, and the code, to successfully port it.

!!! todo
    1. [X] Run the 3.16.35 kernel. Untouched.
    2. [ ] Study the old kernel and see the difference.
    3. [ ] Understand the page table working completely.
    4. [ ] How can we get access to the entries within the page offset.
    5. [ ] Run the vanilla version of the BadgerTrap
    6. [ ] Run the modified version



[badgergithub]: https://github.com/sonarsys/BadgerTrap
[qemu_tut]: http://nickdesaulniers.github.io/blog/2018/10/24/booting-a-custom-linux-kernel-in-qemu-and-debugging-it-with-gdb/
