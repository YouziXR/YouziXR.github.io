### 采用 git 作为源代码管理系统。

- 版本管理策略

1. 最新的开发在 master 上进行。
2. 每次发布和部署，都要建立一个对应的版本。版本号采用【分支类型*业务名称*版本号】的格式，例子：F_Project1_V1.0 ，F 代表 features，Project1 为业务名称，版本号为 V1.0。
3. 线上版本的 bug 升级在对应版本上建立开发分支，相应的修改根据需要 merge 到开发主干上来。

> 参考文档：[http://nvie.com/posts/a-successful-git-branching-model/](http://nvie.com/posts/a-successful-git-branching-model/)

- 主要要点：

1. master：代表的总是最新的线上发布版本。
2. develop：开发主干在此进行（在这个分支上进行每日构建）
3. release：一个开发版本进入到发布准备阶段，进行 bug fix 处理。
4. hotfix： 对线上版本进行 Bug 修复（或紧急需求处理）
5. features: 相对较长的一个特性开发版本

- 版本号规则：

1. master 上的版本，1.1， 1.2.， 1.3
2. hotfix 的版本：1.1.1, 1.1.2, 1.2.1, 1.2.2

- 注意事项

1. features 分支上，一个分支尽量开发一个功能模块，不要多个功能模块在一个分支上开发。
2. 开发过程中，如果组员 A 开发的功能依赖组员 B 正在开发的功能，可以待组员 B 开发好相关功能之后，组员 A 直接 pull 组员 B 的分支下来开发，不需要先将组员 B 的分支 merge 到 develop 分支。
3. feature 分支在申请合并之前，最好是先 pull 一下 develop 主分支下来，看一下有没有冲突，如果有就先解决冲突后再申请合并。
