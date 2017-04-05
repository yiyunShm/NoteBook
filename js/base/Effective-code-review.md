[TOC]

## 高效的代码审查
在团队中进行代码审查(Code Review)可以提升代码质量，分享项目知识。而且在reivew过程中，能让你的代码可以更好的组织起来，最终达到构建更好的软件、更好的团队。如果你花几分钟搜索代码审查的相关信息，你会看到许多关于代码审查带来的价值的文章。目前有很多方法来进行代码审查：在github中提pull request，或使用像JetBrains的Upsource之类的工具。然而即使用拥有清晰的流程和正确的工具，还遗留了一个大问题需要解决——我们需要找寻哪些问题。

### 要点
* 代码审查者在审查代码时有非常多的东西需要关注。一个团队需要明确对于自己的项目哪些点是重要的，并不断在审查中就这些点进行检查。
* 人工审查代码是十分昂贵的，因此尽可能地使用自动化方式进行审查，如：代码格式，代码样式，检查常见bug，确定常见安全问题以及运行自动化测试。
* 当针对性能进行审查时，了解系统的性能需求是明确潜在问题的关键。
* 一些简单的人工检查可以显著提升应用的安全性。
* 代码审查是应该在互相沟通中进行讨论的，而不是相互对抗。预先确定哪些是要点哪些不是，可以减少冲突并拟定预期。

#### 设计
* 如何让新代码与全局的架构保持一致？
* 代码是否遵循SOLID原则，是否遵循团队使用的设计规范，如领域驱动开发等？
* 新代码是否有使用设计模式？这样使用是否合适？
* 基础代码是否有结合使用了一些标准或设计样式，新的代码是否遵循当前的规范？代码是否正确迁移，或参照了因不规范而淘汰的旧代码？
* 代码的位置是否正确？比如涉及订单的新代码是否在订单服务相关的位置？
* 新代码是否重用了现存的代码？新代码是否可以被现有代码重用？新代码是否有重用代码？如果是，那是否要重构，还是当前可以接受？
* 新代码是否被过度设计了？是否现在还不需要的重用设计？团队如何平衡可重用和YAGNI(You Ain't Gonna Need It)这两种观点？

#### 可读性和可维护性
* 字段、变量、参数、方法、类的命名是否真实反映它们所代表的事物。
* 函数是否过长？
* 我是否可以通过代码阅读来理解它做了什么？
* 我是否理解测试用例测了什么？
* 测试是否很好地覆盖了用例的各种情况？它们是否覆盖了正常和异常用例？是否有忽略的情况？
* 错误信息是否可被理解？
* 不清晰的代码是否被文档、注释或容易理解的测试用例所覆盖？具体可以根据团队自身的喜好决定使用哪种方式。
* 第三方库的引入是否有API文档？

#### 功能
* 代码是否真的达到了预期的目标？如果有自动化测试来确保代码的正确性，测试的代码是否真的可以验证代码达到了协定的需求？
* 代码看上去是否包含不明显的bug，比如使用错误的变量进行检查，或误把and写成or？

#### 性能
让我们深入探讨下性能，这是一个真正能从代码审查中获益的方面。

系统对性能方面的非功能性需求应当同所有架构、设计的领域一样被置于重要的位置。无论你是开发只容许纳秒级延时的低延迟交易系统，还是管理"待办事项"的手机应用，你都应该了解用户所认为的"太慢"。

在考虑我们是否需要就代码性能进行代码审查之前，我们应该问自己几个关于具体需求是什么的问题。虽然一些应用确实不需要考虑每毫秒都花费在哪里，对于大部分应用，花费几个小时的折腾进行优化来获得的些许CPU下降的价值也是有限的，但有些地方还是审查者可以检查一下的，进而确保代码不会有一些常见可避免的性能缺陷。