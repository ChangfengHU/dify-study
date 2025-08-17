# 🎯 Dify 简化调试指南 - 立即可用！

## ✅ 当前状态
- ✅ Docker 容器正常运行
- ✅ debugpy 已安装
- ✅ 调试端口 5678 已开放
- ✅ Flask 调试器已激活

## 🚀 马上开始调试（3分钟）

### 方法一: 使用 PyCharm 连接现有服务

#### 1. PyCharm 配置
1. **打开 Run → Edit Configurations**
2. **点击 + → Python Debug Server**
3. **填写配置**:
   ```
   Name: Dify-Debug
   IDE host name: localhost
   Port: 5678
   Path mappings: 暂时不设置（我们用容器内直接调试）
   ```

#### 2. 设置断点的简单方法
由于源码挂载有问题，我们用容器内直接编辑的方式：

```bash
# 在容器内编辑文件并添加断点
docker exec -it docker-api-1 bash -c "
cat >> /app/api/controllers/console/auth/login.py << 'EOF'

# 添加调试断点函数
def debug_breakpoint():
    import debugpy
    debugpy.breakpoint()
    
EOF
"
```

#### 3. 在登录处添加断点
```bash
# 编辑登录文件，在关键位置添加调试
docker exec -it docker-api-1 bash -c "
cd /app/api
# 备份原文件
cp controllers/console/auth/login.py controllers/console/auth/login.py.backup

# 在第77行（用户认证处）添加断点
sed -i '77i\\            import debugpy; debugpy.breakpoint()' controllers/console/auth/login.py
"
```

#### 4. 重启服务并开始调试
```bash
docker-compose restart api
```

### 方法二: 使用 Flask 内置调试器（更简单）

当前 Flask 调试器已激活，您可以：

#### 1. 访问错误页面
访问: `http://localhost/console`

如果有错误，会显示详细的调试信息和调用栈。

#### 2. 使用 pdb 断点
在容器内添加 Python 调试断点：

```bash
# 进入容器
docker exec -it docker-api-1 bash

# 编辑登录文件
cd /app/api
vi controllers/console/auth/login.py

# 在第77行添加:
import pdb; pdb.set_trace()
```

## 🎯 推荐的学习调试流程

### 1. 简单开始 - 观察日志
```bash
# 实时观察 API 日志
docker-compose logs -f api
```

### 2. 触发登录请求
打开浏览器访问: `http://localhost/signin`
输入任意邮箱密码，观察日志输出。

### 3. 使用 print 调试法
最简单的方式，直接在代码中添加打印：

```bash
docker exec -it docker-api-1 bash -c "
cd /app/api
# 在认证函数添加打印
sed -i '77i\\            print(f\"🔍 用户认证: {args[\\\"email\\\"]}\")' controllers/console/auth/login.py
"

# 重启查看效果
docker-compose restart api
```

## 🛠️ 实用调试技巧

### 查看请求参数
```python
# 在 login.py 第51行后添加
print(f"📨 登录请求参数: {args}")
```

### 查看数据库查询
```python  
# 在认证逻辑中添加
print(f"🔍 查找用户: {args['email']}")
```

### 查看异常信息
```python
# 在异常处理块添加
except Exception as e:
    print(f"❌ 登录错误: {type(e).__name__}: {e}")
    raise
```

## 📍 关键调试点

### 1. 登录入口
**文件**: `/app/api/controllers/console/auth/login.py`
**行号**: 43 (post 方法)

### 2. 参数解析
**行号**: 51 (`args = parser.parse_args()`)

### 3. 用户认证
**行号**: 77 (`AccountService.authenticate`)

### 4. Token 生成
**行号**: 102 (`AccountService.login`)

## 🚀 立即测试

运行以下命令测试登录调试：

```bash
# 1. 添加简单的调试打印
docker exec docker-api-1 bash -c "
cd /app/api
sed -i '43a\\        print(\"🔍 收到登录请求\")' controllers/console/auth/login.py
sed -i '51a\\        print(f\"📨 请求参数: {args}\")' controllers/console/auth/login.py  
sed -i '77a\\        print(f\"🔐 开始认证用户: {args[\\\"email\\\"]}\")' controllers/console/auth/login.py
"

# 2. 重启服务
docker-compose restart api

# 3. 等待启动完成
sleep 10

# 4. 触发登录请求
curl -X POST http://localhost/console/api/login \\
  -H "Content-Type: application/json" \\
  -d '{
    "email": "test@example.com",
    "password": "123456"
  }'

# 5. 查看调试输出
docker-compose logs --tail=20 api
```

## 🎉 成功标志

如果您看到类似输出：
```
🔍 收到登录请求
📨 请求参数: {'email': 'test@example.com', 'password': '123456', ...}
🔐 开始认证用户: test@example.com
```

恭喜！您已经成功开始调试 Dify 登录流程了！

## 💡 下一步

1. **理解登录流程**: 跟踪完整的登录逻辑
2. **学习数据模型**: 查看用户表结构和关系
3. **探索应用创建**: 调试应用创建流程
4. **深入工作流引擎**: 理解 Dify 的核心功能

调试愉快！🎊