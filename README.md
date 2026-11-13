# EndfieldHUD 会议截稿

「会议截稿」是 EndfieldHUD 的 Windows x64 源码补丁。它在 HUD 中央显示学术会议的论文截稿日期。EndfieldHUD 由 **DDDuoDuo** 创建。本补丁是独立维护的非官方扩展。[EndfieldHUD 原项目](https://github.com/DDDuoDuo/EndfieldHUD)

## 使用方法

1. 按下文安装，在新生成的 `out\bin` 目录双击 `EndfieldHUD.exe`。如果旧版正在运行，先从系统托盘退出。
2. 按 <kbd>Ctrl</kbd> + <kbd>`</kbd> 打开 HUD。如果你已更改快捷键，使用自己的设置。
3. 滚动 HUD 右侧模块列表，点击「电源」前的 **「会议截稿」**。
4. 中央列表按日期排列，显示本地日期今天及之后的会议，每页 6 条。点击「上一页」或「下一页」翻页，点击「刷新」重新获取。刷新失败时会保留本次运行中已加载的列表。

首次进入页面会请求 Call4Papers 页面使用的公开 `ccf.json`。上游提供的是日期，HUD 按原日期显示。点击页眉的「Call4Papers」可打开来源网页。[Call4Papers 会议截稿日历](https://call4papers.org/deadlines)

## 安装

需要 Windows 11 x64、Git，以及安装了 C++ 桌面开发组件和 Windows SDK 的 Visual Studio。可使用 Visual Studio 自带的 CMake 和 Ninja。补丁的基准是 EndfieldHUD 提交 `97cab1497e3993d2f5018717448073e977ce1e69`，应用后需重新编译 `EndfieldHUD.exe`。[基准提交](https://github.com/DDDuoDuo/EndfieldHUD/commit/97cab1497e3993d2f5018717448073e977ce1e69)

### 1. 获取源码和运行资源

下载本仓库的 [conference-deadlines.patch](conference-deadlines.patch)，保存到工作目录。从原项目的 v1.3.1 Release 下载 **`EndfieldHUD-1.3.1-build21-windows-x64-portable.zip`**，解压到同一工作目录下的 `official` 文件夹。解压后应能找到 `official\EndfieldHUD\resources\shell\runtime-input\runtime-input.json`。[官方便携包](https://github.com/DDDuoDuo/EndfieldHUD/releases/tag/v1.3.1)

### 2. 应用补丁并构建

在 Visual Studio 的 **Developer PowerShell** 中进入工作目录，执行下面的命令。示例使用 Ninja，从官方便携包读取 HUD 运行资源，再由构建系统生成与本次编译匹配的资源清单。

```powershell
git -c core.autocrlf=false clone https://github.com/DDDuoDuo/EndfieldHUD.git EndfieldHUD-source
Set-Location EndfieldHUD-source
git switch --detach 97cab1497e3993d2f5018717448073e977ce1e69
git apply --check ..\conference-deadlines.patch
git apply ..\conference-deadlines.patch

$shell = (Resolve-Path '..\official\EndfieldHUD\resources\shell').Path
$inputs = Join-Path $PWD 'out\runtime-inputs'
New-Item -ItemType Directory -Force -Path $inputs | Out-Null
Copy-Item -LiteralPath (Join-Path $shell 'runtime-input') -Destination (Join-Path $inputs 'runtime-compiled-animation') -Recurse
Copy-Item -LiteralPath (Join-Path $shell 'catalog.ehscene') -Destination (Join-Path $inputs 'profile-outline-catalog.ehscene')
Copy-Item -LiteralPath (Join-Path $shell 'cursor.png') -Destination (Join-Path $inputs 'source-cursor.png')

cmake -S windows -B out -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_TRY_COMPILE_CONFIGURATION=Release "-DEHUD_APP_INPUT_DIR=$inputs" -DBUILD_TESTING=ON
cmake --build out --parallel --target EndfieldHUD conference_deadlines_tests
ctest --test-dir out --output-on-failure -R '^conference_deadlines_contracts$'
cmake --install out --prefix out/bin --component EndfieldHUD
```

如果 `git apply --check` 报错，请检查源码是否处于指定提交，再继续。运行时请把 `out\bin\EndfieldHUD.exe`、`resources` 和 `fonts` 放在同一目录。

### 使用独立配置试跑

在源码目录执行以下命令，将测试数据写到 `out\test-user-data`。

```powershell
$data = Join-Path $PWD 'out\test-user-data'
New-Item -ItemType Directory -Force -Path $data | Out-Null
& .\out\bin\EndfieldHUD.exe --data-root $data --no-onboarding --power
```

## 署名与许可

- EndfieldHUD 原作者 **DDDuoDuo**，原项目原创代码采用 MIT 许可证。[原项目](https://github.com/DDDuoDuo/EndfieldHUD) · [许可证](LICENSE)
- 会议数据来自 **Call4Papers**。[数据来源](https://call4papers.org/deadlines)
- 游戏素材、商标和第三方组件保留原有权利与许可。本仓库发布源码补丁，运行资源由用户从原项目获取。[EndfieldHUD 完整鸣谢](https://github.com/DDDuoDuo/EndfieldHUD/blob/97cab1497e3993d2f5018717448073e977ce1e69/CREDITS.md)
