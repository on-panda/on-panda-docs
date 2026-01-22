# onPanda 作为组件集成进你的前端项目
见 packages/integrate-playground 项目
(这个文档也应该和 integrate-playground 放一起)

- 有两个包： 
    - 完整的 @on-panda/on-panda：开箱即用，自己打包了所有依赖
    - 纯 peer 依赖的 @on-panda/on-panda-core：尽可能和 host 共用依赖，减少了包尺寸，需要自行导入 'element-plus/dist/index.css'




## Custom module configuration

You can inject deployment-specific custom logic at build time by pointing Vite to a custom module. The app loads it via `import('./utils/defaultCustom.js')`, and Vite resolves `./utils/defaultCustom.js` to the path from env. Each build target uses its own env key (all read from the repo root `.env.local`):

- `WEB_IMPORT_CUSTOM_CODE` for `apps/on-panda-web`
- `CORE_IMPORT_CUSTOM_CODE` for `packages/on-panda-core`
- `MAIN_IMPORT_CUSTOM_CODE` for the root build

Example `.env.local`:

```dotenv
WEB_IMPORT_CUSTOM_CODE=src/assets/secret/custom.js
```

If the variable is not set, the build falls back to `src/utils/defaultCustom.js` (no-op).

