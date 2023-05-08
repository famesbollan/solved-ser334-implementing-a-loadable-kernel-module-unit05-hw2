Download Link: https://assignmentchef.com/product/solved-ser334-implementing-a-loadable-kernel-module-unit05-hw2
<br>
Implementing a Loadable Kernel Module

Summary: In this homework, you will implement a loadable kernel module that uses Linux data structures to display details about the processes executing in the kernel.
<h1>1 Background</h1>
The kernel is a program that forms the core of a computer’s operating-system, with control over everything in the system. It performs tasks such as running processes, allocating resources, interfacing with hardware, and all other tasks that happen behind the scenes in kernel space . In contrast, everything the user does is executed in user space . This separation prevents user data and kernel data from interfering with each other and causing instability or security issues.

Kernel modules are pieces of code which provide functionality to the kernel. The word loadable is pre xed to kernel module to show that it eases the process of integrating the module with the kernel. Typically, if functionality is to be provided in the kernel, it has to incorporated by compiling the kernel with the proposed changes. Loadable Kernel Modules (LKM) ease that process by providing the facility to integrate with the existing kernel instead of requiring kernel re-compilation. The best way to start is with the sample LKM provided on the textbook’s <a href="http://os-book.com/">website</a> (also posted on Canvas), and then add your own functionality.

In this assignment you will write a LKM for the Linux kernel that displays certain details of the processes (along with its parent and children) executing in the kernel. As an important step to implementing the program, you should review the section on include les to get an idea of what functionality is available. From there, start to develop the actual source code, while reading the documentation for any functions you used. Much of the documentation exists as source code with the Linux kernel, meaning that you will need to read the code yourself. The assignment source code should be relatively short but uses features you will have to research on your own (such as passing a value to a LKM, or how to manipulate the list of running processes). Expect that a signi cant part of your time on this assignment will be spent reading/researching/understanding LKM documentation and reviewing kernel source code les (such as linux/sched.h, and linux/list.h). The website for Linux kernel source is <a href="https://www.kernel.org/">www.kernel.org/.</a> Any lines numbers mentioned in this document refer to Linux 5.11. The speci c version of the Linux kernel on your machine may di erent slightly.

This document is separated into ve sections: Background, Running a LKM, Requirements, Include Files, and Submission. You have almost nished reading the Background section already. In Requirements, we will discuss what is expected of you in this homework. In Running a LKM, a basic demonstration of an LKM is given. In Include Files, we discuss several header les that include functionality related to le and directory manipulation. Lastly, Submission discusses how your source code should be submitted on Canvas.
<h2>1.1 Accessing Kernel Source Code</h2>
To nd the version of your kernel, use the command uname -r . Below the version is shown as 5.11.034-generic. The screenshot comes from a VM running an up to date (9/28/21) install of Xubuntu 20.04.3 (which can be checked by running lsb_release -a ). The bit of text after 5.11 (the major and minor version information) indicates the patch version. Depending on how up to date your machine is, you may see a more recent kernel version. When you submit your write up, you will be asked to list the version of the kernel you use to nd line numbers. (The version is important since line numbers change as updates occur.)

To see source code for di erent kernels, use: <a href="https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/refs/tags">https://git.kernel.org/pub/scm/linux/kernel/git/stable/linux.git/refs/tags </a>and nd the version closest to the one installed on your VM. Following from the screen shot above where the machine was running 5.11.0-34-generic , we would use the v5.11 version below. Also shown in the list is v5.11.1 which would be a newer version (the screenshot’s version ends in .0 not .1).

Once the source code is downloaded, extract it to a local folder. You will then be able to navigate within that folder to view di erent les. Shown below is an example of looking up task_struct from the le linux/sched.h (seethe section on Include Files later). (If you are not using the v5.11 les, then most likely the le will look di erent!)
<h1>2 Running a LKM</h1>
As a rst step in this assignment, we suggest trying to get a simple module to run. A simple module such as the sample provided by the textbook! The textbook (Operating System Concepts 9e) provides additional information on creating and running LKMs on page 96. Reading that section of the book is strongly recommended. Next, we give a short summery of the basic instructions to run an LKM. Note that we need to use a so-called “make le” to manage the compilation of modules. Do not try to use an IDE such as CLion or NetBeans. Four basic commands are needed for dealing with LKMs: 1) make to create the binary les, 2) insmod to insert the LKM into the running kernerl, 3) dmesg to view the kernel log le, and 4) rmmod to remove a loaded LKM. See below:

make sudo insmod simple . ko dmesg sudo rmmod simple

As seen below, running make compiles the LKM into an insertable .ko le. It runs the make le for the module (see the Make les subsection for more information). We then do the insert with insmod which means that it is running. To see the results (printf won’t work), we use dmesg to view the log (where the output from commands like printk will go). There will be a lot of information shown when using dmesg…

