# Docker 镜像导出工具

本项目提供了 GitHub Actions workflow，用于将 Docker 镜像导出为 tar 包并上传到 GitHub Release。

## 支持的镜像

- `kuaifan/dootask-ai`
- `kuaifan/minder`
- `kuaifan/doookr`

## 导出镜像

### 使用 GitHub Actions 导出

1. 进入 GitHub 仓库的 **Actions** 页面
2. 在左侧工作流列表中选择 **"Export Docker Image to Release"**
3. 点击右侧的 **"Run workflow"** 按钮
4. 填写以下参数：
   - **选择要导出的镜像**：从下拉菜单中选择一个镜像（kuaifan/dootask-ai、kuaifan/minder 或 kuaifan/doookr）
   - **镜像版本/标签**：输入镜像的标签，例如 `latest`、`v1.0.0` 等
5. 点击 **"Run workflow"** 开始执行

> **注意**：Release 标签会根据镜像名称自动生成，无需手动输入。每个镜像有独立的 Release，同一镜像的多个版本会添加到同一个 Release 中。

### 执行流程

workflow 会自动完成以下步骤：
1. 拉取指定的 Docker 镜像
2. 将镜像导出为 tar 文件
3. 压缩为 tar.gz 格式
4. 根据镜像名称自动生成 Release 标签（例如：`kuaifan/dootask-ai` → `dootask-ai`）
5. 创建或更新对应的 GitHub Release（如果 Release 已存在，会将新版本添加到该 Release）
6. 将压缩包上传到 Release

### 下载导出的镜像

导出完成后，可以在以下位置找到镜像文件：
- **GitHub Release**：访问仓库的 Releases 页面，找到对应镜像的 Release（例如：`dootask-ai`、`minder`、`doookr`），下载对应的 `.tar.gz` 文件
  - 每个镜像有独立的 Release，同一镜像的多个版本文件都会在同一个 Release 中
- **GitHub Actions Artifacts**：在 workflow 运行完成后，可以在 Actions 页面下载 Artifacts（保留 7 天）

## 导入镜像

### 方法一：从 GitHub Release 导入

1. 从 GitHub Release 下载对应的 `.tar.gz` 文件
2. 解压文件（如果需要）：
   ```bash
   gunzip kuaifan-dootask-ai-latest.tar.gz
   ```
3. 导入镜像到本地 Docker：
   ```bash
   docker load -i kuaifan-dootask-ai-latest.tar
   ```

### 方法二：直接导入压缩包（推荐）

Docker 支持直接加载压缩的 tar.gz 文件：

```bash
# 下载并直接导入（无需手动解压）
docker load < kuaifan-dootask-ai-latest.tar.gz
```

或者：

```bash
gunzip -c kuaifan-dootask-ai-latest.tar.gz | docker load
```

### 验证导入

导入完成后，可以使用以下命令验证：

```bash
# 查看导入的镜像
docker images | grep kuaifan

# 查看镜像详细信息
docker inspect kuaifan/dootask-ai:latest
```

## Release 标签规则

每个镜像会自动生成独立的 Release 标签，规则如下：
- `kuaifan/dootask-ai` → Release 标签：`dootask-ai`
- `kuaifan/minder` → Release 标签：`minder`
- `kuaifan/doookr` → Release 标签：`doookr`

**重要**：同一镜像的多个版本会添加到同一个 Release 中。例如：
- 第一次导出 `kuaifan/dootask-ai:latest` → 创建 `dootask-ai` Release，包含 `kuaifan-dootask-ai-latest.tar.gz`
- 第二次导出 `kuaifan/dootask-ai:v1.0.0` → 更新 `dootask-ai` Release，添加 `kuaifan-dootask-ai-v1.0.0.tar.gz`
- 此时 `dootask-ai` Release 中包含两个文件

## 使用示例

### 导出 dootask-ai 镜像的 latest 版本

1. 在 GitHub Actions 中选择：
   - 镜像：`kuaifan/dootask-ai`
   - 版本：`latest`

2. 系统会自动创建或更新 `dootask-ai` Release

3. 下载后导入：
   ```bash
   docker load < kuaifan-dootask-ai-latest.tar.gz
   ```

### 导出 dootask-ai 镜像的 v1.0.0 版本

1. 在 GitHub Actions 中选择：
   - 镜像：`kuaifan/dootask-ai`
   - 版本：`v1.0.0`

2. 系统会自动更新 `dootask-ai` Release（如果已存在），添加新版本文件

3. 下载后导入：
   ```bash
   docker load < kuaifan-dootask-ai-v1.0.0.tar.gz
   ```

### 导出特定版本的 minder 镜像

1. 在 GitHub Actions 中选择：
   - 镜像：`kuaifan/minder`
   - 版本：`v1.2.0`

2. 系统会自动创建或更新 `minder` Release

3. 下载后导入：
   ```bash
   docker load < kuaifan-minder-v1.2.0.tar.gz
   ```

## 注意事项

- **Release 管理**：每个镜像有独立的 Release，同一镜像的多个版本会自动添加到同一个 Release 中
- 如果镜像需要认证才能拉取，请在仓库的 **Settings > Secrets and variables > Actions** 中配置：
  - `DOCKERHUB_USERNAME`：Docker Hub 用户名
  - `DOCKERHUB_TOKEN`：Docker Hub 访问令牌
- 公开镜像无需配置凭证即可正常导出
- 导出的文件会自动压缩为 `.tar.gz` 格式以节省空间
- 文件会同时上传到 GitHub Release 和 Artifacts（作为备份，保留 7 天）

## 文件命名规则

导出的文件命名格式为：`{镜像名}-{版本}.tar.gz`

其中镜像名中的 `/` 会被替换为 `-`，例如：
- `kuaifan/dootask-ai:latest` → `kuaifan-dootask-ai-latest.tar.gz`
- `kuaifan/minder:v1.0.0` → `kuaifan-minder-v1.0.0.tar.gz`

