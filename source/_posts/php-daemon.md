---
title: PHP程序守护进程化
date: 2017-10-13 11:58:12
tags:
- PHP
categories:
- PHP
- 守护进程
---

　　一般Server程序都是运行在系统后台，这与普通的交互式命令行程序有很大的区别。glibc里有一个函数daemon。调用此函数，就可使当前进程脱离终端变成一个守护进程，具体内容参见man daemon。PHP中暂时没有此函数，当然如果你有兴趣的话，可以写一个PHP的扩展函数来实现。

PHP命令行程序实现守护进程化有2种方法：

<!--more -->

### 使用nohup

```bash
nohup php myprog.php > log.txt &

```

这里就实现了守护进程化。

- 单独执行 php myprog.php，当按下ctrl+c时就会中断程序执行，会kill当前进程以及子进程。

- php myprog.php &，这样执行程序虽然也是转为后台运行，实际上是依赖终端的，当用户退出终端时进程就会被杀掉。

### 使用PHP代码来实现

#### 代码如下
```php

function daemon()
{
    if(($pid1 = pcntl_fork()) === 0)
    //First child process
    {
        posix_setsid(); //Set first child process as the session leader.

        if(($pid2 = pcntl_fork()) === 0)
        //Second child process, which run as daemon.
        {
            // todo. here is your code.           
        }
        else
        {
            //First child process exit;
            exit;
        }
    }
    else
    {
        //Wait for first child process exit;
        pcntl_wait($status);
    }
}

daemon();
sleep(1000);
```

用上面代码即可实现守护进程化，当你的PHP程序需要转为后台运行时，只需要调用一次封装好的函数daemonize()即可。
注：这里没有实现标准输入输出的重定向。

#### 如何理解?

如果不理解这段代码的话，可能需要了解一些Unix系统进程的知识。

##### 系统调用

Unix系统由用户空间和内核组成，系统调用允许用户空间的程序通过内核与计算机硬件交互。不同的语言可以通过不同的函数使用系统调用，例如系统调用fork在C中的函数是fork()，在PHP中的函数是`pcntl_fork()`. PCNTL是Process Control的缩写，PHP中与进程有关的函数都以此开头。

##### 子进程

系统中的每个进程都有对应的父进程，进程的ppid标识符即为其父进程的pid。当子进程执行完毕之后，父进程应该通过wait请求这些信息，否则内核会一直保留子进程的状态信息。如果父进程没有调用wait来等待它，子进程就会变成僵尸进程。
如果父进程先于子进程结束，子进程通常并不会受影响，而会一直执行下去。父进程结束时系统会扫描其子进程，并将其ppid置为1，成为init进程的子进程。

##### fork

系统调用fork可以使进程衍生一个子进程。两个进程除了pid与ppid之外一模一样。PHP中可以通过`pcntl_fork()`实现fork，执行完该函数之后，代码会在父子两个进程中继续执行下去，`pcntl_fork()`在子进程中的返回值为0，在父进程中的返回值为子进程的pid。因此`pcntl_fork()`经常被放在if条件里，以实现父子进程执行不同的代码，例如：

```
if(($pid1 = pcntl_fork()) === 0)
{
    //子进程执行此处的代码
}
else
{
    //父进程执行此处的代码
}

```
##### 进程组

每个进程都属于某个组，进程组是一个相关进程的集合，通常是父进程与其子进程。每个进程组都有一个整数id，可以通过`posix_getpgrp()`函数获得，通常进程组的id和进程组leader的pid相同。当终端收到终止信号(Ctrl-C)时，会转发给进程组中的所有进程。因此，如果你在终端执行一个PHP脚本，并在脚本中fork了子进程。当在终端中按Ctrl-C终止脚本执行时，子进程会同时被中止。

##### 会话组

会话组是进程组的集合，当在shell中执行命令：`git log | grep shipped | less`时，每个命令都有一个进程组，三个进程组属于同一个会话组。当终端收到Ctrl-C时，发送给会话leader的信号会被转发给该会话组的每个进程组，然后再被转发到进程组中的所有进程。

##### setsid

setsid系统调用会使衍生进程成为一个新会话组leader（同时也是新进程组leader），并返回新建的会话组id。在PHP中对应的函数是`posix_setsid();`

##### 为何要fork两次。

这个问题就是这段代码中最难解释的部分了，有一些人认为只fork一次就够用了，剩下的观点分为两种，一种是两次fork是为了杜绝守护进程控制终端的可能，另一种认为两次fork是为了避免产生僵尸进程。

- 防止守护进程控制终端

如果要生成一个守护进程，我们应该确保它不会控制终端，完全在后台运行。而只有会话组leader才能控制终端，因此就要确保守护进程不是会话组leader，这就是第二次fork的作用。

第一次fork并setsid使进程脱离当前终端，第二次fork确保进程永远不会控制终端。

- 避免守护进程成为僵尸进程

网上另有一种说法是，父进程的存在并非只为生成守护进程。如果只fork一次，且父进程不退出，那么守护进程终止之后就会成为僵尸进程。因此要生成一个子进程来fork出守护进程，fork出守护进程之后子进程就exit，以便将守护进程交由init进程托管。我不是很倾向于这种说法，因为很多fork两次的代码中都不存在「父进程生成守护进程后，还有自己的事要做，它的人生意义并不只是为了生成守护进程」这种事。

关于这个问题，也可以参考[stackoverflow上的一个讨论](http://stackoverflow.com/questions/881388/what-is-the-reason-for-performing-a-double-fork-when-creating-a-daemon)。