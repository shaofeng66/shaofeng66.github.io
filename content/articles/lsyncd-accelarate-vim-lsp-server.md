---
title: "用lsyncd加速vim language server"
date: 2021-03-20T11:32:35+08:00
tags: ["vim", "lsp"]
---




远程编辑在VSCode里面的解决方案是 [remote ssh](https://code.visualstudio.com/docs/remote/ssh), VSCode会在远端运行一个agent以加速language server对代码文件的访问和解析。然而对vimer，我们可能的解决方案是sshfs，这个的问题在于通过sshfs对文件的访问很慢，导致coc-vim这种ls client慢到基本不能用的地步。

那么我们可以用[lsyncd](https://github.com/axkibe/lsyncd)来帮忙，这东西结合了inotify和rsync来实现近实时的目录同步。我们可以在本地clone一份代码，然后建立一个lsyncd的配置文件lsyncd.conf：
```
sync {
    default.rsync,
    source    = "/home/user/intel/machine/proj",
    target    = "machine:/root/src/proj",
    delay     = 5,
    delete    = false,
    excludeFrom = "/home/user/intel/machine/proj/lsyncd.exclude",
    rsync     = {
        binary   = "/usr/bin/rsync",
        archive  = true,
        compress = true,
        chown = "root:root"
    }
}

```
其中的lsyncd.exclude是需要exclude的一些文件，比如.git等:
```
node_modules
.git
dist
lib
lsyncd.conf
lsyncd.exclude
lsyncd.pid
```

然后，我们只需要启动lsyncd的daemon就可以直接在本地用vim编辑，实时同步到远端了。因为language server访问的是本地文件，速度杠杠的。
```
lsyncd -pidfile lsyncd.pid lsyncd.conf 
```
