# 农产品信息展示与售卖管理系统（乡村助农实战项目）

> 课程实训 · 乡村助农类特色实战项目 · Spring Boot 3 + Vue 3 全栈

面向乡村助农场景的农产品信息展示与售卖管理系统：农户/管理员可上架农产品、公示产地溯源，系统支持分类展示、下单与订单流转、销量与销售额统计，并内置管理员 / 农户 / 消费者三类角色与登录鉴权。

## ✨ 已实现功能

- ✅ **用户登录与权限**：管理员 / 农户 / 消费者三角色，简易 Token 鉴权（请求拦截器 + 角色注解），密码 BCrypt 加密；消费者可自助注册。菜单与接口按角色放行。
- ✅ **角色化视图**：消费者端为农产品展示与「立即购买」下单；管理员端 / 农户端为管理表单与表格，农户仅管理自己的农产品与相关订单。
- ✅ **农产品上架/管理**：新增、编辑、上下架、删除，含价格、库存、单位、封面（图片上传）、描述
- ✅ **分类展示**：分类增删改，产品按分类筛选
- ✅ **产地信息公示**：产地/基地、农户、联系方式、资质认证、产地介绍的维护与展示
- ✅ **订单管理**：模拟下单（自动扣减库存、累加销量）、订单明细、状态流转（待付款→待发货→已发货→已完成/已取消，取消自动回滚库存与销量）
- ✅ **销量统计**：经营总览（商品数/订单数/累计销量/销售额）、各品类销量与销售额、销量 TOP 商品（按管理员全局 / 农户本人范围统计）

## 🛠 技术栈

| 层级 | 技术 |
| --- | --- |
| 后端 | Spring Boot 3.3.5、Java 17、Spring Data JPA、MySQL 8、Maven、Lombok、Spring Security Crypto（BCrypt） |
| 前端 | Vue 3、Vite 5、Element Plus、Vue Router、Axios |

## 📁 目录结构

```
agri-market-system/
├── agri-market-server/              # Spring Boot 后端
│   ├── pom.xml
│   └── src/main/
│       ├── java/com/cmh/agrimarket/
│       │   ├── AgriMarketApplication.java
│       │   ├── common/        # 统一返回、全局异常、跨域、鉴权拦截器、角色注解、当前用户、演示数据初始化
│       │   ├── entity/        # JPA 实体（用户/角色/产品/分类/产地/订单/订单项）
│       │   ├── dto/           # 请求与统计返回对象（含登录/注册）
│       │   ├── repository/    # Spring Data JPA
│       │   ├── service/       # 业务逻辑（含认证、Token）
│       │   └── controller/    # REST 接口（/api/**，含 /api/auth/**）
│       └── resources/application.yml
├── agri-market-web/                 # Vue 3 前端
│   ├── package.json、vite.config.js、index.html
│   └── src/
│       ├── main.js、App.vue
│       ├── api/               # axios 封装与接口（含 auth）
│       ├── stores/            # 会话状态（token + 当前用户，localStorage）
│       ├── router/            # 路由（含守卫与 meta.roles）
│       └── views/             # 登录/注册 + 产品/分类/产地/订单/统计 页面
├── db/                              # 数据库脚本
│   ├── schema.sql       # 建库建表（重建结构，会清空数据）
│   └── seed.sql         # 演示数据（可重复导入）
└── README.md
```

## ✅ 环境要求

- JDK 17+（本机 Maven 已使用 JDK 17）
- Maven 3.6+
- MySQL 8.0（服务需处于运行状态）
- Node.js 18+ 与 npm

## 🚀 快速开始

**1. 配置数据库密码**

打开 `agri-market-server/src/main/resources/application.yml`，把 `spring.datasource.password` 改成你本机 MySQL 的 root 密码。

> 连接 URL 中带 `createDatabaseIfNotExist=true`，启动时若 `agrimarket` 库不存在会自动创建，**无需手动建库**。

**2. 初始化数据库（二选一）**

- **方式 A（推荐，脚本全权管理）**：依次执行建表与导入演示数据
  ```bash
  mysql -u root -p < db/schema.sql
  mysql -u root -p agrimarket < db/seed.sql
  ```
