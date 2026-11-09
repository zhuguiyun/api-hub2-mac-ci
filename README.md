# api-hub2 mac 打包工作流仓

GitHub Actions 触发 mac 打包:源码从 CNB 私有仓 `yunone.top/api-hub2` 拉取,macOS runner 上 PyInstaller 打包,zip + latest.json 回传**同一仓库的 `dist/` 目录**(与 Windows 打包约定一致,`config.example.json` 的 `update_url` 即指向 `main` 分支 `dist/latest.json`)。
本仓库只放工作流文件(几 KB),不含源码。

## 一次性配置

1. Settings → Secrets and variables → Actions,配两个 repository secret:
   - `CNB_PULL_TOKEN`:拉源码用,CNB 访问令牌,权限 `repo-code:只读`,仅应用于 `yunone.top/api-hub2`
   - `CNB_PUSH_TOKEN`:回传产物用,CNB 访问令牌,权限 `repo-code:读写`,仅应用于 `yunone.top/api-hub2`
   - 令牌管理页:<https://cnb.cool/profile/token>
2. 源码仓 `yunone.top/api-hub2` 的 `main` 分支即打包基线。

## 触发打包

Actions → build-mac → Run workflow → 选架构 → Run:

- `arm64` = Apple Silicon(M1/M2/M3/M4),跑在 `macos-15`(arm64)
- `x64` = Intel,跑在 `macos-15-intel`(x64)

约 5-10 分钟跑完。

## runner 标签(2026-11-09 迁移)

GitHub 已下线 `macos-13` / `macos-14` 镜像:用旧标签触发时 job 会在 1 秒内被拒,注解为
`The configuration 'macos-14-arm64-us-default' is not supported`(gh run 42550960378)。

现在公开仓可用的 macOS 标准 runner(见 docs:Standard runners for public repositories):

| 架构 | 可用标签 |
|---|---|
| arm64(Apple Silicon) | `macos-15`、`macos-26`、`macos-latest` |
| Intel(x64) | `macos-15-intel`、`macos-26-intel` |

工作流固定用 `macos-15` / `macos-15-intel`(与 macOS 14 最接近的稳定版本);
`-large` / `-xlarge` 属付费 larger runner,本账号会因账单/额度被拒。
job 第一步会校验 runner 真实架构与所选 `arch` 一致,不一致直接失败,避免产出错架构的 zip。

## 取产物

- 首选:CNB 源码仓 dist 目录 <https://cnb.cool/yunone.top/api-hub2/-/blob/main/dist/> 下载 zip(国内直连,快)
- 备份:GitHub 本次 run 页面底部的 Artifacts(30 天自动过期)
- `latest.json`:Windows 用 `x64` 键(build-exe.ps1 维护),mac 用 `darwin-arm64` / `darwin-x64` 键;dist/ 里 mac 包只保留最新一个(不分 mac 架构,新构建顶掉旧的)

## mac 上运行

```bash
unzip api-hub2-<版本>-darwin-<架构>.zip -d api-hub2 && cd api-hub2
xattr -dr com.apple.quarantine .   # 浏览器下载的 zip 带 macOS 隔离标记,解一次即可;终端 curl 下载则无需此步
./api-hub2                          # 默认 8791 端口,配置写同级 config.json
```

## 更新源码后重新打包

源码 push 到 CNB(`yunone.top/api-hub2`)后,回本仓库 Actions 再触发一次即可,无需改任何文件。
