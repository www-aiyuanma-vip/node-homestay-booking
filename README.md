## 特色民宿 · 短租预订小程序

## 一、项目功能详细介绍

本项目面向特色民宿短租场景，由微信小程序房客端、Vue3 运营管理端与 Express + MySQL 后端组成。房客在小程序浏览地区与分类房源、按日历选入住退房、报价下单并用钱包支付；运营人员在后台维护房源与日历价格库存、处理订单与评价、运营 Banner/公告/优惠券等。

### 微信小程序（房客端）

- 首页：轮播 Banner、地区入口、热门/推荐房源列表，进入列表或详情。
- 房源列表：按地区、分类、关键词筛选浏览。
- 房源详情：图集、设施、房东信息、评价预览；弹层日历选择入住/退房（酒店半开区间，不可跨不可订日）；本地校验后报价；收藏；预约窗口限制为今天起未来两个月内，单次最多 30 晚。
- 确认下单：联系人、优惠券、备注；进入页强制重报；钱包余额支付或占待支付单。
- 订单列表/详情：状态流转查看、支付、取消（按房源免费取消时限）。
- 我的：资料、收藏、优惠券、钱包与充值（本地演示 mock 入账）、意见反馈、公告资讯。
- 登录：仅微信授权登录（须配置小程序 AppID/Secret）。

### 管理后台（admin-vue3）

- 登录页：民宿实景全屏背景；账号密码登录（种子管理员/运营）。
- 工作台：房源/订单/房客等统计与图表，快捷入口。
- 房源管理：增删改查；宜住人数、卧室、床位、卫生间、默认库存等；「价格库存」月历任意月份切换与设价/库存/关房（不受小程序两月窗口限制）；批量改平日/周末价。
- 订单管理：确认、入住、完成、取消、退款等运营操作。
- 房客管理：资料查看与钱包调账。
- 地区/分类、优惠券、Banner、公告、评价审核、意见反馈。
- 系统管理：用户、角色、菜单（RBAC 动态路由）；独立「库存日历」菜单已下线，日历能力并入房源。

### 后端 API（Express）

- 启动时自动检查/导入数据库；为上架房源补齐「今天起约两个月」日历价格（已有行不覆盖）；平日价/周末价取房源字段，库存取房源默认库存。
- 小程序：房源检索与详情、日历不可订日、报价、下单扣库存、钱包支付、优惠券、收藏、评价、反馈等。
- 管理端：业务 CRUD、日历批量读写（权限随房源菜单）、上传、工作台统计、系统用户角色菜单。
- 待支付订单超时自动取消并还库存、还券。

## 二、标准目录和备注

以下目录树放在代码块中展示，避免 Markdown 预览折叠空格导致树形错乱。

