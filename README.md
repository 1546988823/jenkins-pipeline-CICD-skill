**Jenkins-Generator** 是一款专为 DevOps 工程师设计的自动化脚本工具。它能够基于预设的 XML 模板，快速生成并部署适用于 **Java、Go、Node.js** 项目的 Jenkins CI/CD 流水线配置文件。

通过简单的交互式指令，即可完成从项目识别、参数收集、动态端口分配到 Jenkins API 任务创建的全流程自动化。

## 🚀 核心特性

* **多语言模板支持**：内置 Java、Go、Node.js 专属 CI 逻辑模板。
* **自动化 API 交互**：自动通过 Jenkins API 检查任务状态，支持一键创建/更新。
* **动态端口分配**：集成外部端口管理 API，自动获取并解析 `NODE_PORT`。
* **安全替换机制**：采用白名单占位符替换逻辑，严格保护 Jenkins 原生环境变量（如 `${env.WORKSPACE}`）不被破坏。
* **XML 实体保护**：自动处理单引号、`&` 等特殊字符转义，确保生成的 XML 配置合法。

## 🛠️ 技术流程

1. **项目扫描**：识别项目语言类型。
2. **信息收集**：获取 Git 地址、编译命令、Dockerfile、命名空间等核心参数。
3. **动态获取**：调用 `192.168.7.166` 接口获取可用的服务端口。
4. **模板渲染**：基于 `.tpl` 文件生成临时的 `jenkins-ci-temp.xml` 与 `jenkins-cd-temp.xml`。
5. **API 部署**：通过 `curl` 自动将配置推送到 Jenkins 服务端。

## 📋 快速开始

### 1. 准备工作
* **Jenkins 服务端**：默认 `http://192.168.7.57:8080`。
* **认证信息**：需配置 `JENKINS_USER` 与 `JENKINS_TOKEN`。
* **模板路径**：确保 `.claude/skills/jenkins-automation/templates/` 目录下存在对应的 `.tpl` 文件。

### 2. 执行指令
在支持此 Skill 的终端中输入：
```bash
/jenkins-generator