- **方式 B（省心）**：先按第 3 步启动一次后端，让 JPA 自动建表；再导入演示数据
  ```bash
  mysql -u root -p agrimarket < db/seed.sql
  ```

> `db/seed.sql` 可重复执行：每次会先清空业务表再插入，便于重置演示数据。订单未预置，可在前端「模拟下单」体验完整流程。

**3. 启动后端**（端口 8081）

```bash
cd agri-market-server
mvn spring-boot:run
```

> 或在 IDEA 中导入 `agri-market-server` 目录为 Maven 项目，运行 `AgriMarketApplication`。
> 首次启动会自动创建三个演示账号（密码均为 `123456`）：`admin`（管理员）、`farmer`（农户）、`consumer`（消费者），并将演示农产品归属到 `farmer`。

**4. 启动前端**（端口 5173）

```bash
cd agri-market-web
npm install      # 首次执行
npm run dev
```

**5. 访问**：浏览器打开 <http://localhost:5173>，使用上述演示账号登录（消费者也可在登录页自助注册）。

## 🔌 主要接口（前缀 `/api`）

> 除 `/api/auth/**`、`/uploads/**` 外，所有接口均需登录（请求头 `Authorization: Bearer <token>`），写操作按角色放行。

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| POST | `/auth/login`、`/auth/register` | 登录 / 消费者注册，返回 `{ token, user }` |
| GET | `/auth/me` | 当前登录用户信息 |
| POST | `/auth/logout` | 退出登录 |
| GET/POST/DELETE | `/categories` | 分类列表 / 新增·修改（仅管理员）/ 删除（仅管理员） |
| GET/POST/DELETE | `/origins` | 产地列表 / 新增·修改（仅管理员）/ 删除（仅管理员） |
| GET/POST | `/products` | 产品列表（按角色过滤；支持 `categoryId/status/keyword`）/ 新增（管理员·农户） |
| GET/PUT/DELETE | `/products/{id}` | 详情 / 修改（管理员·农户）/ 删除（管理员·农户） |
| PATCH | `/products/{id}/status?status=1\|0` | 上下架（管理员·农户） |
| POST | `/upload` | 图片上传 |
| GET/POST | `/orders` | 订单列表（按角色过滤）/ 创建订单 |
| GET | `/orders/{id}` | 订单明细 |
| PATCH | `/orders/{id}/status?status=PAID` | 变更订单状态（管理员·农户） |
| GET | `/stats/overview`、`/stats/by-category`、`/stats/top-products` | 销量统计（管理员·农户） |

统一返回结构：`{ "code": 0, "message": "success", "data": ... }`，`code=0` 为成功；鉴权失败返回 HTTP 401，权限不足返回 HTTP 403。

## 🔐 角色与权限

| 功能 | 管理员 | 农户 | 消费者 |
| --- | --- | --- | --- |
| 农产品 | 全部，管理表单 | 仅本人产品，管理表单 | 只读展示 + 立即购买 |
| 分类 / 产地 | 增删改 | 不可见 | 不可见 |
| 订单 | 全部 + 状态流转 | 含本人产品的订单 | 我的订单 |
| 销量统计 | 全局 | 本人范围 | 不可见 |

## ❓ 常见问题

- **Lombok 报错**：IDEA 需安装 Lombok 插件并开启 *Settings → Build → Compiler → Annotation Processors → Enable*。
- **连不上 MySQL**：确认 MySQL 服务已启动，并核对 `application.yml` 中的用户名/密码。
- **页面没有数据 / 跳转到登录**：确认已执行 `db/seed.sql` 导入演示数据，并使用演示账号登录（见快速开始第 3 步）。
- **重置演示数据**：`mysql -u root -p agrimarket < db/seed.sql`（先清空再插入，可重复执行；不会重置用户表）。
- **端口被占用**：后端默认 8081、前端默认 5173，可在 `application.yml` / `vite.config.js` 修改。

## 🌱 后续可扩展

支付对接、评价与收藏、数据大屏（ECharts）、农户与产地的多对一绑定、登录日志持久化等。