So we typically need to scroll down to nd the output from our module. In the screen shot below, the last line ( Loading Module ) is the output from the LKM we compiled and inserted. To remove the LKM, we nally issue the rmmod command (we could also reboot).
<h2>2.1 Make les</h2>
A make le is a le containing a set of directives used with the make tool to automate the build process for a le or a set of les. For example, assume we have a codebase that has hundreds of les with lots of dependencies between them. Generally, we would compile each le, generate an object le and link all the object les together to generate an executable. Even if there is a small change in one of the les, we would have to recompile and link them which is quite a cumbersome and ine cient process. Using Make les, you can ensure that only the les that have been modi ed since the last build and those which are dependent on the changed les are the ones which are compiled.

For this assignment, you will use a specially set up make le. See Algorithm 2 below. In this make le, we have two targets, all and clean. Rather than being dependent on a speci c le, they are more actions that the make le can execute. In this case, all is used to build the le, while clean is to clean the build outputs. For us, we would only need to type make in the correct folder to use this make le. The top most line is special, it says obj-m += simple.o . This say takes the result of compiling the le simple.c (which will be called simple.o), and combine it with the default binaries for creating an module. This produces the .ko le that represents a built LKM that you can load into the operating system.

Algorithm 1 Sample LKM make le (from Operating System Concepts by Silberschatz, Galvin, Gagne)

obj<sup>=</sup>m += simple . o a l l :

make <sup>=</sup>C / lib /modules/$( shell uname <sup>=</sup>r )/ build M=$(PWD) modules clean :

make <sup>=</sup>C / lib /modules/$( shell uname <sup>=</sup>r )/ build M=$(PWD) clean

WARNING: do not use this make le within a path that includes a space ( ) in it’s name, or the module will not build! For example, don’t create a folder called HW M5 on your desktop and develop within it. The space between HW and M5 will cause the build to fail.
<h1>3 Requirements [36 points total]</h1>
In this assignment you will write a LKM for the Linux kernel that displays the following details for all the processes whose PID is greater than an integer given by the user as a module parameter: [26 points]

PROCESS NAME

PID

STATE

PRIORITY

STATIC-PRIORITY

NORMAL-PRIORITY

As shown below, the PID is given as arguments while inserting the module in the kernel.

The next screenshot shows the required details for a process, its child processes (may have 0 or more), and its parent process. You are expected to keep a similar format (not an exact one) for your output which is readable. Your programs must compile and run under Xubuntu (or another variant of Ubuntu)

20.04.
<h2>3.1 Writeup [10 points]</h2>
You are to provide a short write up that illustrates what you found when looking at the Linux source les and documentation. As you will experience, the information about the kernel is somewhat fragmented, which your write up and pull together and complete. Include the following:

The version number of your kernel you used to nd the line numbers (typically the version you found on the kernel website). [0 points but needed to grade others]

Main source le: list each function that you de ned and describe its purpose. Typically this would be functions for init and exit, plus any helpers you created.

Task_struct: at a high level, document each of the struct attributes that your use in your program (e.g., pid, state, prio, etc).

For each header le (except linux/module.h), document each function/macro/struct you use:

Functions: Include the function’s signature, what le/line it is de ned, and a description (include parameters and return values). Example (self-contained, well styled, and readable):
<table width="557">
<tbody>
<tr>
<td width="557"><u>printf</u>De ned on line 133 in stdio.h.Signature:int printf(const char *format, …); Parameters:The rst parameter (format) takes a string, followed by any number of other parameters (see below).Description:This string will be displayed as standard console output. The string may contain special formatting called control characters. For example, including %c indicates a position in the string where a character should appear. The variable containing the string should be passed as the second parameter. The function may take any number of parameters, where the rst is always the main string, and everything else is a value that may be substituted into it. After completion, the function returns the number of displayed (an int).</td>
</tr>
</tbody>
</table>
Macro: same as function.

Struct: Indicate what le/line it is de ned, and document each of the attributes used.

The purpose of the write up is show the research you did when creating your program. Do not quote (or paraphrase) any 3rd party documentation – use your own words and understanding. Although there is no mandatory format, please be professional. Your document should be self-contained, well styled (e.g., no comic sans, random font sizes), and readable (e.g., minimal spelling or grammar errors).
<h1>4 Include Files</h1>
To complete this assignment, you may nd the following include les useful:

linux/sched.h: De nes processes and their associated details.

Useful types:
<ul>
 	<li>struct task_struct – A structure to contain process details (see line 649).</li>
</ul>
linux/sched/signal.h: De nes macros for manipulating processes and threads.

Useful macros:
<ul>
 	<li>for_each_process – A macro to loop over each currently running process (see line 601).</li>
</ul>
linux/list.h: Implements a circular-linked list in C language which could be used to fetch the child processes.

Useful types:
<ul>
 	<li>struct list_head – purpose and line number left for you to discover.</li>
</ul>
Useful functions:
<ul>
 	<li>list_for_each – purpose and line number left for you to discover. <sup>* </sup>list_entry – purpose and line number left for you to discover.</li>
</ul>
Note that you are not limited to using these functions, nor are you required to use all of them.