# 部署和配置 on-panda-web 教程

## 前置要求
- 安装 Node.js & npm & pnpm
- pnpm 是一个快速、节省磁盘空间的包管理器。安装方法如下：

```bash
# 在 Ubuntu/Debian 上
sudo apt-get install nodejs npm
# 在 macOS 上
brew install node

# 使用 npm 安装 pnpm
npm install -g pnpm
```

## 部署命令
部署 on-panda-web
```bash
pnpm install && pnpm build:web && pnpm preview --host 0.0.0.0
```

