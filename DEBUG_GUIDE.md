# Dify 远程调试指南 🐛

## 🚀 快速开始（3分钟搞定）

### 1. 准备工作
确保容器中安装了 debugpy：
```bash
cd docker
docker-compose exec api pip install debugpy
```

### 2. 重启服务
```bash
docker-compose down
docker-compose up -d
```

### 3. PyCharm 配置调试

#### 方法A: PyCharm Professional（推荐）
1. 打开 `Run -> Edit Configurations`
2. 点击 `+` → 选择 `Python Debug Server`
3. 配置：
   - Name: `Dify Remote Debug`
   - IDE host name: `localhost`
   - Port: `5678`
   - Path mappings: 
     - Local: `/Users/huchangfeng/PycharmProjects/dify-study/api`
     - Remote: `/app/api`

#### 方法B: PyCharm Community（免费）
1. 创建新的运行配置
2. 选择 `Python → Remote Interpreter`
3. 配置 Docker 连接即可

### 4. 开始调试
1. 在代码中设置断点
2. 启动 PyCharm 调试器（绿色小虫子图标）
3. 访问 `http://localhost/console` 触发断点

## 🎯 调试技巧

### 关键断点位置
```python
# API 入口
api/app_factory.py:create_app()

# 请求处理
api/controllers/console/*.py

# 业务逻辑  
api/services/*.py

# 数据模型
api/models/*.py
```

### 环境变量控制
```bash
# 立即连接调试器（阻塞启动）
DEBUG_WAIT_FOR_CLIENT=true

# 仅启动调试服务器（推荐）  
DEBUG_WAIT_FOR_CLIENT=false

# 禁用调试
ENABLE_REMOTE_DEBUG=false
```

### 常见问题

#### Q: 连接不上调试器？
A: 检查端口是否被占用：
```bash
lsof -i :5678
```

#### Q: 断点不触发？
A: 确认路径映射正确，容器内代码路径是 `/app/api`

#### Q: 调试慢？
A: 设置 `DEBUG_WAIT_FOR_CLIENT=false`，让服务正常启动

## 🔥 进阶用法

### 只调试特定服务
```bash
# 只启用 API 调试，Worker 正常运行
docker-compose up -d worker web db redis
docker-compose up api  # 在前台启动便于查看调试输出
```

### 调试 Celery Worker
在 `docker-compose.yaml` 的 `worker` 服务中也添加相同的调试配置，使用不同端口如 `5679`

### 性能分析
debugpy 支持性能分析，可以在 PyCharm 中查看执行时间和调用栈

## 💡 最佳实践

1. **设置条件断点** - 避免在循环中频繁暂停
2. **使用表达式求值** - Ctrl+Alt+F8 快速查看变量
3. **日志断点** - 不暂停程序，只输出日志
4. **异常断点** - 自动在异常处暂停

---
现在你可以像调试本地代码一样调试 Docker 容器中的 Dify 了！🎉