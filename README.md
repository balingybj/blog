# 我的个人博客仓库

本仓库专门用于存放我的个人博客。每个目录就是一个主题，目录下面的目录就是子主题了。每个 *.md 文件就是一篇文章。希望大家毫不犹豫地给我 watch、start、issues、follow。

## 为什么不用现有的博客平台，比如  CSDN、博客园、OSChina？

之前一直在 CSDN 上写博客。这类平台的优点是可以被搜索引擎索引，方便网友检索，并且可以和同一个平台的其他用户互动。虽然 CSDN 也支持 Markdown。但 CSDN 上修改文章不方便，而且图片容易丢失。CSDN 还会用，这里主要存放个人认为比较关键的文章，并且会同步到 CSDN 上。

## 为什么不用现有的笔记软件，比如有道云笔记、为知笔记？

为知笔记一直在用，为知笔记适合放所有需要用文字记录的东西，包括那些没有技术含量、很琐碎的文章，比如某个工具的安装，资料收集等。而且这类比较软件可以在各个客户端同步，查看比较方便。但为知笔记不方便和小伙伴分享（不花钱的话），Markdown 的编辑体验也不够好。

## 为什么不用 GitHub Pages？

本来挺看好 Github Pages 的。但 Github Pages 需要使用到静态博客框架或模板，会在目录下引入其他文件，比如模板文件、配置文件、生成的静态网页文件。但我有简洁强迫症，不想把笔记目录弄乱。我只想在目录下看到自己写的文章。所以就不用 Github Pages 了。而且 Github 页面本来就支持渲染 Markdown 文件，这个默认的渲染效果就能满足我了。

## 使用普通 GitHub 仓库写博客的好处

-   编辑内容方便。可以在本地用 Markdown 编辑。配合 Typora 编辑器，舍我其谁。
-   不用担心排版问题。博文都是 Markdown 格式的纯文本内容，容易渲染。而且 GitHub 自动支持代码高亮。
-   发布方便。每次编辑完，只要`git pull`就行。
-   修改方便。发现文章中有需要更新修改的地方，本地编辑，然后`git pull`就行。

## 这里存放哪些内容？

我会在为知笔记上存放所有需要文字记录的东西。在 [blog.csdn](http://blog.csdn.net/candcplusplus) 的博客上发布所有可以分享的东西。在这里会发布我特别想分享给小伙伴的东西。这里的内容会同步到 [blog.csdn](http://blog.csdn.net/candcplusplus) ，同步到为知笔记。所以内容数量上是：为知笔记 >  [blog.csdn](http://blog.csdn.net/candcplusplus)  >本仓库。内容平均质量上是：本仓库 >  [blog.csdn](http://blog.csdn.net/candcplusplus)  > 为知笔记。文章的更新速度是：本仓库 > 为知笔记 >  [blog.csdn](http://blog.csdn.net/candcplusplus)。

## 内容组织

### 目录对应主题、文件对应文章

每个目录对应一个主题，每个 *.md 文件对应一篇文章。目录下的目录表示某个主题的子主题。目录层次尽量不要太深。.images 目录比较特殊，用于存放文章引用的图片。

### 图片的存放位置

每级目录下最多出现一个 .images 目录用于存放该级目录下所有文章需要引用的图片。

这么组织是考虑到：如果将所有图片集中放置到最外层的 .images 的目录下，虽然这样比较清爽，目录层次也比较简洁，但在文章中引用图片不方便。这种情况下要引用图片的话，要么使用绝对路径。要么使用相对路径时记住文章所处的层级；如果将文章需要引用的图都放置在和文章同级的目录下，那么文章目录下会出现很多图片文件，看起来比较乱。为了目录不那么乱，也为了引用图片时方便。在文章同级目录下创建一个 .images 目录是比较折中的方案。

### 为什么不用图床？

目前没找到可以长期提供稳定、快速、廉价的图床。以后找到了再改用图床。而且使用图床的话，在没网时在本地查看文章会看不到图片。



