
# 📚 WebSearch 后端服务


---

## 🧰 技术栈

* **Java 17 + Spring Boot**
* **MariaDB**
* **JWT 安全认证**
* **Docker / Docker Compose / Kubernetes 兼容**
* **RestTemplate 调用外部搜索与爬虫服务**
* **Swagger 文档**

---

## 🗂️ 项目结构

```
.
├── Dockerfile                     # Spring Boot 镜像构建配置
├── docker-compose.yml            # 多服务组合部署（含数据库）
├── init_db/                      # 数据库初始化 SQL 脚本
│   └── create_user_database.sql
├── k8s/                          # K8s yaml 清单（留空或添加你的部署配置）
├── src/                          # Spring Boot 项目源码
├── application.properties        # Spring 配置（支持环境变量注入）
└── pom.xml                       # Maven 构建文件
```

---

## 🚀 快速启动

### 1️⃣ 使用 Docker Compose（本地调试）

```bash
docker-compose up --build
```

服务默认暴露：

* 后端服务：[http://localhost:8080](http://localhost:8080)
* 数据库服务：`localhost:3306`，用户名密码为 `WebSearch` / `WebSearch123!`

数据库启动后会自动创建以下表：

* `user`
* `search_history`
* `user_search_count`
* `bookmark`

如需预初始化，请修改 `init_db/create_user_database.sql` 并挂载到数据库容器中。

---

### 2️⃣ 使用 K8s 部署

镜像已上传至 DockerHub，可直接拉取：

```yaml
image: bertha6651/web_backend:latest
```

你可以使用以下清单示例部署服务：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-backend
  labels:
    app: web-backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web-backend
  template:
    metadata:
      labels:
        app: web-backend
    spec:
      containers:
        - name: web-backend
          image: bertha6651/web_backend:latest
          ports:
            - containerPort: 8080
          env:
            - name: SPRING_DATASOURCE_URL
              value: jdbc:mariadb://mariadb:3306/WebSearch
            - name: SPRING_DATASOURCE_USERNAME
              value: WebSearch
            - name: SPRING_DATASOURCE_PASSWORD
              value: WebSearch123!
            - name: CRAWLER_URL
              value: http://crawler:3000
            - name: SEARCHENGINE_URL
              value: http://search-engine:3001/search
            - name: JWT_SECRET
              value: MySuperSecretJWTKey
```

---

## 🔐 配置说明（支持环境变量）

| 配置项                          | 描述           | 默认值（可覆盖）                                |
| ---------------------------- | ------------ | --------------------------------------- |
| `SPRING_DATASOURCE_URL`      | 数据库连接 URL    | `jdbc:mariadb://mariadb:3306/WebSearch` |
| `SPRING_DATASOURCE_USERNAME` | 数据库用户名       | `WebSearch`                             |
| `SPRING_DATASOURCE_PASSWORD` | 数据库密码        | `WebSearch123!`                         |
| `CRAWLER_URL`                | 外部爬虫服务地址     | `http://localhost:3000`                 |
| `SEARCHENGINE_URL`           | 外部搜索引擎服务地址   | `http://localhost:3001/search`          |
| `JWT_SECRET`                 | JWT 密钥       | 默认内置                                    |
| `JWT_EXPIRATION`             | JWT 过期时间（毫秒） | `86400000`（1天）                          |

---

## 📎 API 文档

启动后访问：

```
http://localhost:8080/swagger-ui/index.html
```

或者查看  `./api.markdown`

---

## 📦 Docker 镜像构建说明

你使用的是 **两阶段构建** Dockerfile：

```dockerfile
# 第一阶段：构建
FROM openjdk:17-jdk-slim AS builder
RUN apt-get update && apt-get install -y maven
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn clean package -DskipTests

# 第二阶段：运行
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY --from=builder /app/target/WebSearch-0.0.1-SNAPSHOT.jar app.jar
ENV SPRING_DATASOURCE_URL=...
CMD ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

---

## 🛢️ 数据库初始化

`init_db/create_user_database.sql` 中包含所有所需表定义，可以在 Kubernetes 或 Compose 中通过 `initContainers` 或挂载方式导入。

---

## 📌 注意事项

* 请确保你的外部服务（爬虫 / 搜索引擎）地址在 K8s 网络中可访问。
* 本项目默认允许跨域访问，可根据实际生产环境添加 CORS 策略。

---

