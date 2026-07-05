# 目录结构

## Server 端（ThinkPHP8）

```
buildadmin/
├── app/                        # 应用目录
│   ├── admin/                  # 后台管理应用
│   │   ├── controller/         # 控制器
│   │   ├── model/              # 模型
│   │   ├── validate/           # 验证器
│   │   └── library/            # Traits、Auth、CRUD 辅助
│   ├── api/                    # 前台会员端 API
│   └── common/                 # 公共应用
│       ├── controller/         # Backend.php、Api.php、Frontend.php 基类
│       ├── model/              # 共享模型
│       └── library/            # Auth.php 等
├── config/                     # 配置目录
│   ├── buildadmin.php          # 系统配置
│   ├── terminal.php            # 终端配置
│   └── database.php            # 数据库配置
├── database/                   # 数据库迁移（Phinx）
├── extend/                     # 扩展目录（\ba\ 命名空间）
├── public/                     # 运行目录
│   ├── static/                 # 静态资源
│   └── storage/                # 上传文件存储
├── runtime/                    # 运行时（缓存、日志）
└── web/                        # WEB 端源代码
```

## Web 端（Vue3）

```
web/src/
├── api/                        # 接口请求函数
│   ├── common.ts               # 公共 URL 定义和请求方法
│   ├── backend/                # 后台接口方法
│   └── frontend/               # 前台接口方法
├── components/                 # 全局组件
│   ├── baInput/                # 输入组件封装
│   ├── contextmenu/            # tabs 右击菜单
│   ├── formItem/               # 表单项（结合 baInput）
│   ├── icon/                   # 字体图标组件
│   ├── table/                  # 表格封装
│   ├── mixins/                 # 可供模块混入的代码
│   ├── clickCaptcha/           # 点选文字验证码
│   └── terminal/               # 终端
├── lang/                       # 语言包
│   ├── autoload.ts             # 按需加载映射表
│   ├── globs-en.ts             # 全局英文
│   ├── globs-zh-cn.ts          # 全局中文
│   ├── backend/                # 后台页面语言包
│   └── frontend/               # 前台页面语言包
├── layouts/                    # 布局
│   ├── backend/                # 后台布局（含四种方案）
│   ├── frontend/               # 前台布局
│   └── common/                 # 公共布局组件
├── router/                     # 路由
│   └── static.ts               # 静态路由配置
├── stores/                     # Pinia 状态管理
│   ├── adminInfo.ts            # 管理员资料
│   ├── config.ts               # 布局配置
│   ├── navTabs.ts              # 后台标签页
│   ├── terminal.ts             # 终端
│   ├── constant/cacheKey.ts    # 缓存 Key 定义
│   └── interface/index.ts      # 接口类型定义
├── styles/                     # 样式表
│   ├── var.scss                # CSS 变量定义
│   ├── dark.scss               # 暗黑模式变量
│   ├── element.scss            # Element Plus 样式覆盖
│   └── mixins.scss             # SCSS mixins
├── utils/                      # 工具库
│   ├── axios.ts                # 网络请求封装
│   ├── baTable.ts              # 表格管家类
│   ├── common.ts               # 公共方法
│   ├── directives.ts           # 指令定义
│   ├── validate.ts             # 表单验证方法
│   └── router.ts               # 路由辅助
└── views/                      # 页面视图
    ├── backend/                # 后台页面
    ├── frontend/               # 前台页面
    └── common/                 # 公共页面
```
