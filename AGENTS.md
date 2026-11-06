# AGENTS.md

rogsoft 是 koolshare 1.5 代软件中心的插件仓库，面向华硕路由器（梅林改 / 官改，平台 hnd / axhnd / axhnd.675x / p1axhnd.675x）。

## 目录结构

每个顶层目录是一个插件模块（共 48 个）：

```text
<module>/
  build.sh              # 唯一的构建入口
  build_<platform>.sh   # 可选，多数是指向 build.sh 的符号链接，少数是独立脚本
  <module>/             # 打入 tar.gz 的包内容（koolcenter 例外，用 softcenter/）
    install.sh  uninstall.sh  version
    bin_*/  scripts-*/  webs/  res/  init.d/
```

- `<module>.tar.gz` 由 build.sh 生成，被 `.gitignore` 忽略。
- `version` 与 `config.json.js` 虽由 build.sh 生成，但**纳入版本控制**，软件中心靠它们读取版本号。
- `checkout-build-files.sh` 用于批量还原被构建改动的 `tar.gz` / `version` / `config.json.js`。

## 平台与架构

| 平台 | `.valid` | 二进制目录 |
| ---- | -------- | ---------- |
| hnd | hnd | bin_32 |
| ipq32 | ipq32 | bin_32 |
| qca | qca | bin_32 |
| mtk | mtk | bin_64 |
| ipq64 | ipq64 | bin_64 |

- 平台判定见 `softcenter/softcenter/scripts/ks_tar_install.sh` 的 `VALID_STRING`；**不在特例名单的机型一律按 `hnd` 处理**（如 RT-AX88U）。
- `.valid` 只在**离线安装**时校验（`ks_tar_install.sh`）；在线安装（`ks_app_install.sh`）的校验是注释掉的。
- 部分机型内核为 64 位但 userspace 是 32 位，默认发 `bin_32`。
- `softcenter/build_all.sh` 会给所有模块写死 `hnd`；要为其它平台出包，必须单独跑对应的 `build_<platform>.sh`。
- `.valid` 由构建脚本写入并被 `.gitignore` 忽略；仓库里残留的被跟踪文件内容可能过期，不影响构建。

## 构建约定

- 同一模块的多个 `build_*.sh` 通常覆盖同一个 `<module>.tar.gz`，**最后执行的那个决定产物平台**。改动前先 `ls -l` 确认它是符号链接还是独立脚本。
- 拷贝 `bin_*` / `scripts-*` 用 `cp -a`，不要用 `cp -rf`：macOS 的 `cp -rf` 会把符号链接解引用成实体文件，包体积暴涨。
- `version` 必须在 `tar` **之前**写入包目录，否则包内版本号停留在上一次构建。
- build.sh 调用 python 用 `python3`；`$BASH_SOURCE[0]` 在 dash 下会 bad substitution，改用 `${BASH_SOURCE:-$0}`。
- 离线包名必须与模块目录同名（`<module>.tar.gz`），`ks_tar_install.sh` 从文件名反推模块名。
- `config.json.js` 里的 `md5` 必须等于 `tar.gz` 的实际 md5。
- 各模块 `install.sh` 普遍带平台守卫（`[ -d /koolshare ] && [ -f /usr/bin/skipd ] && 内核主次版本拼接 >= 41`，如 4.1.x → 41），新增模块应保持一致。

## 仓库约定

- `config.json.js` 是**纯 JSON**，不是 JS：`softcenter/gen_install.py` 用 `json.loads` 读取。不要加 `;` 或改写为 JS 语法，否则软件中心解析失败。
- 仓库里有 48 个同名 `config.json.js`，按 `.js` 后缀交给 TS/ESLint 检查会全量误报 `';' expected`，属预期现象，可忽略。
