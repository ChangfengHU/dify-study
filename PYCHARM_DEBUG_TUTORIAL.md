# PyCharm + debugpy 远程调试 Dify 完全指南 🔥

## 📖 什么是 debugpy？

**debugpy** 就像一座桥梁，让你在本地 PyCharm 中调试运行在 Docker 容器里的 Python 代码。

想象一下：
- 🏠 本地：你的 PyCharm IDE
- 🐳 容器：运行 Dify 的 Docker 容器  
- 🌉 debugpy：连接本地和容器的调试桥梁

## 🎯 最终效果

完成后你可以：
1. ✅ 在本地 PyCharm 中设置断点
2. ✅ 访问 Dify 网页触发断点
3. ✅ 在 PyCharm 中查看变量、执行表达式
4. ✅ 单步调试，就像调试本地代码一样

## 🚀 详细步骤

### 步骤1: 安装 debugpy 到容器

打开终端，在项目根目录执行：

```bash
cd /Users/huchangfeng/PycharmProjects/dify-study/docker
```

```bash
# 进入 API 容器
docker-compose exec api bash

# 安装 debugpy
pip install debugpy

# 退出容器
exit
```

### 步骤2: 重启服务使配置生效

```bash
# 停止所有服务
docker-compose down

# 后台启动服务
docker-compose up -d
```

等待服务启动（大约1-2分钟），你会看到类似输出：
```bash
[+] Running 8/8
 ✔ Container dify-redis-1     Started
 ✔ Container dify-db-1        Started  
 ✔ Container dify-api-1       Started
 ...
```

### 步骤3: 验证调试端口

检查调试端口是否开放：
```bash
# 检查容器日志，应该看到调试信息
docker-compose logs api | grep "调试"
```

你应该看到：
```
🐛 启用远程调试，端口: 5678
🐛 等待调试器连接...
```

### 步骤4: PyCharm 配置（关键步骤）

#### A. 打开运行配置
1. 在 PyCharm 中，点击顶部菜单 `Run`
2. 选择 `Edit Configurations...`
3. 弹出配置对话框

#### B. 创建调试配置
1. 点击左上角的 `+` 号
2. 在下拉菜单中选择 `Python Debug Server`
   
   💡 **找不到这个选项？**
   - **PyCharm Professional**: 直接有这个选项
   - **PyCharm Community**: 选择 `Python` → 然后选择合适的远程配置

#### C. 配置参数（重要）

填写以下信息：
```
Name: Dify-Remote-Debug
IDE host name: localhost  
Port: 5678
```

**路径映射配置**（这是关键）：
点击 `Path mappings` 右侧的文件夹图标，然后添加：

```
Local Path:  /Users/huchangfeng/PycharmProjects/dify-study/api
Remote Path: /app/api
```

#### D. 保存配置
点击 `OK` 保存配置

### 步骤5: 开始调试（激动人心的时刻）

#### A. 设置断点
1. 打开文件：`/Users/huchangfeng/PycharmProjects/dify-study/api/app_factory.py`
2. 找到 `create_app()` 函数（大约第50行左右）
3. 在函数内的某一行左侧点击，设置红色断点 ●

#### B. 启动调试器
1. 在 PyCharm 顶部，确保选择了 `Dify-Remote-Debug` 配置
2. 点击 🐛 绿色调试按钮（或按 Shift+F9）
3. 在底部 Debug 窗口中，你会看到：
   ```
   Starting debug server at port 5678...
   Waiting for debugger to attach...
   ```

#### C. 触发断点
1. 打开浏览器，访问：`http://localhost/console`
2. 回到 PyCharm，你会发现程序在断点处停止了！🎉

### 步骤6: 享受调试体验

现在你可以：

**查看变量**：
- 在左下角的 Variables 窗口查看当前作用域的所有变量

**执行表达式**：
- 在 Debug 窗口下方的 Console 中输入 Python 代码，比如：
  ```python
  print("Hello from debugger!")
  app.config
  ```

**单步调试**：
- F8: 单步执行（不进入函数内部）
- F7: 进入函数内部
- Shift+F8: 跳出当前函数
- F9: 继续执行到下个断点

## 🔧 常见问题解决

### Q1: "Connection refused" 错误
```bash
# 检查容器是否启动
docker-compose ps

# 检查端口是否开放
docker-compose logs api | grep 5678
```

### Q2: 断点不触发
```bash
# 检查路径映射是否正确
# 本地路径：你的项目路径
# 远程路径：/app/api
```

### Q3: 调试器连接慢
在 `.env` 文件中设置：
```
DEBUG_WAIT_FOR_CLIENT=false
```

## 🎓 调试技巧进阶

### 1. 条件断点
右键断点 → `More` → 在 `Condition` 中输入条件，如：
```python
request.method == "POST"
```

### 2. 日志断点
右键断点 → 取消 `Suspend` → 勾选 `Log message to console`

### 3. 异常断点
`Run` → `View Breakpoints` → `+` → `Python Exception Breakpoint`

### 4. 调试不同的 API 端点
试试这些 URL 来触发不同的代码路径：
- `http://localhost/console/api/setup`
- `http://localhost/console/api/workspaces/current`
- `http://localhost/v1/chat-messages`

## 🌟 成功标志

如果你看到：
1. ✅ PyCharm Debug 窗口显示 "Connected"
2. ✅ 访问网页时程序在断点处暂停
3. ✅ 可以查看变量和执行表达式
4. ✅ 可以单步调试

恭喜！你已经成功设置了远程调试！🎉

## 💡 下一步

现在你可以：
1. 研究 Dify 的核心业务逻辑
2. 理解请求处理流程
3. 学习 Flask 应用架构
4. 自定义功能开发

调试愉快！有问题随时问我 😊