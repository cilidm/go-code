# Vscode Could't Start Client Rust Language Server

在MacOS使用 Vscode 作为 Rust 开发工具的时候，遇到了报错信息如下：

```bash
Couldn't start client Rust Language Server

Rustup not available. Install from https://www.rustup.rs/
```

这个错误信息提示说明 Rustup 不可用。搜索 github 找到了rust-lang/vscode-rust 的 (issue)\[https://github.com/rust-lang/vscode-rust/issues/622#issuecomment-561596264] 我们需要在 vscode 的 settings.json 中添加 rustup 的路径 `$HOME/.cargo/bin/rustup`。

```json
"rust-client.rustupPath": "$HOME/.cargo/bin/rustup"
```
