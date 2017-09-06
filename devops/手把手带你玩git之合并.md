

## 合并指定文件

有的时候，我们只期望从其他分支，合并某些文件到本地，而非全部文件。

- 合并全部

``` bash
$ git merge some-other-branch
```

- 合并指定文件/目录

``` bash
$ git checkout some-other-branch some-dir-files
```

**注意**
>合并全部，用``git merge``；合并部分，竟然不是``merge``，而是``git checkout``。
