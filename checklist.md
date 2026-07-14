# GitHub 上传准备清单

## 1. 确认已完成的修改

- ✅ `README.md` → 重写为 DLZ Calculator 简体中文版
- ✅ `cli/Cargo.toml` → name 改为 dlz-calculator，authors 已更新
- ✅ `cli/src/main.rs` → App 名称改为 dlz-calculator
- ✅ `cli/src/repl.rs` → 欢迎消息和缓存路径改为 DLZ Calculator
- ✅ `cli/help.txt` → 路径和名称更新
- ✅ `web/public/index.html` → title 改为 DLZ Calculator
- ✅ `kalk/src/integration_testing.rs` → 扩展名 .kalker → .dlz
- ✅ 测试文件 → .kalker 扩展名批量改为 .dlz
- ✅ `target/` 已在 .gitignore 中，不会被上传

## 2. 还需手动确认/修改的

- [ ] `kalk/Cargo.toml` → authors 改为你的名字
- [ ] `cli/Cargo.toml` → repository 填入你的 GitHub 仓库地址
- [ ] 删除 `cli/kalker.ico` 或替换为你的图标
- [ ] `.gitignore` 添加 `web/public/kalk/` 避免上传 WASM 构建产物

## 3. 上传步骤

```bash
# 1. 确认所有更改已提交
git add -A
git commit -m "Rename project to DLZ Calculator, fix log/log10, improve equation solving"

# 2. 创建 GitHub 仓库
# 在 GitHub.com 上创建新仓库（例如 dlz-calculator），不要勾选 README

# 3. 替换远程仓库地址
git remote remove origin
git remote add origin https://github.com/你的用户名/dlz-calculator.git

# 4. 推送
git push -u origin master
```

## 4. 文件大小提醒

- `target/` 已忽略 ✅
- WASM 构建产物 `kalk/pkg/` 较小（~1MB），可保留
- `web/public/kalk/` 建议加入 .gitignore