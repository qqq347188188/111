# GMInject.dylib — GitHub 上传编译版

把这个文件夹里的**全部内容**传到 GitHub 仓库根，用 Actions 编出 `GMInject.dylib`。

## 文件说明
| 文件 | 作用 |
|---|---|
| `GMInject.m` | dylib 源码，IL2CPP 反射注入，配置驱动 |
| `gm_offsets.h` | 占位头（反射模式不需要真实偏移） |
| `build.sh` | 编译脚本（arm64 + arm64e，ad-hoc 签名） |
| `Makefile` | 本地构建入口（`make`） |
| `.github/workflows/build-dylib.yml` | **Actions 编译 workflow（隐藏目录，上传时别漏）** |
| `GMConfig.json` | 只加检阅车模型 x99（测试用） |
| `GMConfig_数值验证.json` | 数值全加，先验证注入链路用 |
| `GMInject.plist` | 越狱注入的 filter（Bundles=com.midagame.truckstar） |

## 上传（务必用 git 命令行，网页拖拽会漏掉 .github）
```bash
cd upload-github-dylib
git init
git add -A
git commit -m "GMInject dylib"
git branch -M main
git remote add origin https://github.com/<你>/<新空仓库>.git
git push -u origin main
```
仓库必须设 **Public**。

## 编译
GitHub → Actions → `build-dylib` → **Run workflow** → runner 选 `macos-13`（不行再换 `macos-14`/`macos-15`/`macos-latest`）→ Run。
1–2 分钟后 Artifacts 下载 `GMInject.dylib`。

## 如果一直卡 "Starting job"（你之前的情况）
那是 GitHub macOS runner **排队**，不是配置问题。仓库结构和 workflow 都正确时也会排 1 小时+：
- 取消当前 run → 重新 Run workflow → 换 runner 标签再试
- `Settings → Billing` 查 Actions 分钟数是否耗尽
- 错峰（夜里/周末更易分到）
- 或借一台 Mac 本地 `bash build.sh` 直接出 dylib（不用 GitHub）

## 拿到 dylib 后怎么用
1. **注入**
   - 越狱：`GMInject.dylib` + `GMInject.plist` 放 `/Library/MobileSubstrate/DynamicLibraries/`（iOS 16 rootless 自动映射到 `/var/jb/...`）
   - TrollStore：用 `GMInject/inject_ipa.py` 给游戏 Mach-O 加 `@executable_path/Frameworks/GMInject.dylib`
2. **放配置**：把 `GMConfig.json`（或 `GMConfig_数值验证.json`）放进 app 沙盒 `Documents/`
3. **冷启动游戏**，看 `Documents/GMInject.log`：
   - 先跑数值验证版：日志出现 `AddCoin -> OK`、蓝钻变 9.99 亿 = 链路通
   - 再换检阅车配置：`[0] AddItem -> OK` 后背包多检阅车模型
