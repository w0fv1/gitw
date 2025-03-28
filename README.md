## 脚本概述

在日常开发中，Git 是最常用的版本控制工具之一。**gitw.ps1** 脚本正是为了解决在没有预装 Git 的环境下，用户能够快速获得 Git 工具的问题。脚本通过以下步骤实现目标：

- **检测 Git 安装**：首先检查系统中是否已有 Git 命令。
- **下载 PortableGit**：如果系统中没有 Git，则自动下载指定版本的 PortableGit 压缩包。
- **解压安装**：下载完成后解压 PortableGit 并进行配置。
- **代理支持**：支持在命令行任意位置指定 `-Proxy` 参数，下载及 Git 执行过程中会使用该代理设置。
- **传递命令参数**：除了 `-Proxy` 参数，剩余参数会直接传递给 Git 命令，从而支持各种 Git 操作。

---

## 参数解析

脚本开头通过手动解析传入参数，确保 `-Proxy` 参数可以放在任意位置。解析过程包括：

- 将所有非 `-Proxy` 参数收集到 `$gitArgs` 数组中。
- 当遇到 `-Proxy` 参数时，将其后的值保存到 `$Proxy` 变量，并在下载文件时以及后续执行 Git 命令时使用该代理。

这种灵活的参数解析方式，使得用户在执行脚本时可以更自由地指定代理，而不会影响 Git 命令的其他参数传递。

---

## 检查与下载 Git

脚本首先尝试使用 `Get-Command git` 检查系统是否已有 Git。如果没有找到：

1. **判断本地是否已有解压好的 PortableGit**  
   如果在指定目录下已经存在解压后的 Git，则直接使用该版本。

2. **下载 PortableGit**  
   如果本地不存在，则通过指定的 GitHub URL 下载 PortableGit 安装包。下载过程中会根据用户传入的代理设置参数，确保在网络受限的环境下依然可以成功下载。

3. **解压安装**  
   下载完成后，调用解压程序（PortableGit 本身提供的解压参数），将压缩包内容解压到指定文件夹。解压成功后删除安装包，保证磁盘整洁。

通过这种方式，用户无需预先安装 Git，即可在任意 Windows 环境下快速获得可用的 Git 工具。

---

## 代理支持

对于处于防火墙或需要代理访问外部网络的用户，脚本支持通过 `-Proxy` 参数来设置 HTTP 和 HTTPS 代理。主要体现在两个地方：

- **下载阶段**：在调用 `Invoke-WebRequest` 下载 Git 安装包时，将代理参数添加到请求中。
- **Git 执行阶段**：在执行 Git 命令前，通过设置环境变量 `HTTP_PROXY` 和 `HTTPS_PROXY`，确保 Git 操作过程中能够正常通过代理访问网络资源（例如远程仓库）。

这种设计大大增强了脚本的适用性，适合在各种网络环境下使用。

---

## 使用方法

1. **下载脚本**：将 **gitw.ps1** 保存在你希望执行的位置。
2. **运行脚本**：在 PowerShell 中运行该脚本。如果系统中已有 Git，则会直接使用系统 Git；否则脚本将自动下载并解压 PortableGit。
3. **传递参数**：  
   - 若需要设置代理，可在命令中添加 `-Proxy` 参数，如：  
     ```powershell
     .\gitw.ps1 -Proxy http://your-proxy-address:port status
     ```  
   - 其余参数（例如 `status`、`commit` 等）会直接传递给 Git 命令。

4. **输出结果**：如果没有传入任何 Git 参数，脚本会默认执行 `git --version` 命令，显示 Git 的版本信息。

---

## 使用案例：克隆 React 仓库

假设你希望使用 **gitw.ps1** 脚本来克隆 [React 仓库](https://github.com/facebook/react.git)，你可以按如下方式执行脚本：

1. **无代理的情况下：**

   在 PowerShell 中运行：
   
   ```powershell
   .\gitw.ps1 clone https://github.com/facebook/react.git
   ```

   这条命令会检查系统中是否存在 Git。如果没有，脚本会自动下载 PortableGit，然后使用它执行 `clone` 命令，从而克隆 React 仓库到本地。

3. **使用代理的情况下：**

   如果你的网络环境需要代理，可以通过 `-Proxy` 参数指定代理地址。当代理地址为`http://127.0.0.1:7890`，可以这么写：
   
   ```powershell
   .\gitw.ps1 -Proxy http://127.0.0.1:7890 clone https://github.com/facebook/react.git
   ```
   这条命令会让脚本在下载 PortableGit（如果需要）和后续的 Git 命令执行时均使用指定的代理，确保能够顺利连接到 GitHub。

---

## 总结

**gitw.ps1** 脚本为开发者提供了一个便捷的解决方案，确保在任何 Windows 环境下都能快速使用 Git 工具。无论是通过检测系统 Git，还是自动下载 PortableGit，以及灵活的代理设置，该脚本都体现了对使用场景的全面考虑。对于需要在无 Git 环境中工作或者受限网络环境下的用户来说，这是一个非常实用的工具。

希望这篇介绍文章能帮助你更好地理解并使用 **gitw.ps1** 脚本，从而提高工作效率。
