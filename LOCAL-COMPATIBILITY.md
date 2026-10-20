# 本地工作目录兼容补丁

来源：本机已安装的 dsh-purge 1.1.71；未重新下载远端 master，也未安装到当前运行的 Harness。
补丁标识：1.1.71+cwdcompat.1。保留上游署名和许可。

## 修复范围

新版 Harness 通过 `working-directory:current` 运行时上下文提供工作目录，不再注册 `cwd` 模板变量。旧扩展生成的 `{{cwd}}` 后缀会在严格渲染时失败。

- 保留已清空的 bundle personaSuffix 和 CWD_ONLY_SUFFIX。
- 增加独立的小型兼容函数，只去除扩展生成的旧工作目录句式。
- 旧 persona 转换分支不再重新生成 `{{cwd}}`。
- 清理 core.js 两个旧文本常量中的同一引用。
- 不填造 cwd，不用进程目录代替会话目录，不修改 Harness 渲染器。
- 自定义变量、路径示例以及运行时上下文保持原有处理方式。

## 已验证

本机 Harness 包版本：0.2.1-rc.2。
使用本机真实 dsh-system-prompt 渲染器测试，共 13 项通过、0 项跳过。
测试覆盖旧后缀及 persona 变体、无后缀、自定义路径、字面量、未知变量报错、输入不可变性、工作目录上下文保留，以及旧式带 cwd 变量的组装输入。
另对所有随包 JavaScript / MJS 文件做语法检查。

这不是完整桌面端认证，也没有验证其它 Harness 版本的实际渲染器。尚未执行安装、桌面重启或真实模型请求。

## 在本机复测

在源码目录执行：

```sh
DSH_RENDERER_PATH='/Applications/DeepSeek Harness.app/Contents/Resources/app/dsh/node_modules/@deepseek-ai/dsh-system-prompt/lib/index.js' \
node --permission --allow-fs-read='*' tests/cwd-compat.test.mjs
```

Node 22.14.0 已验证。权限模式下直接运行测试文件，避免测试运行器再派生子进程。

## 更新包与安装

交付的 tar.gz 使用 npm 风格 `package/` 根目录，含源码、兼容测试和本说明；排除本地备份及 default-prompt-inject.md。
源码目录可继续用于本地维护。

当前会话无权写入 `~/.dsh/`，因此本补丁没有覆盖已安装插件。安装应由有该目录写权限的会话或用户本机操作完成；安装前备份现有插件，并在 Harness 完全退出后替换。不要执行整套宿主补丁脚本来应用这个仅涉及提示词兼容的修复。

若以本地 tarball 安装，先确认使用的插件管理器支持该来源；本次未验证其安装流程。不提供未经验证的一键安装命令。

## 后续版本维护

上游更新可能覆盖本地补丁。升级 Harness 或插件后，先对新副本重跑回归检查，再做桌面端对话冒烟测试。不能据此保证所有未来版本稳定。

最小改动清单：lib/identity.js、lib/core.js、lib/legacy-cwd.js。bundle 配置中的空后缀也必须保留。
