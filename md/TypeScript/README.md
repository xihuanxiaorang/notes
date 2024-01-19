## 环境搭建

### 创建项目

为了方便学习 TypeScript，咱们以最简单的方式来搭建一个 TypeScript 学习环境。使用 `yarn create vite` [命令](https://cn.vitejs.dev/guide/#scaffolding-your-first-vite-project) 创建项目，即使用 vite 构建工具。项目名称：`learn-typescript`，预设模板：`vanilla-ts`。如下所示：<br /> ![](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202401121249434.png)

使用 vscode 打开项目，删除不必要的文件，如 `counter.ts`、`style.css` 、`public/vite.svg `，将 `src/typescript.svg` 移动到 `public` 目录下。稍微改动 ` index.html ` 至如下所示：

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/typescript.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>TypeScript学习</title>
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

删除 `main.ts` 文件中的所有内容，然后编写如下代码：

```typescript
let a = '123';
console.log(a);
```

最后使用 `yarn` 命令下载所有依赖，`yarn dev` 命令启动项目，打开浏览器，可以在控制台中看到 `123` 输出。

> [!tip]
> 以后每学习一节知识点，只需将 `main.ts` 文件**拷贝**一份然后**改为其他名称**即可，如 `1.TypeScript基本类型.ts`，这样下次学习在其他小节的知识点时依旧可以在 `main.ts` 中进行操作。

### 推荐插件安装

VSCode 推荐安装插件：

- [TypeScript Importer](https://marketplace.visualstudio.com/items?itemName=pmneo.tsimporter)
- [Move TS - Move TypeScript files and update relative imports](https://marketplace.visualstudio.com/items?itemName=stringham.move-ts)
- [Error Lens](https://marketplace.visualstudio.com/items?itemName=usernamehw.errorlens)
- [EditorConfig for VS Code](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig)
- [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
- [ESLint Chinese Rules](https://marketplace.visualstudio.com/items?itemName=maggie.eslint-rules-zh-plugin)
- [Prettier - Code formatter](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)

在项目根目录下创建 `.vscode/extensions.json` 文件，内容如下所示：

```json
{
  "recommendations": [
    "pmneo.tsimporter",
    "stringham.move-ts",
    "usernamehw.errorlens",
    "editorconfig.editorconfig",
    "esbenp.prettier-vscode",
    "dbaeumer.vscode-eslint",
    "maggie.eslint-rules-zh-plugin"
  ]
}
```

这样团队其他小伙伴在拉取代码使用 VSCode 打开之后，在扩展中输入 `@recommended ` 就会推荐安装这些插件。<br />![image-20240116172229977](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202401161722007.png)

### editorconfig

VSCode 安装 [EditorConfig for VS Code](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig) 插件，然后在项目根目录下创建 `.editorconfig` 文件，内容如下所示：

```
# EditorConfig is awesome: https://EditorConfig.org

# top-most EditorConfig file
root = true

# Unix-style newlines with a newline ending every file
[*]
#编码格式，通常都是选 utf-8
charset = utf-8
#缩进风格，可选配置有 tab 和 space
indent_style = space
#缩进大小，可设定为 1-8 的数字
indent_size = 2
#换行符，可选配置有 lf ，cr ，crlf
end_of_line = lf
#去除多余的空格
trim_trailing_whitespace = false
#在尾部插入一行
insert_final_newline = false
```

### eslint

1. 安装：使用 `yarn add eslint -D` 命令安装 Eslint；
2. 初始化：使用 `npx eslint --init` 或者 `npm init @eslint/config` 命令进行初始化，参考自 [Getting Started with ESLint - ESLint - Pluggable JavaScript Linter](https://eslint.org/docs/latest/use/getting-started)，如下所示：<br />![image-20240116171158831](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202401161711935.png)

   安装完成之后会在项目根目录下生成一个 `.eslintrc.cjs` 文件。此时可以看到 `main.ts` 文件已经报错，如下所示：提示应该使用 `const` 来代替 `let`，证明 `eslint` 已经生效！<br />![image-20240116171533722](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202401161715755.png)

3. 创建 `.eslintignore` 文件用于排除某些文件的 eslint 检测，根据需要进行配置，如下所示：

   ```
   node_modules
   dist
   *.md
   .vscode
   .idea
   public
   *.js
   *.cjs
   ```

### prettier

为了兼容 ESlint，除了安装 Prettier 本身之外，还需要安装另外两个插件，命令：`yarn add -D prettier eslint-config-prettier eslint-plugin-prettier`，创建 `.prettierrc.cjs` 文件，内容如下所示：

```javascript
// 参考文档：https://www.prettier.cn/docs/options.html
module.exports = {
  // 换行长度
  printWidth: 150,
  // 指定每个缩进级别的空格数
  tabWidth: 2,
  // 使用制表符而不是空格缩进行
  useTabs: true,
  // 在语句末尾是否需要分号
  semi: true,
  // 是否使用单引号
  singleQuote: true,
  // 更改引用对象属性的时间可选值"<as-needed|consistent|preserve>"
  quoteProps: "as-needed",
  // 多行时尽可能打印尾随逗号，可选值"<none|es5|all>"，默认none
  trailingComma: "es5",
  // 在对象文字中的括号之间打印空格
  bracketSpacing: true,
  // 在单独的箭头函数参数周围包括括号 always要，avoid不要
  arrowParens: "always",
};
```

配置保存时自动格式化，在项目根目录下创建 `.vscode/settings.json` 文件，内容如下所示：除保存时自动格式化之外，还有关于 TypeScript 的相关提示配置，提高 TypeScript 开发体验。

```json
{
  "editor.formatOnSave": true,
  "typescript.inlayHints.enumMemberValues.enabled": true,
  "typescript.inlayHints.functionLikeReturnTypes.enabled": true,
  "typescript.inlayHints.parameterNames.enabled": "all",
  "typescript.inlayHints.parameterTypes.enabled": true,
  "typescript.inlayHints.propertyDeclarationTypes.enabled": true,
  "typescript.inlayHints.variableTypes.enabled": true,
  "typescript.preferences.preferTypeOnlyAutoImports": true
}
```

在 `.eslintrc.js` 的 `extends` 选项中添加 `"plugin:prettier/recommended",`，然后稍微弄乱 `main.ts` 文件的代码格式，可以发现 prettier 已经报错，如下所示：<br />![image-20240116173600902](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202401161736941.png)

如果此时咱们再稍微改动一下代码，使用 `Ctrl + S` 保存文件之后可以惊奇地发现文件中的红色波浪线全部消失不见，并且代码格式已经帮咱们修复！

同 eslint 一样，可以创建 `.prettierignore` 文件用于排除某些文件的 prettier 检测，根据需要进行配置，如下所示：

```
node_modules
dist
```

### husky

> 文档地址：[🐶 husky](https://typicode.github.io/husky/)

如果仅有 eslint 和 prettier，那么咱们需要在代码提交之前手动执行 prettier 和 eslint 对代码进行格式化以及代码质量和格式检查，但是咱们希望在提交代码时自动执行 eslint 对代码进行检查，那么咱们可以使用 git 的 hook 功能，为 git 命令创建咱们所需要的钩子，在这里咱们使用 husky 工具来创建、管理代码仓库中的所有 git hooks。

通过 husky 工具来为咱们创建所需要的 git hook，首先需要执行 `yarn add husky -D` 命令安装 husky，然后执行 `npx husky install` 启用 git hook。此外，咱们需要在 `package.json` 中新增一个 prepare 脚本：`"prepare": "husky install"`，使得团队中其他小伙伴在克隆该项目并安装依赖时会自动通过 husky 启用 git hook。

咱们需要的第一个 git hook 是在提交 commit 之前执行咱们的 eslint 工具对代码进行质量和格式检查，也就是在提交 commit 之前执行 `package.json` 中的 `lint` 脚本（其中 lint 脚本为 `"lint": "eslint ./ --ext .ts,.js --max-warnings=0",`）。咱们通过 husky 命令来创建 pre-commit 这个 hook：`npx husky add .husky/pre-commit "npm run lint"` 。此时可以看到在 `.husky` 文件夹下已经创建 `pre-commit` 文件，内容如下所示：

```
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npm run lint
```

此时，将 `main.ts` 文件随便弄乱，使其不符合 eslint 规范，如下所示：<br />

```typescript
let a = '123';
console.log(a);
const obj = { message: 'hello' };
```

然后将工作区的代码保存到暂存区之后使用 `git commit` 命令进行提交，从终端中可以看到，确实执行了 `package.json` 中的 lint 脚本，然后 eslint 输出了错误信息并且中断了 git commit 过程，这非常好，符合咱们的预期！如下所示：<br />![image-20240116174750795](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202401161747841.png)

### lint-staged

> 文档地址：[GitHub - lint-staged/lint-staged: 🚫💩 — Run linters on git staged files](https://github.com/lint-staged/lint-staged)

随着代码存储库的代码量增多，如果在每一次提交代码时，咱们都对存储库内的全量代码执行 prettier 和 eslint 命令，则必然会性能吃紧，所以，咱们希望提交代码时只对当前发生了代码变更的文件执行 prettier 和 eslint 命令，同时略过咱们所忽略的文件，那么咱们就需要用上 lint-staged 工具。

首先执行 `yarn add -D lint-staged` 命令安装 lint-staged，安装完成之后, 配置 `package.json` 文件，如下所示：

```json
{
  "script": {
    // ... 省略 ...
    "lint-staged": "lint-staged"
  },
  "lint-staged": {
    "*.{ts,js}": ["eslint --fix"]
  },
}
```

此外，咱们还需要手动更改一下 husky 为咱们创建的 pre-commit 这个 git hook，将其变更为执行 lint-staged 命令，如下所示：

```
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npm run lint-staged
```

效果如下所示：<br />![image-20240116175230545](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202401161752589.png)

从上图可以看到，在提交时会执行 `eslint --fix` 进行修复，不过对于没有使用的变量这个错误 eslint 无法自动修复，需要人为的手动删除该变量。

### commitlint

[commitlint - Lint commit messages](https://commitlint.js.org/#/) 结合 husky 可以在 git commit 时校验 commit 信息是否符合规范。

使用 `yarn add -D @commitlint/config-conventional @commitlint/cli` 命令安装 commitlint，然后在项目根目录下创建 `.commitlintrc.json` 文件，内容如下所示：

```json
{
  "extends": ["@commitlint/config-conventional"]
}
```

之后使用 husky 添加 commit-msg 的 git hook, 执行 `npx husky add .husky/commit-msg 'npx --no -- commitlint --edit ${1}'` 命令。它的作用是在咱们提交 commit 或者修改 commit message 时执行相关校验，如此依赖，咱们就可以保证咱们的项目拥有一个统一的符合规范的 commit message。

测试效果如下所示：<br />![image-20240116175853465](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202401161758510.png)

从上图可以看到，这个 commit message 不符合咱们设置的 commitlint 规则，所以 commit-msg 这个 git hook 报错并且中断了 commit 过程。咱们接着使用一个符合 commitlint 规则的 commit message 来看看效果，可以看到没有报错没有被中断，满足咱们的预期，这意味着 commitlint 配置成功！如下所示：<br />![image-20240116180159863](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202401161801914.png)

### commitizen

> 中文文档地址：[约定式提交](https://www.conventionalcommits.org/zh-hans/v1.0.0/)

引入 [commitizen](https://github.com/commitizen/cz-cli) 来帮助咱们便捷式地创建符合 commitlint 规范的 commit message，首先使用 `yarn add -D commitizen cz-conventional-changelog` 命令安装 commitizen，然后在项目根目录下创建 `.czrc` 文件，内容如下所示：

```
{
  "path": "cz-conventional-changelog"
}
```

其中 cz-conventional-changelog 是 commitizen 的 conventional-changelog 适配器，使用该适配器，commitizen 将以 AngularJS 的 commit message 规范逐步引导咱们完成 commit message 的创建，然后在 `package.json` 文件中新增脚本，如下所示：

```json
{
  "script": {
    // ... 省略 ...
    "cz": "cz"
  },
}
```

使用 `git add .` 将所有的变更文件添加到暂存区，然后再在命令行通过 `npm run cz` 命令执行刚刚添加的脚本，可以看到终端中有了对应的步骤和信息提示，非常好！一切都在咱们的预料当中，满足了咱们的诉求。<br /> ![image-20240116180611582](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202401161806626.png)

## 参考资料🎁

- 文档
  - [TypeScript中文网 · TypeScript——JavaScript的超集](https://www.tslang.cn/docs/home.html)
  - [TypeScript 阮一峰 | 阮一峰 TypeScript 教程](https://typescript.p6p.net/)
  - [TypeScript 入门教程 - 林不渡 - 掘金小册](https://juejin.cn/book/7288482920602271802?enter_from=search_result&utm_source=search)
  - [TypeScript 全面进阶指南 - 林不渡 - 掘金小册](https://juejin.cn/book/7086408430491172901)
  - [TypeScript 类型体操通关秘籍 - zxg\_神说要有光 - 掘金小册](https://juejin.cn/book/7047524421182947366?enter_from=search_result&utm_source=search)
  - [TypeScript进阶手册 - 《📚 技术修行》 - 极客文档](https://geekdaxue.co/read/nardo@goi5e0/zGt03cVcpL5c-djS)
- 视频
  - [TypeScript-珠峰](https://www.bilibili.com/video/BV1wV4y1v73v/?share_source=copy_web&vd_source=84272a2d7f72158b38778819be5bc6ad)
  - [typescript 从入门到放弃](https://www.bilibili.com/video/BV1Fw411w72p/?share_source=copy_web&vd_source=84272a2d7f72158b38778819be5bc6ad)
  - [TypeScript入门实战笔记-拉勾](https://www.bilibili.com/video/BV1K94y1k7PV/?share_source=copy_web&vd_source=84272a2d7f72158b38778819be5bc6ad)
- 工具
  - [TypeScript: 演练场 - 一个用于 TypeScript 和 JavaScript 的在线编辑器](https://www.typescriptlang.org/zh/play)
