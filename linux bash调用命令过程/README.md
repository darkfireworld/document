# linux bash调用命令过程

在Linux，我们看到`native`程序（ELF）和`script`程序（py，sh...）都可以通过`bash`来调用执行。
而`bash`是如何判断调用的指令是`native`还是`script`的呢？

翻看早期版本`sh`的代码：

```
/**/
static int
zexecve(char *pth, char **argv, char **newenvp)
{
    int eno;
    static char buf[PATH_MAX * 2];
    char **eep;
 
    unmetafy(pth, NULL);
    for (eep = argv; *eep; eep++)
    if (*eep != pth)
        unmetafy(*eep, NULL);
    buf[0] = '_';
    buf[1] = '=';
    if (*pth == '/')
    strcpy(buf + 2, pth);
    else
    sprintf(buf + 2, "%s/%s", pwd, pth);
    zputenv(buf);
#ifndef FD_CLOEXEC
    closedumps();
#endif
 
    if (newenvp == NULL)
        newenvp = environ;
    winch_unblock();
    //交托给linux执行ELF,OUT等格式的本地文件，这是通过文件的MAGIC来实现的
    execve(pth, argv, newenvp);
 
    /* If the execve returns (which in general shouldn't happen),   *
     * then check for an errno equal to ENOEXEC.  This errno is set *
     * if the process file has the appropriate access permission,   *
     * but has an invalid magic number in its header.               */
    if ((eno = errno) == ENOEXEC || eno == ENOENT) {
    char execvebuf[POUNDBANGLIMIT + 1], *ptr, *ptr2, *argv0;
    int fd, ct, t0;
 
    if ((fd = open(pth, O_RDONLY|O_NOCTTY)) >= 0) {
        argv0 = *argv;
        *argv = pth;
        execvebuf[0] = '\0';
        ct = read(fd, execvebuf, POUNDBANGLIMIT);
        close(fd);
        if (ct >= 0) {
        if (execvebuf[0] == '#') {
            if (execvebuf[1] == '!') {
            for (t0 = 0; t0 != ct; t0++)
                if (execvebuf[t0] == '\n')
                break;
            while (inblank(execvebuf[t0]))
                execvebuf[t0--] = '\0';
            execvebuf[POUNDBANGLIMIT] = '\0';
            for (ptr = execvebuf + 2; *ptr && *ptr == ' '; ptr++);
            for (ptr2 = ptr; *ptr && *ptr != ' '; ptr++);
            if (eno == ENOENT) {
                char *pprog;
                if (*ptr)
                *ptr = '\0';
                if (*ptr2 != '/' &&
                (pprog = pathprog(ptr2, NULL))) {
                argv[-2] = ptr2;
                argv[-1] = ptr + 1;
                winch_unblock();
                //执行指定解释器
                execve(pprog, argv - 2, newenvp);
                }
                zerr("%s: bad interpreter: %s: %e", pth, ptr2,
                 eno);
            } else if (*ptr) {
                *ptr = '\0';
                argv[-2] = ptr2;
                argv[-1] = ptr + 1;
                winch_unblock();
                execve(ptr2, argv - 2, newenvp);
            } else {
                argv[-1] = ptr2;
                winch_unblock();
                execve(ptr2, argv - 1, newenvp);
            }
            } else if (eno == ENOEXEC) {
            argv[-1] = "sh";
            winch_unblock();
            //如果查询不到，则使用默认的shell执行
            execve("/bin/sh", argv - 1, newenvp);
            }
        } else if (eno == ENOEXEC) {
            for (t0 = 0; t0 != ct; t0++)
            if (!execvebuf[t0])
                break;
            if (t0 == ct) {
            argv[-1] = "sh";
            winch_unblock();
            execve("/bin/sh", argv - 1, newenvp);
            }
        }
        } else
        eno = errno;
        *argv = argv0;
    } else
        eno = errno;
    }
    /* restore the original arguments and path but do not bother with *
     * null characters as these cannot be passed to external commands *
     * anyway.  So the result is truncated at the first null char.    */
    pth = metafy(pth, -1, META_NOALLOC);
    for (eep = argv; *eep; eep++)
    if (*eep != pth)
        (void) metafy(*eep, -1, META_NOALLOC);
    return eno;
}

```

可以发现，基本的流程是这样子的：

1. `bash`首先尝试使用`native模式`运行命令
2. `native模式`下`linux native 加载器`通过识别文件的`magic`，来判断native的类型（ELF，COFF，...），然后执行
3. 如果`native模式`运行失败，则`bash`会采用`script模式`运行命令
4. `script模式`下`bash`会读取文件的第一行(`#!/bin/bash`)，获取指定的`解释器`，如果没有则使用`默认解释器(如/bin/sh)`，最后以该脚本为参数，调用`解释器`。
5. 如果都失败，则报错

# 参考

* [how-does-linux-execute-a-file](http://stackoverflow.com/questions/23295724/how-does-linux-execute-a-file)
* [Magic_number](https://en.wikipedia.org/wiki/Magic_number_(programming))

