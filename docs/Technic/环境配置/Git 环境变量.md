---
date created: 2025/3/18 15:21
date modified: 2025/4/7 10:30
---

[解决问题：电脑已安装了git，vscode识别不到_vscode检测不到git-CSDN博客](https://blog.csdn.net/m0_47876279/article/details/134425427)

要配置 Git 的账号和邮箱，你需要使用以下命令：

```bash
# 配置全局用户名
git config --global user.name "你的名字"

# 配置全局邮箱
git config --global user.email "你的邮箱@example.com"
```

如果你只想为当前仓库配置，可以去掉 `--global` 参数：

```bash
# 配置当前仓库的用户名
git config user.name "你的名字"

# 配置当前仓库的邮箱
git config user.email "你的邮箱@example.com"
```

配置完成后，你可以用以下命令验证：

```bash
# 查看全局配置
git config --global --list

# 或查看当前仓库配置
git config --list
```

这些设置会用于你的 Git 提交记录，让其他人知道谁进行了这些更改。