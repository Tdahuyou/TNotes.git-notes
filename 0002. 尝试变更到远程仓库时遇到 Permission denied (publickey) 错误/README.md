# 0002. 尝试变更到远程仓库时遇到 Permission denied (publickey) 错误

## 📒 notes - 错误示例

```bash
$ git push -u origin main
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

## 🤖 AI - 尝试将代码推送到 GitHub 时遇到 Permission denied (publickey). 错误，应该如何解决？

当你尝试将代码推送到 GitHub 时遇到 `Permission denied (publickey).` 错误，这通常意味着你的 SSH 密钥没有被正确配置或者没有被添加到 GitHub 账户中。以下是一些解决这个问题的步骤：

1. **检查是否已经生成了 SSH 密钥**：
   打开终端并输入以下命令来查看现有的 SSH 密钥。
   ```bash
   ls -al ~/.ssh
   ```

2. **如果没有 SSH 密钥，生成一个新的**：
   如果你没有看到任何 `.pub` 文件（例如 `id_rsa.pub` 或 `id_ed25519.pub`），你需要生成一个新的 SSH 密钥对。
   ```bash
   ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
   ```
   按照提示操作，可以选择保存位置和设置密码。

3. **将 SSH 公钥添加到 GitHub**：
   将新生成的公钥内容复制到剪贴板。
   ```bash
   cat ~/.ssh/id_rsa.pub
   # 或者如果你使用的是 ed25519
   cat ~/.ssh/id_ed25519.pub
   ```
   登录到你的 GitHub 账户，进入 Settings -> SSH and GPG keys 页面，点击 New SSH key 按钮，粘贴你的公钥，并保存。

4. **测试 SSH 连接**：
   在终端中运行以下命令来测试与 GitHub 的连接。
   ```bash
   ssh -T git@github.com
   ```
   如果一切正常，你会看到一条欢迎信息，说明你已经成功通过 SSH 认证。

5. **确保远程 URL 正确**：
   确认你的仓库远程 URL 是正确的。你可以使用以下命令查看当前的远程 URL。
   ```bash
   git remote -v
   ```
   如果 URL 不正确，可以使用以下命令修改它。
   ```bash
   git remote set-url origin git@github.com:Tdahuyou/algorithm_.git
   ```

6. **再次尝试推送**：
   当所有设置都完成后，再次尝试推送代码。
   ```bash
   git push -u origin main
   ```

如果上述步骤之后问题仍然存在，请确保你的 SSH 代理正在运行并且你的密钥已经被添加到 SSH 代理中。你可以通过以下命令来启动 SSH 代理并添加密钥。
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
# 或者
ssh-add ~/.ssh/id_ed25519
```

如果还是无法解决问题，可能需要检查是否有其他安全软件或网络设置阻止了 SSH 连接。

## 🤖 AI - 提交的问题解决了，可是我现在每次 push 都需要输入 Enter passphrase for key '/Users/huyouda/.ssh/id_rsa':

当你每次使用 SSH 密钥进行身份验证时都需要输入密码短语（passphrase），这是因为你的私钥被保护了，以增加安全性。如果你希望避免每次都输入密码短语，可以考虑以下几种方法：

### 1. 使用 `ssh-agent`
`ssh-agent` 是一个守护进程，用于管理你的 SSH 密钥和密码短语。你可以将密钥添加到 `ssh-agent` 中，并在会话期间自动处理认证过程。

#### 启动 `ssh-agent` 并添加密钥
首先，确保 `ssh-agent` 正在运行。你可以在终端中启动它：
```bash
eval "$(ssh-agent -s)"
```

然后，将你的私钥添加到 `ssh-agent` 中：
```bash
ssh-add -K ~/.ssh/id_rsa
# 如果你使用的是 ed25519 密钥，则为
# ssh-add -K ~/.ssh/id_ed25519
```

`-K` 参数会将密码短语保存到 macOS 的 Keychain 中，这样在重启后你也不需要重新输入密码短语。

#### 自动启动 `ssh-agent`
为了确保 `ssh-agent` 在每次打开终端时都能自动启动，你可以将上面的命令添加到你的 shell 配置文件中（例如 `.bashrc`, `.zshrc` 或者 `.profile`）。

### 2. 使用无密码短语的密钥
如果你不介意稍微降低安全性，可以选择生成一个新的没有密码短语的 SSH 密钥。这将允许你在没有任何提示的情况下使用该密钥。

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -N ""
# -N "" 表示不设置密码短语
```

然后将这个新的公钥添加到你的 GitHub 账户中，并更新本地 Git 配置来使用这个新密钥。

### 3. 使用 Keychain Access
macOS 提供了一个名为 Keychain Access 的工具，可以帮助你管理各种密码。你可以将 SSH 密钥添加到 Keychain 中，这样系统就会记住密码短语。

1. 打开 Keychain Access 应用程序。
2. 选择左侧的“登录”键链。
3. 选择菜单栏中的 “Keychain Access” -> “Preferences...”。
4. 勾选 “Show keychain status in menu bar”，这样你就可以从菜单栏快速访问 Keychain。
5. 使用 `ssh-add -K ~/.ssh/id_rsa` 将密钥添加到 Keychain。
6. 当系统提示输入密码短语时，勾选“在钥匙串中记住密码”。

通过以上方法之一，你应该能够避免每次推送代码时都需要输入密码短语。不过，请注意，虽然这些方法提高了便利性，但可能会降低一定的安全性。确保你了解这些风险，并根据你的安全需求做出适当的选择。

---

```bash
$ ssh-add -K ~/.ssh/id_rsa
WARNING: The -K and -A flags are deprecated and have been replaced
         by the --apple-use-keychain and --apple-load-keychain
         flags, respectively.  To suppress this warning, set the
         environment variable APPLE_SSH_ADD_BEHAVIOR as described in
         the ssh-add(1) manual page.
```

这个警告信息表明，`ssh-add` 命令中的 `-K` 和 `-A` 选项已经过时，它们已经被替换为 `--apple-use-keychain` 和 `--apple-load-keychain` 选项。

---

```bash
$ ssh-add --apple-use-keychain ~/.ssh/id_rsa
```