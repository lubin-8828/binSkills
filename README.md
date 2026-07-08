# BinSkills

我的个人 Claude Code Skills 商店。

## 使用方法

### 添加 marketplace

```shell
# 本地测试
claude plugin marketplace add ./path/to/binskills

# 或从 GitHub 添加（推送后）
claude plugin marketplace add <your-github>/binskills
```

### 安装插件

```shell
claude plugin install <plugin-name>@binskills
```

## 目录结构

```
binskills/
├── .claude-plugin/
│   └── marketplace.json    # Marketplace 清单
├── plugins/                # 插件目录
│   └── <plugin-name>/
│       ├── .claude-plugin/
│       │   └── plugin.json # 插件清单
│       ├── skills/         # Skills
│       ├── commands/       # Commands
│       ├── agents/         # Agents
│       └── hooks/          # Hooks
└── README.md
```

## 创建新插件

1. 在 `plugins/` 下创建插件目录
2. 添加 `.claude-plugin/plugin.json` 描述插件
3. 添加 skills、commands、agents 等组件
4. 在 `.claude-plugin/marketplace.json` 的 `plugins` 数组中注册插件

示例 plugin.json：

```json
{
  "name": "my-skill",
  "description": "我的自定义技能",
  "version": "1.0.0"
}
```

## 参考

- [Claude Code 官方文档 — Plugin Marketplaces](https://code.claude.com/docs/zh-CN/plugin-marketplaces)
