---
layout: 使用gitlab和git
title: 对Unity3D进行版本管理（一）
date: 2020-03-02 22:04:45
tags: 
    - Unity
    - 版本管理
    - git
    - git lfs
catalog: true
categories:
    - git
---
## 零，前言

版本管理很常见，git也很常见，这其实都没啥好说的。我之前实习的时候也是用git管理的Unity项目。但是毕竟Unity存在许多的二进制文件，和很多大文件（github不允许超过100m的单个文件上传，超过50m就会警告）。每次更改一多，比方说导入什么3D模型之类的，就会合并很慢，而且仓库太大了，push和pull的速度也会大幅度下降。

因此我们需要用到Git LFS这个有趣的东西。

## 一，关于Git LFS

Git LFS是一个git的拓展，在前几年的git版本中已经整合进去了。

其基本原理如下图所示，就是把一些大文件标记，然后单独的上传到一个服务器上。主仓库中只留下一个指向该文件的指针文件。这样可以避免主仓库过大。

![](https://cdn.jsdelivr.net/gh/JokerLD/Image/Other/20200301211424.png)

## 二，实践

基本上分为几步，

第一，先初始化我们的git repo，然后使用git lfs的初始化指令

```bash
git lfs install
```

然后可以使用该命令加上标记大文件

```bash
git track
```

比方说如下命令就是把当前目录下的所有的gif文件标记为大文件

```bash
git lfs track "*.gif"		
```

当然我们也可以直接在项目的根目录下编辑`.gitattributes`文件，该文件记录有所有被track的信息。

对于Unity项目来说，只需把下面的代码复制进该文件中。几乎已经涵盖了所有的需要记录的文件类型

```bash
## Unity ##

*.cs diff=csharp text
*.cginc text
*.shader text

*.mat merge=unityyamlmerge eol=lf
*.anim merge=unityyamlmerge eol=lf
*.unity merge=unityyamlmerge eol=lf
*.prefab merge=unityyamlmerge eol=lf
*.physicsMaterial2D merge=unityyamlmerge eol=lf
*.physicMaterial merge=unityyamlmerge eol=lf
*.asset merge=unityyamlmerge eol=lf
*.meta merge=unityyamlmerge eol=lf
*.controller merge=unityyamlmerge eol=lf


## git-lfs ##

#Image
*.jpg filter=lfs diff=lfs merge=lfs -text
*.jpeg filter=lfs diff=lfs merge=lfs -text
*.png filter=lfs diff=lfs merge=lfs -text
*.gif filter=lfs diff=lfs merge=lfs -text
*.psd filter=lfs diff=lfs merge=lfs -text
*.ai filter=lfs diff=lfs merge=lfs -text

#Audio
*.mp3 filter=lfs diff=lfs merge=lfs -text
*.wav filter=lfs diff=lfs merge=lfs -text
*.ogg filter=lfs diff=lfs merge=lfs -text

#Video
*.mp4 filter=lfs diff=lfs merge=lfs -text
*.mov filter=lfs diff=lfs merge=lfs -text

#3D Object
*.FBX filter=lfs diff=lfs merge=lfs -text
*.fbx filter=lfs diff=lfs merge=lfs -text
*.blend filter=lfs diff=lfs merge=lfs -text
*.obj filter=lfs diff=lfs merge=lfs -text

#ETC
*.a filter=lfs diff=lfs merge=lfs -text
*.exr filter=lfs diff=lfs merge=lfs -text
*.tga filter=lfs diff=lfs merge=lfs -text
*.pdf filter=lfs diff=lfs merge=lfs -text
*.zip filter=lfs diff=lfs merge=lfs -text
*.dll filter=lfs diff=lfs merge=lfs -text
*.unitypackage filter=lfs diff=lfs merge=lfs -text
*.aif filter=lfs diff=lfs merge=lfs -text
*.ttf filter=lfs diff=lfs merge=lfs -text
*.rns filter=lfs diff=lfs merge=lfs -text
*.reason filter=lfs diff=lfs merge=lfs -text
*.lxo filter=lfs diff=lfs merge=lfs -text
```



## 三，遇到的问题

遇到的问题可太多了。。。

我贴一些解决方案的链接在参考资料里面。

### 问题一，关于文件实际上没有被git lfs标记的问题

这个是因为仓库中已经存在大文件了。我估摸着git lfs的设计者是这么想的。

一开始这个仓库应该是是空的或者没有大文件，然后我们使用**git lfs install**初始化，添加**.gitattributes**文件标记大文件类型。然后再加入大文件进入这个仓库中。

如果说我们有一个Unity项目，已经开发过一会儿了。现在我们加入Git LFS管理这个仓库。那么我们就需要把大文件转换为**file pointers**。官方的说明如下，参见[6]。

- Sometimes files end up committed within your repository when they should have been committed with LFS. This can be accidental, or can come up when converting an existing Git repository to use Git LFS. To make this possible, Git LFS ships with a `git lfs migrate` command that allows for such migrations:

也就是说大文件本来不应该在我们的git commit历史中，但是对于一个已经开发了一段时间的仓库来说，肯定是有大文件已经被commit过了。因此我们需要手动转换，用到**git lfs migrate**这个命令

在 Git LFS v2.2.1 及以上的版本中，该命令已经被集成了。

```bash
git lfs migrate import --include="*.mp4"
```

上述命令是把所有的mp4文件转换为Git LFS Object（也就是文件指针）。这样原先的目录下只剩下一些1kb左右大小的文件指针了。真正的文件在本地被存储在 *.git/objects*目录下。

我们在.gitattritubes文件里已经加了，所以直接

```bash
git lfs migrate import
```

就可以了。

使用git lfs checkout可以把文件释放出来。我之前就犯了一个错误。转换完成之后目录中只剩下file pointers文件指针了，结果我却打开了Unity，这个时候肯定是会出问题了。因为你的各种大文件实际上只是些1KB的指针文件。

同样的我们在clone一个使用Git LFS的仓库之后，如果发现大文件都是些1kb的文件指针。那么我们需要手动使用

```bash
git lfs fetch -all
```

把所有的存储在LFS上的文件都下载下来。

当然也可以直接

```bash
git lfs pull
```

一步到位。



### 问题二，github push失败的各种可能问题

首先需要明确的一点是，git lfs的传输协议里不支持socks5代理！！！

这是非常重要的，我之前就吃了太多的这个方面的亏。。。。因为我在本地git设置的时候把github.com网址下的所有http，https传输都设置为了socks5代理。结果就是各种连接错误。。。

后来我干脆用了GitLab，因为我不想把github的代理改掉，毕竟一改掉速度就慢的一比。。。（虽然gitlab速度也慢的一比）



### 问题三，Git LFS标记的大文件进入了commit历史

这个是不应该出现的，github官网上就说了你应该先把大文件移出去或者或使用**git lfs migrate**转换，然后再**git add，git commit**。



## 四，总结

总而言之，言而总之，Git LFS虽然说已经发展了四五年了。但是问题还是有不少的，感觉用到的人也不是很多。毕竟只有对于整个仓库很大，单个文件超过100m的情况下，才比较有用。

总之踩了不少坑，对于整个git lfs也算是一知半解了。后续实践中有新的了解，再继续更新。



## 参考资料

[1] [使用 Git 管理 Unity 项目，狂飙的博客]([https://networm.me/2018/07/29/git-for-unity-project/#%E5%BF%BD%E7%95%A5%E6%96%87%E4%BB%B6%E7%AE%A1%E7%90%86](https://networm.me/2018/07/29/git-for-unity-project/#忽略文件管理))

[2] [Git 迁移到 Git LFS 实践，狂飙的博客](https://networm.me/2018/05/13/migrate-to-gitlfs/)

[3] [git lfs official site](https://git-lfs.github.com/)

[4] [Unity gitignore](https://github.com/github/gitignore/blob/master/Unity.gitignore)

[5] [Git LFS 操作指南](https://zzz.buzz/zh/2016/04/19/the-guide-to-git-lfs/)

[6] [Git LFS官方教程](https://github.com/git-lfs/git-lfs/wiki/Tutorial)