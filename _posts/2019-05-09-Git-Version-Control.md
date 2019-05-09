### 采用 git 作为源代码管理系统。

- 版本管理策略
1. 最新的开发在 master 上进行。
2. 每次发布和部署，都要建立一个对应的版本。版本号采用【分支类型_业务名称_版本号】的格式，例子：F_Project1_V1.0 ，F代表features，Project1为业务名称，版本号为V1.0。
3. 线上版本的bug升级在对应版本上建立开发分支，相应的修改根据需要 merge  到开发主干上来。

> 参考文档：[http://nvie.com/posts/a-successful-git-branching-model/](http://nvie.com/posts/a-successful-git-branching-model/)

- 主要要点：
1. master：代表的总是最新的线上发布版本。
2. develop：开发主干在此进行（在这个分支上进行每日构建）
3. release：一个开发版本进入到发布准备阶段，进行bug fix处理。
4. hotfix：   对线上版本进行Bug修复（或紧急需求处理）
5. features: 相对较长的一个特性开发版本

- 版本号规则：
1. master 上的版本，1.1， 1.2.， 1.3
2. hotfix 的版本：1.1.1, 1.1.2, 1.2.1, 1.2.2
 
- 注意事项
1. features分支上，一个分支尽量开发一个功能模块，不要多个功能模块在一个分支上开发。
2. 开发过程中，如果组员A开发的功能依赖组员B正在开发的功能，可以待组员B开发好相关功能之后，组员A直接pull组员B的分支下来开发，不需要先将组员B的分支merge到develop分支。
3. feature 分支在申请合并之前，最好是先 pull 一下 develop 主分支下来，看一下有没有冲突，如果有就先解决冲突后再申请合并。