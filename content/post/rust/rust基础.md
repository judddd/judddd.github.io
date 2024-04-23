---
title: rust基础
link: rust基础
categories:
- rust
date: 2024-04-19 14:42:10 +0800
date_modified: 2024-04-19 14:42:10 +0800
---

## 官网
https://rustup.rs/ 

##  rust安装配置
```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# rust更新
rustup update

# 卸载
rustup self uninstall

# 版本
rustc --version

# 文档
rustup doc

# 编译
rustc main.rs
```

## 开发工具
RustRover

https://www.jetbrains.com.cn/rust/

## cargo
```
# 版本
cargo --version

# 创建项目
cargo new project_name

# 编译
cargo build
cargo build --release

# 构建并运行
cargo run

# 检查是否通过编译
cargo check

# 镜像加速
$HOME/.cargo/config.toml

[registries]
ustc = { index = "https://mirrors.ustc.edu.cn/crates.io-index/" }

[source.crates-io]
replace-with = 'ustc'

[source.ustc]
registry = "git://mirrors.ustc.edu.cn/crates.io-index"

```

## 依赖库
https://crates.io/

## 参考文档
https://course.rs/about-book.html