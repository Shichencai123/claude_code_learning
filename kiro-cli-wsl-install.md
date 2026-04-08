# Kiro CLI 在 WSL (Ubuntu-22.04) 上的安装指南

## 环境说明

- 宿主机：Windows
- 子系统：WSL2 + Ubuntu-22.04
- 工具：Kiro CLI

---

## 安装步骤

### 1. 进入 WSL 子系统

```bash
wsl -d Ubuntu-22.04
```

> 必须先进入 WSL 环境，不能直接在 Windows PowerShell/CMD 中执行安装脚本。

### 2. 执行安装脚本

```bash
curl -fsSL https://cli.kiro.dev/install | bash
```

安装完成后输出示例：

```
✓ Downloaded and extracted
✓ Package installed successfully

🎉 Installation complete! Happy coding!

1. Important! Before you can continue, you must update your PATH to include:
   /home/<username>/.local/bin
```

安装位置：`/home/<username>/.local/bin/kiro-cli`

### 3. 配置 PATH

**当前会话立即生效：**

```bash
export PATH="$HOME/.local/bin:$PATH"
```

**永久生效（写入 shell 配置）：**

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

### 4. 验证安装

```bash
kiro-cli --version
```

---

## 常见问题

### WSL 中无法粘贴 Windows 剪贴板内容

| 终端类型 | 粘贴方式 |
|---|---|
| Windows Terminal | `Ctrl+Shift+V` 或 右键单击 |
| CMD / PowerShell 窗口内的 WSL | 右键单击 |
| MobaXterm 等第三方终端 | 参考各自终端的粘贴快捷键 |

> `Ctrl+V` 在大多数 Linux 终端中无效，优先尝试**右键单击**。

---

## 一键安装（从 Windows 直接执行）

如果不想手动进入 WSL，可以在 Windows 终端中一条命令完成：

```bash
wsl -d Ubuntu-22.04 -- bash -c "curl -fsSL https://cli.kiro.dev/install | bash"
```

安装完成后仍需进入 WSL 配置 PATH（见步骤 3）。
