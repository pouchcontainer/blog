[原英文文档](https://github.com/alibaba/pouch/blob/master/CONTRIBUTING.md)

# 为PouchContainer做出贡献

如果您有兴趣参与PouchContainer项目的开源开发，我们十分欢迎您的加入。首先，我们对您的意愿表示鼓励和支持，同时我们为您提供了一系列的贡献向导。

## 主题

* [反馈安全问题](#反馈安全问题)
* [反馈常规问题](#反馈常规问题)
* [贡献代码和文档](#贡献代码和文档)
* [贡献测试用例](#贡献测试用例)
* [提供其他帮助](#提供其他帮助)

## 反馈安全问题

我们一直严肃对待安全问题。作为一个常规原则，我们不希望自己的安全问题被任何人向外扩散。如果您发现PouchContainer存在安全问题，请您不要在公共场合下进行讨论，也不要发布一个面向公共的issue。我们鼓励您发送私信到我们的邮箱来反馈您发现的安全问题：[pouch-dev@list.alibaba-inc.com](mailto:pouch-dev@list.alibaba-inc.com)

## 反馈常规问题

老实说，我们把每个PouchContainer用户都看做是重要贡献者。在体验PouchContainer之后，您可能会对项目有一些反馈。那么我们十分欢迎您在 [NEW ISSUE](https://github.com/alibaba/pouch/issues/new) 中发布新的issue。

自从我们采用分布式的方式对PouchContainer项目进行合作开发，我们鼓励**详尽的**，**具体的**，**格式编排良好的**issue反馈。为了更高效地交流，我们希望大家在发布issue之前，可以通过搜索确定该issue是否已被其他人提出。如果已经有人提出，请将您的反馈详情评论在已存在的issue下，而不要新建一个issue。

为了将issue详情阐述地尽可能标准，我们为反馈者建立了一个issue报告模板：[ISSUE TEMPLATE](./.github/ISSUE_TEMPLATE.md)。请您**务必**遵循模板说明对其进行填写。

在如下很多中情景下，您可以发布一个issue：

* 故障汇报
* 功能请求
* 性能问题汇报
* 功能提议
* 功能设计反馈
* 寻求帮助
* 文档不完整
* 测试改进
* 任何有关项目的疑问
* 其他

同时，我们必须提醒您，当填写一个新的issue时，请您记得移除其中的敏感数据。敏感数据包括密码，密钥，网络地址，商业隐私数据等等。

## 贡献代码和文档

我们鼓励任何可以帮助PouchContainer项目变得更好的贡献。在GitHub上，您可以通过PR（pull请求）对PouchContainer进行改进。

* 若发现拼写错误，请尝试修改！
* 若发现代码故障，请尝试修复！
* 若发现冗余代码，请对其进行删除！
* 若发现测试用例有缺失，请对其进行补充！
* 若可以帮助强化功能，**千万不要**犹豫，请对其进行改进！
* 若发现代码语义不明，请为其添加注释，使之表述更清晰！
* 若发现代码编写不规范，请对其进行重构！
* 若可以帮助改进的文档，那最好不过了，请对其进行改进！
* 如发现文档有误，请对其进行修正！
* ......

我们没有办法将所有场景一一列出，您只需记住一个原则：

> 我们对任何您所能贡献的PR满怀期待。

既然您已经准备好通过PR对PouchContainer项目进行改进，我们建议您阅读下述PR规则。

* [工作区准备](#工作区准备)
* [分支定义](#分支定义)
* [提交规则](#提交规则)
* [PR描述](#PR描述)

### 工作区准备

为了发布PR，我们假定您已经注册了一个GitHub账号。那么您可以按照下述步骤完成准备工作：

1. 将PouchContainer项目**FORK**到您的资源库下。您只需要点击 [alibaba/pouch](https://github.com/alibaba/pouch) 主页右上角的Fork按钮来完成此步骤。接下来您需要进入您自己的资源库 `https://github.com/<your-username>/pouch`，其中 `your-username` 是您的GitHub用户名。

1. **CLONE**您自己的项目资源库来实现本地开发。使用 `git clone https://github.com/<your-username>/pouch.git` 命令将项目资源库克隆到您的本地机器上。接下来您可以新建分支对项目进行修改。

1. 通过下面两个命令，**设置远程** upstream为 `https://github.com/alibaba/pouch.git`：

```
git remote add upstream https://github.com/alibaba/pouch.git
git remote set-url --push upstream no-pushing
```

通过该远程设置，您可以通过下述命令查看您的git远程配置：

```
$ git remote -v
origin     https://github.com/<your-username>/pouch.git (fetch)
origin     https://github.com/<your-username>/pouch.git (push)
upstream   https://github.com/alibaba/pouch.git (fetch)
upstream   no-pushing (push)
```

通过添加该upstream，我们可以轻松地将本地分支同步到upstream分支上。

### 分支定义

目前我们假定每个通过PR实现的贡献都在PouchContainer的主干分支 [branch master](https://github.com/alibaba/pouch/tree/master) 上。在对项目做出贡献之前，了解分支的定义有很大帮助。

作为一个项目贡献者，请您再次注意每个通过PR实现的贡献都要发布到主干分支上。同时，在PouchContainer项目中，除了主干分支还有很多其他分支，我们通常将它们称为rc (release candidate)分支，release分支，以及backport分支。

在正式发布一个项目版本前，我们会切到rc分支。相较于主干分支，在这个分支上我们会做更多的测试工作，并且我们会 [cherry-pick](https://git-scm.com/docs/git-cherry-pick) 一些新的且重要的修复提交。

在正式发布一个版本时，在版本标注前，会有一个release分支。在成功标注后，我们会删除该release分支。

当针对一个已发布的版本进行补丁的向后移植时，我们会将分支切换到backport分支。在完成向后移植后，其影响会在 [SemVer](http://semver.org/) 的PATCH数MAJOR.MINOR.PATCH中得到体现。

### 提交规则

在PouchContainer中，每次提交都必须严格遵守以下两条规则：

* [提交信息](#提交信息)
* [提交内容](#提交内容)

#### 提交信息

提交信息可以帮助评审人更清晰的理解该PR的目的。它也可以增加后续代码评审过程的准确性。我们鼓励贡献者们每次提交都使用**语义明确**的提交信息，不要有留下有歧义的信息。通常，我们推荐以下格式的提交信息：

* docs: xxxx。例如：“docs: add docs about storage installation”
* feature: xxxx。例如：“feature: make result show in sorted order”
* bugfix: xxxx。例如：“bugfix: fix panic when input nil parameter”
* refactor: xxxx。例如：“refactor: simplify to make codes more readable”
* test: xxx。例如：“test: add unit test case for func InsertIntoArray”
* 或者其他有良好可读性且语义就明确的表示方法。

另一方面，我们不推荐使用以下格式的提交信息：

* ~~fix bug~~
* ~~update~~
* ~~add doc~~

#### 提交内容

提交内容是指一次提交中的所有内容上的改变。一次提交的内容应尽可能包含可供评审人完整评审的内容，而不需要依赖其他提交。换句话说，一次提交的内容要能够成功通过持续集成，以此来避免代码混乱。简言之，有两条小规则需要我们记住：

* 在一次提交中，应避免过于庞大的改变
* 每次提交应该是完整且可评审的

此外，在代码修改阶段，我们建议每个贡献者阅读该档：[code style of PouchContainer](docs/contributions/code_styles.md)。

不论是提交信息，还是提交内容，我们都更重视代码评审。

### PR描述

PR是更改PouchContainer项目文件的唯一途径。为了使评审人更好的理解您的目的，PR的描述不应该展现过多细节。我们鼓励贡献者们依照该PR模板来完成pull请求：[PR template](./.github/PULL_REQUEST_TEMPLATE.md)。

## 贡献测试用例

我们欢迎任何测试用例。目前，PouchContainer项目的功能测试用例优先程度最高。

* 编写单元测试时，您需要在与dev包相同的目录下新建一个以 `_test.go` 结尾的测试文件。
* 编写集成测试时，您需要在 `pouch/test/` 目录下添加一个测试脚本。测试会利用 [package check](https://github.com/go-check/check)，一个Go测试包的扩展库来实现。测试脚本以pouch命令命名，例如：所有PouchContainer help api的测试都要写在pouch_api_help_test.go中，所有PouchContainer help命令行测试都要写在pouch_cli_help_test.go中。更多细节请参考 [gocheck document](https://godoc.org/gopkg.in/check.v1)。

## 提供其他帮助

我们选择GitHUb作为PouchContainer合作开发的主要场地，所以PouchContainer的最新更新总会在这里。尽管通过PR实现贡献是一个明确的为项目提供帮助的方法，我们仍然需要其他形式的帮助，例如：

* 在其他人发布的issue下进行回复
* 帮助解决其他用户遇到的问题
* 帮助评审其他的PR设计
* 帮助评审其他PR中的代码
* 与大家讨论PouchContainer，使其设计更明确
* 在GitHub外，对PouchContainer提供支持
* 编写PouchContainer相关的博客等等

总而言之，**您对PouchContainer任何形式的帮助，都是可贵的贡献**
