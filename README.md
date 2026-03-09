# GitHub Actions 自动构建开发镜像

这个仓库会用 GitHub Actions 定时拉取上游基础镜像，然后在其之上安装你自己的开发工具，最后推送到你的 Docker Hub。

默认配置：

- 基础镜像：`lmsysorg/sglang:dev`
- 推送仓库：`DOCKERHUB_USERNAME/sglang:dev`
- 定时任务：每天 `03:00 UTC`
- VS Code Server：构建时自动拉取最新 `stable`
- 多架构：同时推送 `linux/amd64` 和 `linux/arm64`

## 需要配置的 GitHub Secrets

在仓库 `Settings -> Secrets and variables -> Actions` 里添加：

- `DOCKERHUB_USERNAME`：你的 Docker Hub 用户名
- `DOCKERHUB_TOKEN`：你的 Docker Hub Access Token

## 工作方式

workflow 会在以下场景运行：

- 每天定时执行一次
- 手动执行 `Run workflow`
- `main` 分支上的 `docker/` 或 workflow 变更后自动执行

构建时会：

1. 拉取基础镜像
2. 用当前仓库里的 `Dockerfile` 继续构建
3. 推送多个 tag 到 Docker Hub，包括：
   - 主 tag，默认 `dev`
   - 基础镜像对应的 tag 变体
   - 时间戳 tag
   - Git SHA tag

## 自定义开发工具

构建镜像需要的文件都放在 [`docker/`](/Users/julian/py-project/Dev-Env-Setup/docker) 目录下。

直接编辑 [Dockerfile](/Users/julian/py-project/Dev-Env-Setup/docker/Dockerfile) 里的 `apt-get install` 列表即可。

这个 `Dockerfile` 还会在构建时自动查询 VS Code 官方更新接口，下载最新稳定版 `VS Code Server`，并预装常用扩展。

如果你想手动指定别的基础镜像，可以在 GitHub Actions 页面运行 `Build and Push Dev Image`，并填写：

- `base_image`：例如 `lmsysorg/sglang:dev`
- `image_name`：例如 `sglang`
- `image_tag`：例如 `dev`

## 本地构建示例

```bash
docker build \
  --build-arg BASE_IMAGE=lmsysorg/sglang:dev \
  -t your-dockerhub-name/sglang:dev ./docker
```
