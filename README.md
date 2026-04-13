# Hugo 个人网站（GitHub Pages）

这个项目已经用 Hugo 初始化，并配置了 GitHub Pages 自动部署工作流。

## 1. 本地预览

Windows PowerShell:

```powershell
& "C:\Users\langlang\AppData\Local\Microsoft\WinGet\Packages\Hugo.Hugo.Extended_Microsoft.Winget.Source_8wekyb3d8bbwe\hugo.exe" server -D
```

浏览器打开 `http://localhost:1313/`。

## 2. 修改站点信息

- `hugo.toml`：站点标题、菜单、作者等
- `content/_index.md`：首页内容
- `content/about.md`：关于页
- `content/posts/*.md`：文章

建议把 `hugo.toml` 里的以下值改成你自己的：

- `title`
- `params.author`
- `params.description`
- `baseURL`（可选，工作流会自动注入 Pages URL）

## 3. 发布到 GitHub Pages

1. 在 GitHub 新建仓库（例如 `my-site`）。
2. 推送本地代码到远端 `main` 分支。
3. 进入 GitHub 仓库：
   - `Settings` -> `Pages`
   - `Build and deployment` 选择 `GitHub Actions`
4. 等待 `Actions` 里的 `Deploy Hugo site to Pages` 工作流执行完成。
5. 成功后即可通过仓库的 Pages 链接访问网站。

## 4. 可选：自定义域名

如果要绑定域名，可以在 `static/CNAME` 新建文件写入你的域名，例如：

```text
www.example.com
```
