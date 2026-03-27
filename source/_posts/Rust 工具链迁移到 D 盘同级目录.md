---
title: Rust 工具链迁移到 D 盘同级目录
created: 2026-03-27
tags: [Rust, Windows, 迁移, 软链接]
---

# Rust 工具链迁移到 D 盘同级目录

## 概述

把 Rust 工具链从 C 盘的用户目录搬到 D 盘。Rust 在 Windows 上默认占用c盘的空间，包括：
- Rustup（Rust 版本管理器）
- Rust 工具链（编译器、标准库这些）
- Cargo 缓存
- Cargo 包注册表

## 默认安装位置

在 Windows 10 上，Rust 相关文件默认装在：

```
C:\Users\cyz\.rustup\      # Rustup 和工具链
C:\Users\cyz\.cargo\       # Cargo 缓存和注册表
```

## 迁移步骤

### 1. 准备工作

1. **关闭所有 Rust 相关程序**
   - 把正在运行的 Rust 项目都关了
   - 关闭 VS Code 或其他 IDE
   - 确保没有 `cargo` 或 `rustc` 进程在运行

2. **检查当前 Rust 安装状态**
   ```powershell
   rustc --version
   cargo --version
   rustup show
   ```

3. **记一下当前安装的工具链**
   ```powershell
   rustup show
   ```
   把当前安装的工具链和默认工具链记下来，方便迁移后检查。

### 2. 创建目标目录

先在 D 盘创建好 Rust 相关的目录：

**PowerShell 命令：**
```powershell
# 创建 rustup 目录
New-Item -ItemType Directory -Path "D:\.rustup" -Force

# 创建 cargo 目录
New-Item -ItemType Directory -Path "D:\.cargo" -Force
```

**CMD 命令：**
```cmd
:: 创建 rustup 目录
mkdir "D:\.rustup"

:: 创建 cargo 目录
mkdir "D:\.cargo"
```

### 3. 停止 Rustup 服务

```powershell
# 如果有 Rustup 服务在运行，先停了它
# 一般情况下不需要，但如果遇到问题可以执行：
# taskkill /F /IM rustup.exe
```

### 4. 迁移文件

用 PowerShell 或 CMD 把文件移过去（会保留文件权限和属性）：

**PowerShell 命令：**
```powershell
# 迁移 .rustup 目录
robocopy "C:\Users\cyz\.rustup" "D:\.rustup" /E /COPYALL /R:0 /W:0

# 迁移 .cargo 目录
robocopy "C:\Users\cyz\.cargo" "D:\.cargo" /E /COPYALL /R:0 /W:0
```

**CMD 命令：**
```cmd
:: 迁移 .rustup 目录
robocopy "C:\Users\cyz\.rustup" "D:\.rustup" /E /COPYALL /R:0 /W:0

:: 迁移 .cargo 目录
robocopy "C:\Users\cyz\.cargo" "D:\.cargo" /E /COPYALL /R:0 /W:0
```

**参数说明：**
- `/E` - 复制所有子目录，包括空的
- `/COPYALL` - 复制所有文件信息（包括权限）
- `/R:0` - 失败了不重试
- `/W:0` - 重试等待时间为 0

### 5. 验证迁移

检查一下文件是不是完整移过去了：

**PowerShell 命令：**
```powershell
# 检查目录大小
Get-ChildItem "D:\.rustup" -Recurse | Measure-Object -Property Length -Sum
Get-ChildItem "D:\.cargo" -Recurse | Measure-Object -Property Length -Sum

# 检查关键文件是否存在
Test-Path "D:\.rustup\settings.toml"
Test-Path "D:\.cargo\bin\rustc.exe"
```

**CMD 命令：**
```cmd
:: 检查关键文件是否存在
dir "D:\.rustup\settings.toml"
dir "D:\.cargo\bin\rustc.exe"
```

### 6. 删除原目录（可选）

确认迁移成功后，就可以把原目录删掉了：

**PowerShell 命令：**
```powershell
# 先备份一下（建议先重命名）
Rename-Item "C:\Users\cyz\.rustup" "C:\Users\cyz\.rustup.backup"
Rename-Item "C:\Users\cyz\.cargo" "C:\Users\cyz\.cargo.backup"

# 确认没问题后删除
Remove-Item "C:\Users\cyz\.rustup.backup" -Recurse -Force
Remove-Item "C:\Users\cyz\.cargo.backup" -Recurse -Force
```

