---
layout: mypost
title: 容器背后的Kernel Namespace 
categories: [随笔]
---

# 容器背后的Kernel Namespace 
容器技术背后使用的是 kernel namespace和cgroups技术^[https://docs.docker.com/get-started/]，用来隔离容器内外的环境

# Namespace 方式
1. mount：可以隔离挂载点，namespace内部的挂载点在外部看不到
2. UNIX Timing-Sharing System(uts)：可以使内外部的hostname互不影响
3. Interprocess Communication(ipc)：namespace隔离了共享内存（SHM），但可以用相同的 identifies for shared memory segment，以此来掭两块不同的内存区域
4. PID namespace(pid)：不同namespace的进程可以有相同的pid，PID namespace还可以嵌套
5. Network(net)：每个 network namespace 都有一套自己的IP地址、路由表等，可用于实现SDN（Software Defined Networks）
6. User ID(user)：用户id和组id在namespace内外可以不同，即可以在user namespace之内取得root权限（可能fakeroot就是在用这个？）
7. control groups(cgroups)：可以用来限制资源，比如实现一个out-of-memory(OOM) killer

# 附：讨论 chroot 方式
chroot无法用于容器，比如在chroot后，用ps还是能看到外界的进程，也能杀死外界的进程