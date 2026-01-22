# 重要更新



## v0.3.1
on-panda 0.3.x 版本相比 0.2.x 版本破坏性更新汇总：
- 拆分为完整的 @on-panda/on-panda 和纯 peer 依赖的 @on-panda/on-panda-core (需要 @mdit/plugin-katex)
- 不用 js import 导入 png gif 直接用 URL '/img/'
- public/img 都统一 `on-panda-` 开头命名
- 缩减了存储 assets 存储：gif 改为 mp4，logo 缩减图片尺寸
- pnpm 从 --app 启动改为 `:web` `:core`
- 删除 assets/style.css 改为 /style.css
- 拆分两个 index.js，index.core.js 不包含 'element-plus/dist/index.css'
- index.js 删掉 default ，plugin 改名 onPandaPlugin
- OnPanda 和 App.vue 都改名 OnPandaWeb
- 修改 custom.js 载入方案：使用 .env.local + WEB_IMPORT_CUSTOM_CODE
- 新增 VITE_ON_PANDA_WEB_RUNTIME_IMPORT=xxx/on-panda-web-runtime.js，让 secret 不被包含在构建中