```text
node-homestay-booking/
├── homestay.sql                         全库建表、菜单角色与业务种子数据
├── README.md                            功能介绍与目录说明（本文件）
├── 运行手册.md                          本地启动、账号与排错
├── SQL自动导入说明.md                   启动时库检查/导入/修补说明
├── .gitignore                           Git 忽略规则
├── .prettierrc                          根级格式化配置
├── 演示材料/                            答辩或演示用素材
│   └── 演示视频.mp4                     系统演示录像
│
├── express/                             后端 API（默认端口 4924）
│   ├── package.json                     依赖与脚本（dev / start / db）
│   ├── app.js                           Express 应用、中间件、超时取消定时任务
│   ├── prettier.js                      后端 Prettier 相关
│   ├── bin/
│   │   └── www                          启动入口：库就绪 → 补日历价 → 监听端口
│   ├── config/
│   │   ├── index.js                     端口、JWT、微信、CORS、DB、超时分钟等
│   │   └── db.js                        MySQL 连接池
│   ├── routes/
│   │   ├── index.js                     路由总挂载
│   │   ├── dashboard.js                 工作台统计
│   │   ├── upload.js                    文件上传
│   │   ├── sys/                         系统管理路由
│   │   │   ├── user.js                  后台用户
│   │   │   ├── role.js                  角色
│   │   │   └── menu.js                  菜单
│   │   ├── biz/                         管理端业务路由
│   │   │   ├── homestay.js              房源
│   │   │   ├── calendar.js              日历价格库存（权限同房源菜单）
│   │   │   ├── order.js                 订单运营
│   │   │   ├── guest.js                 房客
│   │   │   ├── region.js                地区
│   │   │   ├── category.js              分类
│   │   │   ├── coupon.js                优惠券
│   │   │   ├── banner.js                轮播
│   │   │   ├── notice.js                公告
│   │   │   ├── review.js                评价
│   │   │   └── feedback.js              意见反馈
│   │   └── web/
│   │       └── index.js                 小程序端全部接口
│   ├── controllers/
│   │   ├── dashboardController.js       工作台数据聚合
│   │   ├── sys/                         用户/角色/菜单控制器
│   │   ├── biz/                         管理端业务控制器（含 calendarController）
│   │   └── web/
│   │       ├── authController.js        微信登录与资料
│   │       ├── homeController.js        首页房源日历等
│   │       ├── orderController.js       报价下单支付取消
│   │       └── guestBizController.js    钱包券收藏反馈等
│   ├── scripts/
│   │   ├── init_db.js                   建库、导入 SQL、stock/菜单等修补
│   │   └── ensure_calendar.js           启动补齐约两月日历价（已有行跳过）
│   ├── util/
│   │   ├── orderHelper.js               计价、库存锁、晚数与两月窗口校验
│   │   ├── auth.js                      JWT 与菜单权限中间件
│   │   ├── wechat.js                    微信 jscode2session
│   │   ├── role.js                      角色辅助
│   │   ├── crypto.js                    加解密辅助
│   │   └── index.js                     工具导出
│   ├── services/                        预留业务服务目录
│   └── public/images/                   静态与上传资源根
│       ├── biz/
│       │   ├── banner/                  Banner 种子图
│       │   ├── region/                  地区封面种子图
│       │   ├── homestay/                房源封面与相册种子图
│       │   └── host/                    房东头像种子图
│       ├── default/                     默认图
│       └── file/                        上传落盘目录
│
├── admin-vue3/                          Vue3 + Element Plus 管理端（默认 7842）
│   ├── package.json                     前端依赖与 Vite 脚本
│   ├── vite.config.js                   开发服务器与别名
│   ├── index.html                       Vite HTML 入口
│   ├── eslint.config.js                 ESLint
│   ├── jsconfig.json                    路径提示
│   ├── public/
│   │   └── favicon.ico                  站点图标
│   └── src/
│       ├── main.js                      应用入口、全局注册
│       ├── App.vue                      根组件
│       ├── assets/
│       │   ├── login.jpg                登录页全屏背景
│       │   ├── theme.css                主题色变量
│       │   ├── base.css                 基础样式
│       │   └── main.css                 全局样式入口
│       ├── api/
│       │   ├── index.js                 API 统一导出
│       │   ├── request.js               Axios 实例与 401 处理
│       │   ├── biz.js                   业务接口（含日历）
│       │   ├── sys.js                   系统管理接口
│       │   └── demo.js                  演示用接口残留
│       ├── router/
│       │   ├── index.js                 基础路由、守卫、resetRouter
│       │   └── dynamicRoutes.js         按菜单动态加载页面
│       ├── stores/
│       │   ├── user.js                  登录态与菜单
│       │   └── role.js                  角色相关
│       ├── hooks/
│       │   └── useRole.js               是否管理员等
│       ├── layouts/
│       │   └── MainLayout.vue           侧栏布局与退出
│       ├── components/
│       │   ├── UploadFile.vue           图片上传（v-model）
│       │   ├── Editor.vue               富文本编辑
│       │   └── markedHtml.vue           Markdown/HTML 安全展示
│       ├── utils/
│       │   ├── media.js                 媒体 URL 补全
│       │   ├── formatters.js            格式化
│       │   ├── constants.js             常量
│       │   └── crypto.js                前端加密占位
│       └── views/
│           ├── Login.vue                登录页
│           ├── Dashboard.vue            工作台统计与图表
│           ├── 404.vue                  未匹配路由
│           ├── biz/
│           │   ├── Homestay.vue         房源 CRUD + 任意月价格库存日历
│           │   ├── Order.vue            订单运营
│           │   ├── Guest.vue            房客与调账
│           │   ├── Region.vue           地区
│           │   ├── Category.vue         分类
│           │   ├── Coupon.vue           优惠券
│           │   ├── Banner.vue           轮播
│           │   ├── Notice.vue           公告（HTML 入库消毒）
│           │   ├── Review.vue           评价审核
│           │   └── Feedback.vue         意见反馈
│           └── system/
│               ├── UserManage.vue       后台用户
│               ├── RoleManage.vue       角色
│               ├── MenuManage.vue       菜单
│               └── userCenterMange.vue  个人中心
│
└── min/                                 微信原生小程序房客端
    ├── README.md                        小程序端说明
    ├── app.js                           小程序入口逻辑
    ├── app.json                         页面路由与窗口配置
    ├── app.wxss                         全局样式
    ├── project.config.json              开发者工具项目配置
    ├── project.private.config.json      本机私有配置
    ├── sitemap.json                     索引配置
    ├── utils/
    │   ├── config.js                    baseUrl（对接 Express）
    │   ├── request.js                   请求封装、401 跳登录
    │   ├── api.js                       业务 API 与上传
    │   ├── stayDate.js                  入住区间校验（两月窗、最多 30 晚）
    │   ├── media.js                     媒体地址处理
    │   ├── feedback.js                  Toast/确认框
    │   ├── util.js                      时间等通用工具
    │   ├── date.js                      日期辅助
    │   └── markdown.js                  资讯 Markdown 辅助
    ├── components/
    │   ├── dateRangeCalendar/           入住/退房区间日历（主用）
    │   ├── weekCalendar/                周历组件（保留）
    │   └── wxParse/                     富文本/公告 HTML 解析
    ├── images/
    │   ├── tabbar/                      底部导航图标
    │   ├── biz/                         默认头像、默认房源图
    │   ├── banner.jpg                   本地占位 Banner
    │   ├── star.png / star1.png         评分星标
    │   ├── checked.png                  选中态
    │   └── right.png                    箭头
    └── pages/
        ├── index/                       首页 Banner、地区、推荐房源
        ├── list/                        房源列表筛选
        ├── detail/                      详情、报价、日历弹层、收藏
        ├── confirm/                     确认下单与支付
        ├── orderList/                   订单列表
        ├── orderDetail/                 订单详情
        ├── auth/                        微信授权登录
        ├── my/                          个人中心入口
        ├── userInfo/                    资料编辑
        ├── favorite/                    我的收藏
        ├── coupon/                      我的优惠券
        ├── wallet/                      钱包与流水
        ├── walletAdd/                   充值（本地 mock）
        ├── newsList/                    公告列表
        ├── newsDetail/                  公告详情
        └── feedback/                    意见反馈
```
---

## 获取完整源码

- 网站：[AI源码](https://www.aiyuanma.vip)
- 本项目详情：[https://www.aiyuanma.vip/posts/node-homestay-booking](https://www.aiyuanma.vip/posts/node-homestay-booking)

## 联系方式

- QQ：861077046
- 邮箱：861077046@qq.com

> 本文由 AI源码 自动同步，完整源码与技术支持请访问官网。
