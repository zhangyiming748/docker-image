# 通过工作流下载指定的docker镜像

本项目使用 GitHub Actions 工作流自动下载 `list.txt` 中列出的 Docker 镜像，并保存为 `.tar` 文件供下载使用。

## 使用方法

### 1. 添加镜像列表

在 `list.txt` 文件中添加需要下载的 Docker 镜像，每行一个：

```txt
linuxserver/code-server:latest
nginx:latest
redis:7-alpine
```

支持注释（以 `#` 开头的行会被忽略）。

### 2. 触发工作流

#### 方式一：手动触发

1. 进入 GitHub 仓库的 **Actions** 标签页
2. 选择 **Download and Save Docker Images** 工作流
3. 点击 **Run workflow** 按钮

#### 方式二：推送标签触发

```bash
git tag v1.0.0
git push origin v1.0.0
```

### 3. 下载镜像文件

工作流完成后：

1. 在工作流运行详情页面找到 **Artifacts** 部分
2. 点击 **docker-images** 下载压缩包
3. 解压后得到所有 `.tar` 格式的镜像文件

## 导入到服务器

### 方法一：单个镜像导入

```bash
# 上传 .tar 文件到服务器后执行
docker load -i linuxserver-code-server-latest.tar
```

### 方法二：批量导入所有镜像

```bash
# 上传所有 .tar 文件到服务器后执行
for file in *.tar; do
  echo "Importing $file..."
  docker load -i "$file"
done
```

### 方法三：使用 SCP 传输并导入

```bash
# 从本地传输到远程服务器
scp docker-images/*.tar user@your-server:/tmp/docker-images/

# SSH 登录到服务器并导入
ssh user@your-server

# 在服务器上执行
cd /tmp/docker-images
for file in *.tar; do
  docker load -i "$file"
done
```

### 方法四：使用 rsync 同步并导入

```bash
# 同步文件到服务器
rsync -avz docker-images/ user@your-server:/tmp/docker-images/

# 在服务器上批量导入
ssh user@your-server "cd /tmp/docker-images && for file in *.tar; do docker load -i \$file; done"
```

### 验证导入结果

```bash
# 查看已导入的镜像
docker images

# 测试运行镜像
docker run --rm linuxserver/code-server:latest --version
```

## 注意事项

- 确保服务器已安装 Docker
- 大镜像文件可能需要较长时间传输，建议使用压缩工具或断点续传
- 导入前检查磁盘空间是否充足：`df -h`
- 可以使用 `docker system prune` 清理未使用的镜像释放空间

## 配置选项

### Docker Hub 认证（可选）

如需下载私有镜像或避免速率限制，在仓库 Settings > Secrets 中添加：

- `DOCKER_USERNAME`: Docker Hub 用户名
- `DOCKER_PASSWORD`: Docker Hub 密码或 Access Token

### 修改工件保留时间

在 `.github/workflows/image.yml` 中修改：

```yaml
retention-days: 7  # 改为需要的天数
```

## 故障排除

### 问题：镜像加载失败

- 检查 `.tar` 文件是否完整下载
- 验证文件完整性：`ls -lh *.tar`
- 确认 Docker 版本兼容性

### 问题：传输速度慢

- 使用压缩：`tar czf images.tar.gz *.tar`
- 使用 rsync 替代 scp
- 考虑使用对象存储中转

### 问题：磁盘空间不足

```bash
# 清理未使用的 Docker 资源
docker system prune -a

# 检查磁盘使用情况
df -h
du -sh /var/lib/docker
```
