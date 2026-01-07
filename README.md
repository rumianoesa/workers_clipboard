# workers_clipboard

一个基于 Cloudflare Workers 和 KV 存储构建的简易在线剪贴板服务。可以使用图形化网页前端，在不同设备之间复制和粘贴文本。同时也提供了 HTTP 接口，可直接与后端交互。支持配置安全入口路径，保护服务不被滥用。

## 部署方式

- 创建新的 Workers 项目
- 点击右上角 `编辑代码`
- 复制 [clip.js](https://github.com/rumianoesa/workers_clipboard/blob/main/clip.js) 代码，`部署`
- 创建并绑定 KV 命名空间，变量名使用 `KV`
- 添加变量 `TOKEN` 作为安全入口路径（可选）
- 添加自定义域或路由

## 接口说明

**保存到在线剪贴板的接口路径 `/save`**
 * 请求方式： `POST`
 * 成功返回： `saved`，状态 `200`
 * 失败返回： `failed`，状态 `500`
 * 请求内容： 文本
 * 请求内容为空时返回： `empty`，状态 `400`

**读取在线剪贴板内容的接口路径 `/read`**
 * 请求方式： `GET`
 * 成功返回： 在线剪贴板内容，状态 `200`
 * 失败返回： 空，状态 `400`
 * 在线剪贴板内容为空时返回： 空，状态 `400`

**清空在线剪贴板内容的接口路径 `/clear`**
 * 请求方式： 任意
 * 成功返回： `cleared`，状态 `200`
 * 失败返回： `failed`，状态 `500`

## 变量说明

| 变量名 | 示例 | 必填 | 备注 | 
|--|--|--|--|
| TOKEN | `expectedPath` |❌| 设置主页和接口的安全路径（不添加则仍然使用默认路径） |

配置安全入口路径变量 `TOKEN` 可以保护服务不被滥用。

**假设变量 `TOKEN` 设置为 `expectedPath`**
 * 主页入口路径变更为 `/expectedPath`（默认入口路径为 `/` ）
 * 保存到在线剪贴板的接口路径变更为 `/expectedPath/save`
 * 读取在线剪贴板内容的接口路径变更为 `/expectedPath/read`
 * 清空在线剪贴板内容的接口路径变更为 `/expectedPath/clear`

## 使用技巧

**提供了方便与 WINDOWS 本地剪贴板交互的 .bat 脚本文件模板。修改 `set url=` 为自己的域名，`set token=` 为自己的安全入口路径变量 `TOKEN` 即可（未设置变量 `TOKEN` 则删除 `set token=`的行和 `%url%/%token%` 部分的 `/%token%` ）**
 * 读取本地剪贴板内容，保存到在线剪贴板
```
@echo off
chcp 65001 >nul

set url="https://clipboard.example.com"
set token="expectedPath"

echo.
echo ==========================
echo 保存到在线剪贴板
echo ==========================
echo.
powershell -command "[System.IO.File]::WriteAllText('save.txt', (Get-Clipboard -Raw))"
if not exist "save.txt" (
    exit /b
)
for %%A in ("save.txt") do (
    if %%~zA equ 0 (
        del "save.txt"
        echo.
        echo ==========================
        echo 剪贴板为空！
        echo ==========================
        echo.
        timeout /t 3 >nul
        exit /b
    )
)
for /f "usebackq delims=" %%a in (`curl -s -X POST "%url%/%token%/save" --data-binary "@save.txt"`) do set "resp=%%a"
if /I "%resp%"=="saved" (
    set "msg=已保存到云端！"
) else (
    set "msg=保存失败！"
)
del "save.txt"
echo.
echo ==========================
echo %msg%
echo ==========================
echo.
timeout /t 3 >nul

```
 * 读取在线剪贴板内容，复制到本地剪贴板
```
@echo off
chcp 65001 >nul

set url="https://clipboard.example.com"
set token="expectedPath"

echo.
echo ==========================
echo 读取在线剪贴板
echo ==========================
echo.
curl -s "%url%/%token%/read" > "read.txt"
if not exist "read.txt" (
    exit /b
)
for %%A in ("read.txt") do (
    if %%~zA equ 0 (
        del "read.txt"
        echo.
        echo ==========================
        echo 读取失败或剪贴板为空！
        echo ==========================
        echo.
        timeout /t 3 >nul
        exit /b
    )
)
type "read.txt" | "%SystemRoot%\System32\clip.exe"
del "read.txt"
echo.
echo ==========================
echo 已复制到本地剪贴板！
echo ==========================
echo.
timeout /t 3 >nul

```
 * 清空在线剪贴板内容
```
@echo off
chcp 65001 >nul

set url="https://clipboard.example.com"
set token="expectedPath"

echo.
echo ==========================
echo 清空在线剪贴板
echo ==========================
echo.
for /f "usebackq delims=" %%a in (`curl -s "%url%/%token%/clear"`) do set "resp=%%a"
if /I "%resp%"=="cleared" (
    set "msg=剪贴板已清空！"
) else (
    set "msg=清空失败！"
)
echo.
echo ==========================
echo %msg%
echo ==========================
echo.
timeout /t 3 >nul

```

