# harness-lzcapp

[Harness](https://github.com/harness/harness) 的 LazyCat LPK 打包 —— 开源开发者平台（代码托管 + CI/CD + 托管开发环境 + 制品仓库）。

## 形态

- 单容器镜像：`harness/harness`，面板 3000，Git 服务 3022
- 镜像交付：mirror 模式（`docker.1ms.run/harness/harness:{tag}`），digest 校验开启
- 版本发现：Docker Hub 严格三段式 SemVer tag，排除 latest / unstable / test-unstable
- 仅喵喵商店；仅 PC 平台

## 部署参数与凭据

| 项 | 说明 |
|------|------|
| `admin_email` / `admin_password` | 主体管理员账号，自动生成，登录页自动填充 |
| `GITNESS_HTTP_HOST` | 固定为 0.0.0.0，容器外才能访问 |

## Docker socket

Harness 的流水线在 Docker 容器内运行，需要宿主 daemon socket，
因此通过 `lzc-build.yml` 的 `compose_override` 保留 `/var/run/docker.sock` 挂载。
挂载后需在应用内验证 daemon API 连通性；路径与权限以目标主机实际为准。

## 持久化

`/lzcapp/var/data` → `/data`（数据库、仓库、制品）。官方强烈建议使用卷，否则容器停止即丢数据。

## 结构

- `package.yml` — LPK v2 包元数据 + 平台限制
- `lzc-manifest.yml` — 单服务、免密登录注入、文件选择器注入
- `lzc-deploy-params.yml` — 管理员凭据部署参数
- `lzc-build.yml` — 构建配置 + docker.sock 的 compose_override
- `content/lazycat-injects/` — 文件选择器拦截脚本
- `.github/lazycat-action.yml` — 镜像版本源，mirror 交付，仅私有商店
- `.github/workflows/lazycat.yml` — 每日自动跟版 + 发布喵喵商店
