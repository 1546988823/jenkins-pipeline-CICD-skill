---
name: jenkins-generator
description: 自动生成 Java/Go/Node 的 Jenkins CI/CD 流水线xml配置文件并调用jenkins api接口生成job。
disable-model-invocation: false  #仅在你输入 /jenkins-generator 时触发，节省 token
---

# Jenkins 流水线生成指令

当你被调用时，请严格执行以下流程：
# Jenkins 自动化全流程指令

你作为一个经验丰富的 DevOps 工程师，现在你将根据用户提供的信息和本地模版生成xml文件，然后请求jenkins的创建job api接口来创建流水线
### 1. 初始信息设置
- JENKINS_URL = 
- JENKINS_USER = jinke
- JENKINS_TOKEN = 
### 2. 扫描与信息收集
- **项目识别**：询问用户项目语言类型(目前只支持Java,Go,Node)。
- **参数获取**：
  - 请求用户提供：`服务名`、`Git地址`、`编译命令`、`Dockerfile文件名`、`服务端口`、`服务启动命令`。
  - `APP_NAME`：服务命名
  - `GIT_URL`：项目 Git 仓库地址。
  - `BUILD_COMMAND`：项目编译指令，默认提供 `mvn clean install -am -Ptest -Dmaven.test.skip=true`。
  - `DOCKERFILE`：项目dockerfile文件名，默认提供 `Dockerfile`。
  - `APP_PORT`：项目启动监听端口
  - `APP_NAMESPACE`: 项目所在k8s命名空间
  - `APP_COMMAND`：服务启动命令，默认提供`java -jar /app/app.jar --spring.profiles.active=test`

### 3. 动态端口处理 
  - **调用 API**：在终端执行 `curl -s http://flask_url:5000/api/port`。
  - **解析结果**：获取返回的端口号并赋值给 `${NODE_PORT}`。如果失败，需请求用户手动输入。
  - **调用 API**：在终端执行 `curl -s http://flask_url:5000/api/namespace`。
  - **解析结果**：获取返回的所有值展示给用户并让其选择一个命名空间，然后将用户选择的值赋值给`${APP_NAMESPACE}

### 4. CI配置文件生成 (核心替换逻辑)
- **读取模板**：根据用户输入的语言类型，如果是java，则读取 `.claude/skills/jenkins-automation/templates/java-ci.xml.tpl`；其它开发语言则读取同目录下带各自语言命名的模版文件
- **精准替换守则 (白名单机制)**：
  - **仅替换** 以下全大写占位符：`${APP_NAME}`, `${GIT_URL}`, `${BUILD_COMMAND}`。
  - **绝对禁止修改** Jenkinsfile 原生变量：如 `${env.WORKSPACE}`, `${env.docker_image}`, `${env.docker_image_tag}`, `${env.app_name}` 等。
  - **转义处理**：替换时必须维持 XML 实体格式。例如，确保单引号周围是 `&apos;`，如果 `BUILD_COMMAND` 中有特殊字符（如 `&`），必须转义为 `&amp;`。
  - **逻辑补充**：CD_JOB_NAME默认生成规则为test-${APP_NAME}-update
- **文件输出**：在当前目录生成临时文件 `jenkins-ci-temp.xml`。由于在 Windows 环境，请直接使用 `Edit` 或 `writeFile` 工具，不要使用 `sed`。

### 5. CD配置文件生成 (核心替换逻辑)
- **读取模板**：读取 `.claude/skills/jenkins-automation/templates/cd.xml.tpl`；
- **精准替换守则 (白名单机制)**：
  - **仅替换** 以下全大写占位符：`${APP_NAME}`,  `${NODE_PORT}`、 `${APP_PORT}`、 `${APP_NAMESPACE}`、`${APP_COMMAND}`。
  - **绝对禁止修改** Jenkinsfile 原生变量：如 `${env.WORKSPACE}`, `${env.docker_image}`, `${env.docker_image_tag}`, `${env.app_name}` 等。
  - **转义处理**：替换时必须维持 XML 实体格式。例如，确保单引号周围是 `&apos;`，如果 `BUILD_COMMAND` 中有特殊字符（如 `&`），必须转义为 `&amp;`。
- **文件输出**：在当前目录生成临时文件 `jenkins-cd-temp.xml`。由于在 Windows 环境，请直接使用 `Edit` 或 `writeFile` 工具，不要使用 `sed`。

### 6. Jenkins API 交互
- **准备工作**：检查环境变量 `$JENKINS_USER` 和 `$JENKINS_TOKEN` 是否已设置。
- **任务创建/更新**：
  - 先检查 CI/CD Job 是否存在：`curl -u $JENKINS_USER:$JENKINS_TOKEN  [JENKINS_URL]/job/test-[APP_NAME]-build/config.xml`;`curl -u $JENKINS_USER:$JENKINS_TOKEN  [JENKINS_URL]/job/test-[APP_NAME]-update/config.xml`
  - 若都返回 404：使用 `POST` 访问 `/createItem?name=test-[APP_NAME]-build`，并上传 `jenkins-ci-temp.xml`；使用 `POST` 访问 `/createItem?name=test-[APP_NAME]-update`，并上传 `jenkins-cd-temp.xml`
  - 若任务已存在：告知用户等待用户判断再进行下一步
- **清理**：操作完成后删除 `jenkins-ci-temp.xml`,`jenkins-cd-temp.xml`。

### 7. 执行反馈
- 打印生成的 Jenkins Job 链接。