**CMD 命令：**
```cmd
:: 先备份一下（建议先重命名）
ren "C:\Users\cyz\.rustup" ".rustup.backup"
ren "C:\Users\cyz\.cargo" ".cargo.backup"

:: 确认没问题后删除
rmdir /s /q "C:\Users\cyz\.rustup.backup"
rmdir /s /q "C:\Users\cyz\.cargo.backup"
```

## 创建软链接

### 方法：用 mklink 命令（需要管理员权限）

1. **以管理员身份打开命令提示符**
   - 右键点击「命令提示符」图标
   - 选择「以管理员身份运行」

2. **创建软链接**
   **CMD 命令：**
   ```cmd
   :: 删除原目录（如果还存在）
   rd /s /q "C:\Users\cyz\.rustup" 2>nul
   rd /s /q "C:\Users\cyz\.cargo" 2>nul

   :: 创建软链接
   mklink /D "C:\Users\cyz\.rustup" "D:\.rustup"
   mklink /D "C:\Users\cyz\.cargo" "D:\.cargo"
   ```

### 方法二：用 PowerShell 命令

1. **以管理员身份打开 PowerShell**
   - 右键点击 PowerShell 图标
   - 选择「以管理员身份运行」

2. **创建软链接**
   **PowerShell 命令：**
   ```powershell
   # 删除原目录（如果还存在）
   Remove-Item "C:\Users\cyz\.rustup" -Recurse -Force -ErrorAction SilentlyContinue
   Remove-Item "C:\Users\cyz\.cargo" -Recurse -Force -ErrorAction SilentlyContinue

   # 创建符号链接
   New-Item -ItemType SymbolicLink -Path "C:\Users\cyz\.rustup" -Target "D:\.rustup"
   New-Item -ItemType SymbolicLink -Path "C:\Users\cyz\.cargo" -Target "D:\.cargo"
   ```

**注意：** 在 Windows 10 上创建符号链接通常需要开发者模式或管理员权限。

## 配置环境变量（可选）

如果 Rustup 没能正确识别新位置，可以手动设置环境变量：

1. **打开环境变量设置**
   - 右键点击「此电脑」→「属性」
   - 点击「高级系统设置」
   - 点击「环境变量」

2. **添加或修改以下环境变量**（在用户变量里）：
   ```
   RUSTUP_HOME = D:\.rustup
   CARGO_HOME = D:\.cargo
   ```

3. **更新 PATH 环境变量**
   - 把 `C:\Users\cyz\.cargo\bin` 改成 `D:\.cargo\bin`

## 验证迁移结果

### 1. 验证软链接

**PowerShell 命令：**
```powershell
# 检查软链接是否创建成功
Get-Item "C:\Users\cyz\.rustup" | Select-Object LinkType, Target
Get-Item "C:\Users\cyz\.cargo" | Select-Object LinkType, Target
```

**CMD 命令：**
```cmd
:: 检查软链接是否创建成功
dir "C:\Users\cyz" | findstr "rustup cargo"
```

应该显示类似这样的结果：
```
LinkType  Target
-------  ------
SymbolicLink D:\.rustup
```

### 2. 验证 Rust 功能

**PowerShell 命令：**
```powershell
# 检查 Rust 版本
rustc --version
cargo --version

# 检查工具链
rustup show

# 检查 Rustup 主目录
rustup which rustc

# 测试编译一个简单项目
cargo new test_project
cd test_project
cargo build
```

**CMD 命令：**
```cmd
:: 检查 Rust 版本
rustc --version
cargo --version

:: 检查工具链
rustup show

:: 检查 Rustup 主目录
rustup which rustc

:: 测试编译一个简单项目
cargo new test_project
cd test_project
cargo build
```

### 3. 检查磁盘空间

**PowerShell 命令：**
```powershell
# 检查 C 盘空间
Get-PSDrive C

# 检查 D 盘 Rust 目录大小
Get-ChildItem "D:\.rustup" -Recurse | Measure-Object -Property Length -Sum
Get-ChildItem "D:\.cargo" -Recurse | Measure-Object -Property Length -Sum
```

**CMD 命令：**
```cmd
:: 检查 C 盘空间
fsutil volume diskfree C:

:: 检查 D 盘空间
fsutil volume diskfree D:
```

## 常见问题

### Q1: 权限不足，没法创建软链接

