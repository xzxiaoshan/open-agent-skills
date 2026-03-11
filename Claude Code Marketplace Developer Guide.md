# Claude Code 插件市场创建与分发

> 原文：[Create and distribute a plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces) | [创建和分发插件市场](https://code.claude.com/docs/zh-CN/plugin-marketplaces) | 

插件市场是一个集中式目录，用于向他人分发插件。市场提供集中发现、版本跟踪、自动更新，并支持多种源类型（Git 仓库、本地路径等）。本指南将帮助您创建自己的市场，以便与团队或社区共享插件。

## 概述

创建和分发市场涉及以下步骤：

1. **创建插件**：构建包含 commands、agents、hooks、mcpServers、lspServers 或 skills 的插件。本指南假设您已有待分发的插件。
2. **创建市场文件**：定义 `marketplace.json` 文件，列出插件及其获取位置。
3. **托管市场**：推送到 GitHub、GitLab 或其他 Git 托管服务。
4. **与用户分享**：用户通过 `/plugin marketplace add` 添加市场，并安装插件。

市场上线后，可通过推送更改来更新。用户使用 `/plugin marketplace update` 刷新本地副本。

## 核心层级结构

```
Marketplace (市场)
│   ├── name: "company-tools"
│   ├── owner: { name, email }
│   └── metadata: { description, version }
│
├── plugins (数组)
│   │
│   ├── Plugin 1
│   │   ├── name: "document-tools"
│   │   ├── source: "./plugins/doc-tools"
│   │   ├── description: "..."
│   │   ├── strict: false
│   │   │
│   │   │── components（平级关系）
│   │   │   ├── commands[]     → slash commands
│   │   │   ├── agents[]       → agents
│   │   │   ├── skills[]       → skills
│   │   │   ├── hooks{}        → hooks
│   │   │   ├── mcpServers{}   → mcpServers
│   │   │   └── lspServers{}   → lspServers
│   │   │
│   │   └── ...
│   │
│   ├── Plugin 2
│   │   └── ...
│   │
│   └── Plugin N
│       └── ...
```

| 层级 | 名称        | 说明                                                                |
| ---- | ----------- | ------------------------------------------------------------------- |
| 1    | Marketplace | 最顶层容器，类似应用商店                                            |
| 2    | plugins     | 插件数组，包含多个 Plugin                                           |
| 3    | Plugin      | 独立扩展单元，包含多种组件                                          |
| 4    | components  | commands、agents、skills、hooks、mcpServers、lspServers（平级关系） |

## 实践演练：创建本地市场

本示例创建一个包含 `/quality-review` skill 的市场，用于代码审查。

### 创建目录结构

```bash
mkdir -p my-marketplace/.claude-plugin
mkdir -p my-marketplace/plugins/review-plugin/.claude-plugin
mkdir -p my-marketplace/plugins/review-plugin/commands
```

### 创建 commands 文件

在 `my-marketplace/plugins/review-plugin/commands/` 目录下创建 `review.md`：

```markdown
Review the code I've selected or the recent changes for:
- Potential bugs or edge cases
- Security concerns
- Performance issues
- Readability improvements

Be concise and actionable.
```

### 创建插件清单

在 `my-marketplace/plugins/review-plugin/.claude-plugin/` 目录下创建 `plugin.json`：

```json
{
  "name": "review-plugin",
  "description": "Adds a /review command for quick code reviews",
  "version": "1.0.0"
}
```

### 创建市场文件

在 `my-marketplace/.claude-plugin/` 目录下创建 `marketplace.json`：

```json
{
  "name": "my-plugins",
  "owner": {
    "name": "Your Name"
  },
  "plugins": [
    {
      "name": "review-plugin",
      "source": "./plugins/review-plugin",
      "description": "Adds a /review command for quick code reviews"
    }
  ]
}
```

### 添加并安装

```bash
/plugin marketplace add ./my-marketplace
/plugin install review-plugin@my-plugins
```

> **插件安装机制**：用户安装插件时，Claude Code 会将插件目录复制到缓存位置。插件不能使用 `../shared-utils` 等路径引用目录外部的文件。如需跨插件共享文件，请使用符号链接或重新组织市场结构。

## 创建市场文件

在仓库根目录创建 `.claude-plugin/marketplace.json`。此文件定义市场的名称、所有者信息以及插件列表。每个插件条目至少需要 `name` 和 `source` 字段。

```json
{
  "name": "company-tools",
  "owner": {
    "name": "DevTools Team",
    "email": "devops@example.com"
  },
  "plugins": [
    {
      "name": "code-formatter",
      "source": "./plugins/formatter",
      "description": "Automatic code formatting on save",
      "version": "2.1.0",
      "author": {
        "name": "DevTools Team"
      }
    },
    {
      "name": "deployment-tools",
      "source": {
        "source": "github",
        "repo": "company/deploy-plugin"
      },
      "description": "Deployment automation tools"
    }
  ]
}
```

### 市场文件 Schema

#### 必填字段

| 字段      | 类型   | 描述                                                                                            | 示例           |
| --------- | ------ | ----------------------------------------------------------------------------------------------- | -------------- |
| `name`    | string | 市场标识符（kebab-case，无空格）。用户安装时可见，如 `/plugin install my-tool@your-marketplace` | `"acme-tools"` |
| `owner`   | object | 市场维护者信息                                                                                  |                |
| `plugins` | array  | 可用插件列表                                                                                    |                |

> **保留名称**：以下市场名称保留供 Anthropic 官方使用：`claude-code-marketplace`、`claude-code-plugins`、`claude-plugins-official`、`anthropic-marketplace`、`anthropic-plugins`、`agent-skills`、`life-sciences`。模仿官方市场的名称（如 `official-claude-plugins`）也会被阻止。

#### 所有者字段

| 字段    | 类型   | 必填 | 描述               |
| ------- | ------ | ---- | ------------------ |
| `name`  | string | 是   | 维护者或团队的名称 |
| `email` | string | 否   | 维护者的联系邮箱   |

#### 可选元数据

| 字段                   | 类型   | 描述                                                                                                                   |
| ---------------------- | ------ | ---------------------------------------------------------------------------------------------------------------------- |
| `metadata.description` | string | 市场简要描述                                                                                                           |
| `metadata.version`     | string | 市场版本                                                                                                               |
| `metadata.pluginRoot`  | string | 相对插件源路径的前置基础目录（如 `"./plugins"` 允许写 `"source": "formatter"` 而非 `"source": "./plugins/formatter"`） |

## 插件条目

`plugins` 数组中的每个条目描述一个插件及其获取位置。可包含插件清单 Schema 中的任何字段（如 `description`、`version`、`author`、`commands`、`hooks` 等），以及市场特定字段：`source`、`category`、`tags` 和 `strict`。

### 必填字段

| 字段     | 类型           | 描述                                                                                         |
| -------- | -------------- | -------------------------------------------------------------------------------------------- |
| `name`   | string         | 插件标识符（kebab-case，无空格）。用户安装时可见，如 `/plugin install my-plugin@marketplace` |
| `source` | string\|object | 获取插件的位置                                                                               |

### 可选插件字段

**标准元数据字段**：

| 字段          | 类型    | 描述                                                                           |
| ------------- | ------- | ------------------------------------------------------------------------------ |
| `description` | string  | 插件简要描述                                                                   |
| `version`     | string  | 插件版本                                                                       |
| `author`      | object  | 插件作者信息（`name` 必填，`email` 可选）                                      |
| `homepage`    | string  | 插件主页或文档 URL                                                             |
| `repository`  | string  | 源代码仓库 URL                                                                 |
| `license`     | string  | SPDX 许可证标识符（如 MIT, Apache-2.0）                                        |
| `keywords`    | array   | 用于插件发现和分类的标签                                                       |
| `category`    | string  | 用于组织插件的类别                                                             |
| `tags`        | array   | 用于搜索的标签                                                                 |
| `strict`      | boolean | 控制 `plugin.json` 是否为组件定义的权威来源（默认 true）。详见下文 Strict 模式 |

**组件配置字段**：

| 字段         | 类型           | 描述                            |
| ------------ | -------------- | ------------------------------- |
| `commands`   | string\|array  | commands 文件或目录的自定义路径 |
| `agents`     | string\|array  | agents 文件的自定义路径         |
| `skills`     | string\|array  | skills 文件或目录的自定义路径   |
| `hooks`      | string\|object | hooks 配置或 hooks 文件路径     |
| `mcpServers` | string\|object | mcpServers 配置或 MCP 配置路径  |
| `lspServers` | string\|object | lspServers 配置或 LSP 配置路径  |

## 插件源

插件源告诉 Claude Code 从哪里获取每个插件。

| 源类型       | 格式                           | 字段                                      | 说明                                   |
| ------------ | ------------------------------ | ----------------------------------------- | -------------------------------------- |
| 相对路径     | `string`（如 `"./my-plugin"`） | —                                         | 市场仓库内的本地目录，必须以 `./` 开头 |
| `github`     | object                         | `repo`, `ref?`, `sha?`                    | GitHub 仓库                            |
| `url`        | object                         | `url`（必须以 .git 结尾）, `ref?`, `sha?` | Git URL 源                             |
| `git-subdir` | object                         | `url`, `path`, `ref?`, `sha?`             | Git 仓库内的子目录，使用稀疏克隆       |
| `npm`        | object                         | `package`, `version?`, `registry?`        | 通过 npm install 安装                  |
| `pip`        | object                         | `package`, `version?`, `registry?`        | 通过 pip 安装                          |

### 相对路径

```json
{
  "name": "my-plugin",
  "source": "./plugins/my-plugin"
}
```

### GitHub 仓库

```json
{
  "name": "github-plugin",
  "source": {
    "source": "github",
    "repo": "owner/plugin-repo"
  }
}
```

可指定分支、标签或提交：

```json
{
  "name": "github-plugin",
  "source": {
    "source": "github",
    "repo": "owner/plugin-repo",
    "ref": "v2.0.0",
    "sha": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0"
  }
}
```

| 字段   | 类型   | 描述                                       |
| ------ | ------ | ------------------------------------------ |
| `repo` | string | 必填。GitHub 仓库，格式为 `owner/repo`     |
| `ref`  | string | 可选。Git 分支或标签（默认为仓库默认分支） |
| `sha`  | string | 可选。完整的 40 字符 git commit SHA        |

### Git 仓库

```json
{
  "name": "git-plugin",
  "source": {
    "source": "url",
    "url": "https://gitlab.com/team/plugin.git"
  }
}
```

### Git 子目录

使用 `git-subdir` 指向 Git 仓库内子目录中的插件。Claude Code 使用稀疏克隆仅获取子目录。

```json
{
  "name": "my-plugin",
  "source": {
    "source": "git-subdir",
    "url": "https://github.com/acme-corp/monorepo.git",
    "path": "tools/claude-plugin"
  }
}
```

### npm 包

```json
{
  "name": "my-npm-plugin",
  "source": {
    "source": "npm",
    "package": "@acme/claude-plugin"
  }
}
```

指定版本或私有仓库：

```json
{
  "name": "my-npm-plugin",
  "source": {
    "source": "npm",
    "package": "@acme/claude-plugin",
    "version": "^2.0.0",
    "registry": "https://npm.example.com"
  }
}
```

## 高级插件条目示例

```json
{
  "name": "enterprise-tools",
  "source": {
    "source": "github",
    "repo": "company/enterprise-plugin"
  },
  "description": "Enterprise workflow automation tools",
  "version": "2.1.0",
  "author": {
    "name": "Enterprise Team",
    "email": "enterprise@example.com"
  },
  "homepage": "https://docs.example.com/plugins/enterprise-tools",
  "repository": "https://github.com/company/enterprise-plugin",
  "license": "MIT",
  "keywords": ["enterprise", "workflow", "automation"],
  "category": "productivity",
  "commands": [
    "./commands/core/",
    "./commands/enterprise/",
    "./commands/experimental/preview.md"
  ],
  "agents": [
    "./agents/security-reviewer.md", 
    "./agents/compliance-checker.md"],
  "skills": [
    "./skills/code-review",
    "./skills/deployment-guide"
  ],
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
          }
        ]
      }
    ]
  },
  "mcpServers": {
    "enterprise-db": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"]
    }
  },
  "strict": false
}
```

**关键注意事项**：

- **`commands`、`agents` 和 `skills`**：可指定多个目录或单个文件。路径相对于插件根目录。
- **`${CLAUDE_PLUGIN_ROOT}`**：在 hooks 和 mcpServers 配置中使用此变量引用插件安装目录中的文件。插件安装时会被复制到缓存位置。
- **`strict: false`**：插件不需要自己的 `plugin.json`，市场条目定义所有内容。

## Strict 模式

`strict` 字段控制 `plugin.json` 是否为组件定义（commands、agents、hooks、skills、mcpServers、lspServers、outputStyles）的权威来源。

| 值             | 行为                                                                          |
| -------------- | ----------------------------------------------------------------------------- |
| `true`（默认） | `plugin.json` 是权威来源。市场条目可补充额外组件，两者合并                    |
| `false`        | 市场条目是完整定义。如果插件也有 `plugin.json` 声明组件，则冲突，插件加载失败 |

**使用场景**：

- **`strict: true`**：插件有自己的 `plugin.json` 并管理自己的组件。市场条目可在其基础上添加额外 commands 或 hooks。适用于大多数插件。
- **`strict: false`**：市场运营者希望完全控制。插件仓库提供原始文件，市场条目定义哪些文件作为 commands、agents、hooks 等暴露。适用于市场重新组织或重新策划插件组件。

## 市场托管与分发

### 在 GitHub 上托管（推荐）

1. **创建仓库**：为市场设置新仓库
2. **添加市场文件**：创建包含插件定义的 `.claude-plugin/marketplace.json`
3. **与团队分享**：用户通过 `/plugin marketplace add owner/repo` 添加市场

**优点**：内置版本控制、问题跟踪和团队协作功能。

### 在其他 Git 服务上托管

任何 Git 托管服务都适用，如 GitLab、Bitbucket 和自托管服务器：

```bash
/plugin marketplace add https://gitlab.com/company/plugins.git
```

### 私有仓库

Claude Code 支持从私有仓库安装插件。对于手动安装和更新，使用现有的 git 凭据助手。如果 `git clone` 在终端中对私有仓库有效，在 Claude Code 中也有效。

后台自动更新在启动时运行，无需凭据助手（因为交互式提示会阻止 Claude Code 启动）。要为私有市场启用自动更新，请在环境中设置适当的认证令牌：

| 提供商    | 环境变量                     | 说明                                      |
| --------- | ---------------------------- | ----------------------------------------- |
| GitHub    | `GITHUB_TOKEN` 或 `GH_TOKEN` | Personal access token 或 GitHub App token |
| GitLab    | `GITLAB_TOKEN` 或 `GL_TOKEN` | Personal access token 或 project token    |
| Bitbucket | `BITBUCKET_TOKEN`            | App password 或 repository access token   |

```bash
export GITHUB_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxx
```

### 分发前本地测试

```bash
/plugin marketplace add ./my-local-marketplace
/plugin install test-plugin@my-local-marketplace
```

### 为团队强制要求市场

配置仓库，使团队成员在信任项目文件夹时自动提示安装市场。添加到 `.claude/settings.json`：

```json
{
  "extraKnownMarketplaces": {
    "company-tools": {
      "source": {
        "source": "github",
        "repo": "your-org/claude-plugins"
      }
    }
  }
}
```

指定默认启用的插件：

```json
{
  "enabledPlugins": {
    "code-formatter@company-tools": true,
    "deployment-tools@company-tools": true
  }
}
```

## 企业市场限制

使用托管设置中的 `strictKnownMarketplaces` 限制用户可添加的市场。

| 值             | 行为                                 |
| -------------- | ------------------------------------ |
| 未定义（默认） | 无限制。用户可添加任何市场           |
| 空数组 `[]`    | 完全锁定。用户无法添加任何新市场     |
| 源列表         | 用户只能添加与允许列表完全匹配的市场 |

### 常见配置

禁用所有市场添加：

```json
{
  "strictKnownMarketplaces": []
}
```

仅允许特定市场：

```json
{
  "strictKnownMarketplaces": [
    {
      "source": "github",
      "repo": "acme-corp/approved-plugins"
    },
    {
      "source": "github",
      "repo": "acme-corp/security-tools",
      "ref": "v2.0"
    },
    {
      "source": "url",
      "url": "https://plugins.example.com/marketplace.json"
    }
  ]
}
```

使用正则表达式匹配主机允许内部 git 服务器的所有市场：

```json
{
  "strictKnownMarketplaces": [
    {
      "source": "hostPattern",
      "hostPattern": "^github\\.example\\.com$"
    }
  ]
}
```

使用正则表达式匹配路径允许特定目录的文件系统市场：

```json
{
  "strictKnownMarketplaces": [
    {
      "source": "pathPattern",
      "pathPattern": "^/opt/approved/"
    }
  ]
}
```

### 限制工作原理

限制在插件安装过程早期验证，在任何网络请求或文件系统操作之前。允许列表使用精确匹配：

- 对于 GitHub 源：`repo` 必填，如允许列表中指定了 `ref` 或 `path` 也必须匹配
- 对于 URL 源：完整 URL 必须完全匹配
- 对于 `hostPattern` 源：市场主机与正则表达式匹配
- 对于 `pathPattern` 源：市场文件系统路径与正则表达式匹配

由于 `strictKnownMarketplaces` 在托管设置中设置，单个用户和项目配置无法覆盖这些限制。

## 版本解析与发布渠道

插件版本决定缓存路径和更新检测。可在插件清单（`plugin.json`）或市场条目（`marketplace.json`）中指定版本。

### 设置发布渠道

为插件支持"stable"和"latest"发布渠道，可设置两个指向同一仓库不同 ref 或 SHA 的市场，然后通过托管设置分配给不同用户组。

**stable-tools 市场**：

```json
{
  "name": "stable-tools",
  "plugins": [
    {
      "name": "code-formatter",
      "source": {
        "source": "github",
        "repo": "acme-corp/code-formatter",
        "ref": "stable"
      }
    }
  ]
}
```

**latest-tools 市场**：

```json
{
  "name": "latest-tools",
  "plugins": [
    {
      "name": "code-formatter",
      "source": {
        "source": "github",
        "repo": "acme-corp/code-formatter",
        "ref": "latest"
      }
    }
  ]
}
```

## 验证与测试

在分享市场前务必测试。

验证市场 JSON 语法：

```bash
claude plugin validate .
```

或在 Claude Code 内：

```bash
/plugin validate .
```

添加市场进行测试：

```bash
/plugin marketplace add ./path/to/marketplace
```

安装测试插件验证一切正常：

```bash
/plugin install test-plugin@marketplace-name
```

## 故障排除

### 市场无法加载

**症状**：无法添加市场或看不到其中的插件

**解决方案**：

- 验证市场 URL 是否可访问
- 检查 `.claude-plugin/marketplace.json` 是否存在于指定路径
- 使用 `claude plugin validate` 或 `/plugin validate` 确保 JSON 语法有效
- 对于私有仓库，确认您有访问权限

### 市场验证错误

运行 `claude plugin validate .` 或 `/plugin validate .` 检查问题。

**常见错误**：

| 错误                                              | 原因            | 解决方案                                             |
| ------------------------------------------------- | --------------- | ---------------------------------------------------- |
| `File not found: .claude-plugin/marketplace.json` | 缺少清单文件    | 创建包含必填字段的 `.claude-plugin/marketplace.json` |
| `Invalid JSON syntax: Unexpected token...`        | JSON 语法错误   | 检查是否缺少逗号、多余逗号或未加引号的字符串         |
| `Duplicate plugin name "x" found in marketplace`  | 两个插件同名    | 为每个插件提供唯一的 `name` 值                       |
| `plugins[0].source: Path traversal not allowed`   | 源路径包含 `..` | 使用相对于市场根目录的路径，不包含 `..`              |

**警告**（非阻塞）：

- `Marketplace has no plugins defined`：在 `plugins` 数组中添加至少一个插件
- `No marketplace description provided`：添加 `metadata.description` 帮助用户了解市场
- `Plugin "x" uses npm source which is not yet fully implemented`：改用 `github` 或本地路径源

### 插件安装失败

**症状**：市场出现但插件安装失败

**解决方案**：

- 验证插件源 URL 是否可访问
- 检查插件目录是否包含所需文件
- 对于 GitHub 源，确保仓库是公开的或您有访问权限
- 通过克隆/下载手动测试插件源

### 私有仓库认证失败

**症状**：从私有仓库安装插件时出现认证错误

**解决方案**：

对于手动安装和更新：

- 验证您已通过 git 提供商认证（如 GitHub 运行 `gh auth status`）
- 检查凭据助手配置：`git config --global credential.helper`
- 尝试手动克隆仓库验证凭据有效

对于后台自动更新：

- 在环境中设置适当的令牌：`echo $GITHUB_TOKEN`
- 检查令牌有所需权限（仓库读取权限）
- 对于 GitHub，确保令牌有 `repo` scope
- 对于 GitLab，确保令牌至少有 `read_repository` scope
- 验证令牌未过期

### Git 操作超时

**症状**：插件安装或市场更新失败，超时错误如"Git clone timed out after 120s"

**原因**：Claude Code 对所有 git 操作使用 120 秒超时

**解决方案**：使用环境变量增加超时时间（毫秒）：

```bash
export CLAUDE_CODE_PLUGIN_GIT_TIMEOUT_MS=300000  # 5 分钟
```

### URL 市场中相对路径插件失败

**症状**：通过 URL 添加市场（如 `https://example.com/marketplace.json`），但相对路径源如 `"./plugins/my-plugin"` 的插件安装失败

**原因**：URL 市场只下载 `marketplace.json` 文件本身，不下载插件文件

**解决方案**：

- 使用外部源：将插件条目改为使用 GitHub、npm 或 git URL 源
- 使用 Git 市场：在 Git 仓库中托管市场并使用 git URL 添加

### 安装后文件未找到

**症状**：插件安装成功但文件引用失败，尤其是插件目录外的文件

**原因**：插件被复制到缓存目录而非原地使用。引用插件目录外文件的路径（如 `../shared-utils`）无效

**解决方案**：使用符号链接或重新组织目录结构

## 官方 Skills 示例

以下来自官方仓库 [anthropics/skills](https://github.com/anthropics/skills) 的配置示例，展示如何将多个 skills 组织到 plugin 中：

```json
{
  "name": "anthropic-agent-skills",
  "owner": {
    "name": "Keith Lazuka",
    "email": "klazuka@anthropic.com"
  },
  "metadata": {
    "description": "Anthropic example skills",
    "version": "1.0.0"
  },
  "plugins": [
    {
      "name": "document-skills",
      "description": "Collection of document processing suite including Excel, Word, PowerPoint, and PDF capabilities",
      "source": "./",
      "strict": false,
      "skills": [
        "./skills/xlsx",
        "./skills/docx",
        "./skills/pptx",
        "./skills/pdf"
      ]
    },
    {
      "name": "example-skills",
      "description": "Collection of example skills demonstrating various capabilities",
      "source": "./",
      "strict": false,
      "skills": [
        "./skills/algorithmic-art",
        "./skills/brand-guidelines",
        "./skills/canvas-design",
        "./skills/frontend-design",
        "./skills/mcp-builder",
        "./skills/skill-creator"
      ]
    }
  ]
}
```

## 相关资源

- [Discover and install prebuilt plugins](https://code.claude.com/docs/en/discover-plugins) - 从现有市场安装插件
- [Plugins](https://code.claude.com/docs/en/plugins) - 创建自己的插件
- [Plugins reference](https://code.claude.com/docs/en/plugins-reference) - 完整技术规范和 Schema
- [Plugin settings](https://code.claude.com/docs/en/plugin-settings) - 插件配置选项
- [strictKnownMarketplaces reference](https://code.claude.com/docs/en/strictknownmarketplaces) - 托管市场限制
