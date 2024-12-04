---
title: Nix 使用教程
date: 2024-12-03 15:12:07
tags:
  - "linux"
  - "nix"
excerpt: false
---


# 1.Nix 简述

## 1.1 安装方法

1. 官方推荐的安装方式：
```bash
sh <(curl -L https://nixos.org/nix/install)
```

2. 多用户安装（推荐）：
```bash
sh <(curl -L https://nixos.org/nix/install) --daemon
```

安装好后，记得执行下`source ~/.bashrc`或者`source ~/.zshrc`进行初始化，取决于你用的是`bash`还是`zsh`。

还可以手动初始化：
```bash
. ~/.nix-profile/etc/profile.d/nix.sh
```

## 1.2 Nix 的优缺点

优点：
1. 跨平台兼容性
- 可以在 Ubuntu、Fedora、macOS 等多种系统上使用
- 保持了包管理的一致性

2. 依赖管理优势
- 完全可重现的包管理
- 可以精确控制软件包版本
- 不同版本软件可以并存
- 回滚和切换软件版本非常容易

3. 隔离性
- 每个包都在独立的目录中
- 避免包之间的冲突
- 不会污染系统环境

4. 原子性更新
- 要么完全成功，要么完全回滚
- 降低系统损坏风险

缺点：
1. 学习成本高
- Nix 语言和概念相对复杂
- 与传统包管理器差异较大

2. 性能开销
- 首次安装和下载可能较慢
- 额外的存储空间消耗

3. 社区支持
- 相比`apt`、`yum`，生态较小
- 部分软件包可能不够及时

## 1.3 使用示例

```bash
# 安装软件
nix-env -iA nixpkgs.firefox

# 列出已安装软件
nix-env -q

# 卸载软件
nix-env -e firefox

# 更新所有软件
nix-channel --update
nix-env -u
```

建议：
- 对于日常使用，可以并存使用系统原生包管理器
- 适合开发者和追求系统可控性的用户
- 不建议完全替代系统默认包管理器