**解决办法：**
- 以管理员身份运行命令提示符或 PowerShell
- 或者启用开发者模式：设置 → 更新和安全 → 开发者选项 → 开发者模式

### Q2: Rust 命令找不到

**解决办法：**
1. 检查 PATH 环境变量里有没有 `D:\.cargo\bin`
2. 重新打开终端
3. 如果还是不行，重新安装 Rustup：
   ```powershell
   # 卸载 Rustup
   rustup self uninstall

   # 重新安装（指定安装目录）
   $env:RUSTUP_HOME="D:\.rustup"
   $env:CARGO_HOME="D:\.cargo"
   Invoke-WebRequest -Uri https://win.rustup.rs/x86_64 -OutFile rustup-init.exe
   .\rustup-init.exe
   ```

### Q3: Cargo 缓存还是占了很多空间

**解决办法：**
清理 Cargo 缓存：
```powershell
cargo cache --autoclean
```

或者手动清理：
```powershell
Remove-Item "D:\.cargo\registry\cache" -Recurse -Force
```

### Q4: 迁移后编译失败

**解决办法：**
1. 清理项目构建缓存：
   ```powershell
   cargo clean
   ```

2. 重新构建项目：
   ```powershell
   cargo build
   ```

3. 如果还是有问题，检查工具链是否完整：
   ```powershell
   rustup update
   rustup component add rust-src
   ```

## 维护建议

### 定期清理

**PowerShell 命令：**
```powershell
# 清理 Cargo 缓存
cargo cache --autoclean

# 清理旧的工具链
rustup self uninstall
rustup update
```

**CMD 命令：**
```cmd
:: 清理 Cargo 缓存
cargo cache --autoclean

:: 清理旧的工具链
rustup self uninstall
rustup update
```

### 监控空间使用

**PowerShell 命令：**
```powershell
# 查看 Rust 相关目录大小
Get-ChildItem "D:\.rustup" -Recurse | Measure-Object -Property Length -Sum | Select-Object @{Name="Size(GB)";Expression={[math]::Round($_.Sum / 1GB, 2)}}
Get-ChildItem "D:\.cargo" -Recurse | Measure-Object -Property Length -Sum | Select-Object @{Name="Size(GB)";Expression={[math]::Round($_.Sum / 1GB, 2)}}

# 查看 Cargo 缓存大小
Get-ChildItem "D:\.cargo\registry" -Recurse | Measure-Object -Property Length -Sum | Select-Object @{Name="Size(GB)";Expression={[math]::Round($_.Sum / 1GB, 2)}}
```

**CMD 命令：**
```cmd
:: 查看目录大小（用 dir 命令）
dir "D:\.rustup" /s

dir "D:\.cargo" /s

dir "D:\.cargo\registry" /s
```

## 回滚步骤

如果迁移后出问题了，可以按下面的步骤回滚：

1. **删除软链接**
   **PowerShell 命令：**
   ```powershell
   Remove-Item "C:\Users\cyz\.rustup" -Force
   Remove-Item "C:\Users\cyz\.cargo" -Force
   ```

   **CMD 命令：**
   ```cmd
   :: 删除软链接
   rd "C:\Users\cyz\.rustup"
   rd "C:\Users\cyz\.cargo"
   ```

2. **恢复备份**
   **PowerShell 命令：**
   ```powershell
   Move-Item "C:\Users\cyz\.rustup.backup" "C:\Users\cyz\.rustup"
   Move-Item "C:\Users\cyz\.cargo.backup" "C:\Users\cyz\.cargo"
   ```

   **CMD 命令：**
   ```cmd
   :: 恢复备份
   ren "C:\Users\cyz\.rustup.backup" ".rustup"
   ren "C:\Users\cyz\.cargo.backup" ".cargo"
   ```

3. **验证**
   **PowerShell 命令：**
   ```powershell
   rustc --version
   cargo --version
   ```

   **CMD 命令：**
   ```cmd
   :: 验证
   rustc --version
   cargo --version
   ```

## 总结

按照上面的步骤，你就能成功把 Rust 工具链从用户目录迁移到 D 盘，然后通过软链接保持原来的路径，这样既节省了 C 盘空间，又不影响 Rust 的正常使用。

迁移完成后，建议：
1. 定期清理 Cargo 缓存
2. 注意监控磁盘空间使用情况
3. 保持 Rust 工具链更新
4. 定期备份重要配置文件
