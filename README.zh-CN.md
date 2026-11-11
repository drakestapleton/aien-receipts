# aien-receipts

<!-- languages -->
[English](README.md) · **简体中文**
<!-- /languages -->

AIEN 各仓库已发布的签名验证摘要。当 Spark 评判器（运行在本项目 NVIDIA DGX Spark 上的验证流水线）检查一个提交时，它会对一个包进行签名，并把这个包发布到这里。读者可以验证其签名、提交绑定、声明的门禁判定以及凭证摘要。这个凭证架**不**包含底层各门禁的凭证正文、命令、stdout、stderr 或原始日志；仅凭签名无法重新运行或审计这些测试。本仓库只存放数据：没有代码，没有工作流，也没有密钥。唯一的例外是 [`audit/`](audit/README.md) 下的组织审计，它自带检查器和两个工作流。

本仓库为 [Sovereign Intelligence (SI)](https://github.com/aien-dev/sovereign-intelligence) 提供支撑。本项目提出的每一项主张，都应能追溯到这个凭证架上的某个包。

**状态（2026-10-26）：已投入使用。** 首批包于 2026-10-16 发布，此后 Spark 每次验证提交时都会发布已签名的包。凭证架中包含 `aegis-runtime`、`aien-architecture`、`aienos`、`aien-protocols`、`benchmarks`、`omega`、`physics` 以及其他已登记仓库的历史凭证；某个旧提交存在对应的包，并不能说明当前的鉴定状态或合并授权情况。

自 2026-10-24 的政策变更起，PR 和 `main` 上托管运行的 **AIEN Checks** 作业仅供参考，只运行可移植的冒烟门禁；它不读取这个凭证架。只有 `alpha/*` 标签上的 **AIEN Alpha** 作业，要求 `main` 上被打标签的提交具备完整的已签名 Spark 包。早先的 `AIEN Verification` 托管凭证桥接和必需 PR 检查策略属于历史做法，已不再适用。

## 目录结构

```
<repo>/<sha>/bundle.json
```

- `<repo>` 是 `aien-dev` 下的 GitHub 仓库名。
- `<sha>` 是该仓库中被验证提交的完整提交哈希，共 40 个字符。
- `bundle.json` 是由运行门禁的机器（DGX Spark）上的 `aien-test bundle-sign` 生成的已签名摘要包。它为每个门禁携带一个凭证**摘要**和判定，而不是完整的单门禁凭证或日志，此外还带有对整个包的签名。

这个凭证架上只发布已签名的摘要包。底层的门禁证据须另行查找：若已公开，则位于相应仓库保留的 `evidence/` 路径中；若未公开，则需向运行操作员索取。本凭证架不提供通用的产物 URL。不要把一个缺少对应原始字节的摘要，理解为一次经过独立审计的门禁运行。

## 包的检查方式

使用方仓库通过 `aien-test verify-bundle` 验证包，所用的输入是：

- 仓库名和被测提交，
- 该仓库中以 `.aien/verify.pub` 形式提交的公钥，从基础分支读取（绝不从被测提交读取），以及
- 同一基础分支上始终运行的门禁列表 `.aien/always`，作为 `--require` 传入。

签名密钥从不离开验证机器。本仓库只接收签名后的输出，因此任何人都可以读取包，并用公钥重新验证。

## 凭证索引

[`index/receipt_index.json`](index/receipt_index.json) 列出了可以引用的凭证，它们来自这个凭证架和 `aien-dev/omega-evidence`，并附有锁定的提交、实时的 SHA-256 摘要、适用的更正以及最终生效的判定。`tools/replay_receipt.py replay --all` 会重新获取每一个条目，检查凭证架 HEAD 上是否有后来添加的更正，并拒绝任何漂移或任何它不检查的字段。更正文件的优先级高于历史状态，因此 omega-evidence 的 M8 和 M9 凭证在索引中记为 `REFUSED_UNSUPPORTED_PROVENANCE`。准入检查以及少数直接信任的取值，详见 [`docs/receipt-index.md`](docs/receipt-index.md)。

## 工作原理 / 我是如何构建它的

[流水线设计记录](https://github.com/aien-dev/aien-architecture/blob/efbba982f5c095e3ae71c057b22faaafc11ed84d/docs/adr/0033-aien-verification-pipeline.md)指出了问题所在：重复的托管环境搭建，以及需要 Spark 的硬件测试。工作节点发布一个已签名、绑定提交的包；一个独立的使用方借助基础分支上的信任锚来检查它，而不是信任候选提交自带的密钥或门禁列表。签名证明的是所记录陈述的真实性，而不是底层测试的正确性或充分性。

**[逐步检查一个不可变的真实包](docs/verify-one-bundle.md)**：精确锁定的源码、凭证架和验证器版本；基础引用上的公钥和门禁列表；官方的 `aien-test verify-bundle` 命令；以及一个可随时丢弃的篡改/拒绝用例。它链接了实现代码和现有测试。可选的仅签名 OpenSSL 演练已经复现；完整的验证器流程在文档评审环境中明确**尚未复现**，因为该环境中没有 Cargo。无论是仅签名的检查，还是显示通过的参考性 PR 检查，都不能取代完整的已签名 Alpha 判定。

## 规则

- 只追加（这是一项约定：分支上没有强制执行它的保护规则）。某个 `<repo>/<sha>` 对应的包只发布一次，永不改写。对同一提交的再次验证，会在这里形成一个新的提交，而且必须携带完全相同的字节。
- 不对包运行任何工作流。仅有的工作流是两个范围限定在 `audit/` 的组织审计作业（所有者于 2026-10-26 批准；ADR 0033 补遗，aien-dev/aien-architecture-mv#337）。
- 由 DGX Spark 上的 AIEN 验证流水线发布；流水线准则见 `aien-architecture` 仓库（ADR 0033，清单 v2）。

## 许可

Apache-2.0。参见 `LICENSE`。

## 人类主导的智能体工程

当前仓库的维护遵循人类主导的智能体工程（HDAE）：由人类确定目标、架构、约束、验收标准和停止点；智能体在这些边界内实现功能、执行检查、诊断故障并进行修改。人工评审仍对主张、验收和发布决策负责。

有关这一方法的文档说明及一个系统软件案例研究，请参阅 Gregory M. Kurtzer 的 [*Ginux: Building an Independent Kernel That Runs Linux Userspace through Human-Directed Agentic Engineering*（2026）](https://doi.org/10.5281/zenodo.23682548)。历史证据保留其原始措辞和范围；本节不会追溯性地定性用于产生早期记录的方法。
