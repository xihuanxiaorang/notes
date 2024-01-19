## 创建项目

使用 Vue3/Vite 版，创建以 typescript 开发的工程，命令：`npx degit dcloudio/uni-preset-vue#vite-ts my-vue3-project`，其中 `my-vue3-project` 为项目目录名称，可以进行更换。参考自 [uni-app官网](https://uniapp.dcloud.net.cn/quickstart-cli.html#%E5%88%9B%E5%BB%BAuni-app)，效果如下所示：<br /> ![](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202312210001592.png)

## 推荐插件安装

VSCode 推荐安装插件：

- [EditorConfig for VS Code](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig)
- [ESLint](https://marketplace.visualstudio.com/items?itemName=dbaeumer.vscode-eslint)
- [ESLint Chinese Rules](https://marketplace.visualstudio.com/items?itemName=maggie.eslint-rules-zh-plugin)
- [Prettier - Code formatter](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
- [TypeScript Vue Plugin (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.vscode-typescript-vue-plugin)
- [Vue Language Features (Volar)](https://marketplace.visualstudio.com/items?itemName=Vue.volar)
- [uni-create-view](https://marketplace.visualstudio.com/items?itemName=mrmaoddxxaa.create-uniapp-view)
- [uni-helper](https://marketplace.visualstudio.com/items?itemName=uni-helper.uni-helper-vscode)
- [uniapp 小程序扩展](https://marketplace.visualstudio.com/items?itemName=evils.uniapp-vscode)

在项目根目录下创建 `.vscode/extensions.json` 文件，内容如下所示：

```json
{
	"recommendations": [
		"editorconfig.editorconfig",
		"dbaeumer.vscode-eslint",
		"maggie.eslint-rules-zh-plugin",
		"esbenp.prettier-vscode",
		"vue.vscode-typescript-vue-plugin",
		"vue.volar",
		"mrmaoddxxaa.create-uniapp-view",
		"uni-helper.uni-helper-vscode",
		"evils.uniapp-vscode"
	]
}
```

这样团队其他小伙伴在拉取代码使用 VSCode 打开之后，在扩展中输入 `@recommended ` 就会推荐安装这些插件。<br />![image-20231222223329275](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202312222233318.png)

## editorconfig

VSCode 安装 [EditorConfig for VS Code](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig) 插件，然后在项目根目录下创建 `.editorconfig` 文件，内容如下所示：

```
# EditorConfig is awesome: https://EditorConfig.org

# top-most EditorConfig file
root = true

# Unix-style newlines with a newline ending every file
[*]
#编码格式，通常都是选 utf-8
charset = utf-8
#缩进风格，可选配置有 tab 和 space
indent_style = space
#缩进大小，可设定为 1-8 的数字
indent_size = 2
#换行符，可选配置有 lf ，cr ，crlf
end_of_line = lf
#去除多余的空格
trim_trailing_whitespace = false
#在尾部插入一行
insert_final_newline = false
```

## eslint

1. 安装：使用 `yarn add eslint -D` 命令安装 Eslint；
2. 初始化：使用 `npx eslint --init` 或者 `npm init @eslint/config` 命令进行初始化，参考自 [Getting Started with ESLint - ESLint - Pluggable JavaScript Linter](https://eslint.org/docs/latest/use/getting-started)，如下所示：<br />![](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202312210008675.png)

   安装完成之后会在项目根目录下生成一个 `.eslintrc.js` 文件。

3. 启用 ESlint & Prettier 进行格式化：VSCode 设置 -> 工作区 -> 打开右上角设置 JSON 文件，此时会在项目根目录下创建一个 `.vscode/settings.json` 文件，添加如下配置：

   ```json
   {
   	"files.autoSave": "afterDelay",
   	"eslint.enable": true,
   	"eslint.run": "onType",
   	"eslint.options": {
   		"extensions": [".js", ".jsx", ".ts", ".tsx", ".vue"]
   	},
   	"editor.codeActionsOnSave": {
   		"source.fixAll.eslint": "explicit"
   	},
   	"editor.formatOnSave": true, // 保存格式化文件
   	"editor.defaultFormatter": "esbenp.prettier-vscode", // 指定 prettier 为所有文件默认格式化器
   	"[vue]": {
   		"editor.defaultFormatter": "esbenp.prettier-vscode"
   	},
   	"[typescript]": {
   		"editor.defaultFormatter": "esbenp.prettier-vscode"
   	},
   	"[jsonc]": {
   		"editor.defaultFormatter": "esbenp.prettier-vscode"
   	},
   	// 配置语言的文件关联
   	"files.associations": {
   		"pages.json": "jsonc", // pages.json 可以写注释
   		"manifest.json": "jsonc" // manifest.json 可以写注释
   	}
   }
   ```

4. 对于 Vue 项目需要安装 `yarn add -D eslint-plugin-vue` 插件，参考自 [工具链 | Vue.js](https://cn.vuejs.org/guide/scaling-up/tooling.html#linting)，如下所示：<br /> ![image-20231220164312380](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202312210013100.png)

   安装完插件之后，修改 `.eslintrc.js ` 文件中的配置项，参考自 [User Guide | eslint-plugin-vue](https://eslint.vuejs.org/user-guide/)，如下所示：

   ```js
   module.exports = {
     env: {
       browser: true,
       es2021: true,
       node: true,
     },
     extends: [
       "plugin:vue/base",
       "plugin:vue/vue3-essential",
       "plugin:vue/vue3-strongly-recommended",
       "plugin:vue/vue3-recommended",
       "standard-with-typescript",
     ],
     overrides: [
       {
         env: {
           node: true,
         },
         files: [".eslintrc.{js,cjs}"],
         parserOptions: {
           sourceType: "script",
         },
       },
     ],
     parserOptions: {
       ecmaVersion: "latest",
       sourceType: "module",
     },
     plugins: ["vue"],
     rules: {},
   };
   ```

   此时打开 `App.vue` 文件，可以看到第一行就提示错误，间接说明 ESlint 已经生效，如下所示：<br /> ![](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202312211617394.png)

   那么该如何解决该错误呢？其实上图画横线的地方已经给出答案，咱们需要在 `.eslintrc.js` 文件的 `parserOptions` 配置项中添加 `extraFileExtensions: ['.vue']`，参考自 [Troubleshooting & FAQs | typescript-eslint](https://typescript-eslint.io/linting/troubleshooting/#i-use-a-framework-like-vue-that-requires-custom-file-extensions-and-i-get-errors-like-you-should-add-parseroptionsextrafileextensions-to-your-config)，如下所示：

   ```js
   module.exports = {
     env: {
       browser: true,
       es2021: true,
       node: true,
     },
     extends: [
       "plugin:vue/base",
       "plugin:vue/vue3-essential",
       "plugin:vue/vue3-strongly-recommended",
       "plugin:vue/vue3-recommended",
       "standard-with-typescript",
     ],
     overrides: [
       {
         env: {
           node: true,
         },
         files: [".eslintrc.{js,cjs}"],
         parserOptions: {
           sourceType: "script",
         },
       },
     ],
     parserOptions: {
       ecmaVersion: "latest",
       sourceType: "module",
       extraFileExtensions: [".vue"],
     },
     plugins: ["vue"],
     rules: {},
   };
   ```

   发现依旧报错，不过是另外的错误，如下所示：<br />![image-20231221175128236](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202312211751294.png)

   提示咱们更多详细信息可以参考自 [Troubleshooting & FAQs | typescript-eslint](https://typescript-eslint.io/linting/troubleshooting/#i-get-errors-telling-me-eslint-was-configured-to-run--however-that-tsconfig-does-not--none-of-those-tsconfigs-include-this-file) <br />![image-20231221164048850](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202312211640905.png)

   需要咱们在项目根目录下创建一个 `tsconfig.eslint.json` 文件，该文件内容如下所示：

   ```json
   {
       "compilerOptions": {
           "types": [
               "@types/node"
           ],
           "noEmit": true,
           "allowJs": true
       },
       "extends": "./tsconfig.json",
       "include": [
           "src/**/*.ts",
           "src/**/*.vue",
           ".eslintrc.js",
           "vite.config.ts"
       ]
   }
   ```

   然后在 `.eslintrc.js` 文件的 `parserOptions` 配置项中添加 `project: "./tsconfig.eslint.json"`。添加完之后，发现出现大量红色波浪线，如下所示：<br />![image-20231221195623206](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202312211956272.png)

   不用慌，正常现象，咱们先往后走，然后再回过头来看一下！再次查看 `App.vue` 文件，发现依然报错，如下所示：<br />![](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202312212047440.png)

   查阅资料，发现需要安装两个插件 `yarn add -D @typescript-eslint/parser vue-eslint-parser`，参考自 [Troubleshooting & FAQs | typescript-eslint](https://typescript-eslint.io/linting/troubleshooting/#i-am-running-into-errors-when-parsing-typescript-in-my-vue-files) & [User Guide | eslint-plugin-vue](https://eslint.vuejs.org/user-guide/#how-to-use-a-custom-parser)，如下所示：<br /> ![image-20231221211337316](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202312212113357.png)

   打开 `src/pages/index/index.vue` 文件，发现报错，错误如下所示：<br />![image-20231221221229003](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202312212212039.png)

   那么这个错误该如何解决呢？在 `.eslintrc.js` 文件的 `rules` 规则选项中添加 `'vue/multi-word-component-names': 'off',` 选项即可！

   这样一个个排查错误太麻烦了！咱们在 `package.json` 文件的 `scripts` 选项中添加 `"lint:eslint": "eslint \"src/**/*.{js,vue,ts}\" --fix",`，然后执行 `npm run lint:eslint` 命令，自动修复完成之后发现还存在如下错误：<br />![image-20231223142114666](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202312231421733.png)

   咱们按照上面一样的办法，在 `.eslintrc.js` 文件的 `rules` 规则选项中添加 `'@typescript-eslint/triple-slash-reference': 'off',` 和 `'@typescript-eslint/explicit-function-return-type': 'off',` 选项禁用这两个规则的检测。

5. 创建 `.eslintignore` 文件用于排除某些文件的 eslint 检测，根据需要进行配置，如下所示：

   ```
   shims-uni.d.ts
   vite.config.ts
   ```

## prettier

为了兼容 ESlint，除了安装 Prettier 本身之外，还需要安装另外两个插件，命令：`yarn add -D prettier eslint-config-prettier eslint-plugin-prettier`，创建 `.prettierrc.js` 文件，内容如下所示：

```js
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

在 `.eslintrc.js` 的 `extends` 选项中添加 `"plugin:prettier/recommended",`，重新打开 VSCode，使用 `Ctrl + S` 保存文件时，可以发现 `.eslintrc.js` 文件中的红色波浪线全部消失不见！

为了避免手动一个个对文件进行格式化，在 `package.json` 文件的 `scripts` 选项中添加 `"prettier-format": "prettier --config .prettierrc.js \"src/**/*.{vue,js,ts}\" --write",` 然后执行 `npm run prettier-format` 命令进行自动格式化文件。

如果有需要的话，可以在项目根目录下创建 `.prettierignore` 文件忽略某些文件的格式化。

打开 `tsconfig.json` 文件，发现错误，错误如下所示：<br />![image-20231221222055435](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202312212220470.png)

那么这个错误该如何解决呢？在当前文件的 `compilerOptions` 选项中添加 `"ignoreDeprecations": "5.0"` 选项即可！

## husky

> 文档地址：[🐶 husky](https://typicode.github.io/husky/)

如果仅有 eslint 和 prettier，那么咱们需要在代码提交之前手动执行 prettier 和 eslint 对代码进行格式化以及代码质量和格式检查，但是咱们希望在提交代码时自动执行 eslint 对代码进行检查，那么咱们可以使用 git 的 hook 功能，为 git 命令创建咱们所需要的钩子，在这里咱们使用 husky 工具来创建、管理代码仓库中的所有 git hooks。

通过 husky 工具来为咱们创建所需要的 git hook，首先需要执行 `yarn add husky -D` 命令安装 husky，然后执行 `npx husky install` 启用 git hook。此外，咱们需要在 `package.json` 中新增一个 prepare 脚本：`"prepare": "husky install"`，使得团队中其他小伙伴在克隆该项目并安装依赖时会自动通过 husky 启用 git hook。

咱们需要的第一个 git hook 是在提交 commit 之前执行咱们的 eslint 工具对代码进行质量和格式检查，也就是在提交 commit 之前执行 `package.json` 中的 `lint` 脚本（其中 lint 脚本为 `"lint": "eslint ./ --ext .ts,.vue,.js --max-warnings=0",`）。咱们通过 husky 命令来创建 pre-commit 这个 hook：`npx husky add .husky/pre-commit "npm run lint"` 。此时可以看到在 `.husky` 文件夹下已经创建 `pre-commit` 文件，内容如下所示：

```
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npm run lint
```

此时，将 `App.vue` 文件随便弄乱，使其不符合 eslint 规范，然后将工作区的代码保存到暂存区之后使用 `git commit` 命令进行提交，从终端中可以看到，确实执行了 `package.json` 中的 lint 脚本，然后 eslint 输出了错误信息并且中断了 git commit 过程，这非常好，符合咱们的预期！如下所示：<br /> ![image-20231223164744111](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202312231647173.png)

## lint-staged

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
    "*.{vue,ts,js}": ["eslint --fix"]
  },
}
```

此外，咱们还需要手动更改一下 husky 为咱们创建的 pre-commit 这个 git hook，将其变更为执行 lint-staged 命令，如下所示：

```
#!/usr/bin/env sh
. "$(dirname -- "$0")/_/husky.sh"

npm run lint-staged
```

效果如下所示：<br />![image-20231223195814165](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202312231958230.png)

## commitlint

[commitlint - Lint commit messages](https://commitlint.js.org/#/) 结合 husky 可以在 git commit 时校验 commit 信息是否符合规范。

使用 `yarn add -D @commitlint/config-conventional @commitlint/cli` 命令安装 commitlint，然后在项目根目录下创建 `.commitlintrc.json` 文件，内容如下所示：

```json
{
  "extends": ["@commitlint/config-conventional"]
}
```

之后使用 husky 添加 commit-msg 的 git hook, 执行 `npx husky add .husky/commit-msg 'npx --no -- commitlint --edit ${1}'` 命令。它的作用是在咱们提交 commit 或者修改 commit message 时执行相关校验，如此依赖，咱们就可以保证咱们的项目拥有一个统一的符合规范的 commit message。

测试效果如下所示：<br />![image-20231223204220168](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202312232042223.png)

从上图可以看到，这个 commit message 不符合咱们设置的 commitlint 规则，所以 commit-msg 这个 git hook 报错并且中断了 commit 过程。咱们接着使用一个符合 commitlint 规则的 commit message 来看看效果，可以看到没有报错没有被中断，满足咱们的预期，这意味着 commitlint 配置成功！如下所示：<br />![image-20231223205010171](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202312232050221.png)

## commitizen

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

使用 `git add .` 将所有的变更文件添加到暂存区，然后再在命令行通过 `npm run cz` 命令执行刚刚添加的脚本，可以看到终端中有了对应的步骤和信息提示，非常好！一切都在咱们的预料当中，满足了咱们的诉求。<br />![image-20231223211729391](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202312232117454.png)

## typescript

为了优化性能，Volar 提供了一个叫做 “Takeover 模式” 的功能。在这个模式下，Volar 能够使用一个 TS 语言服务实例同时为 Vue 和 TS 文件提供支持。

要开启 Takeover 模式，你需要执行以下步骤来**在你的项目的工作空间中**禁用 VSCode 的内置 TS 语言服务：

1. 在当前项目的工作空间下，用 `Ctrl + Shift + P` (macOS：`Cmd + Shift + P`) 唤起命令面板。
2. 输入 `built`，然后选择“Extensions：Show Built-in Extensions”。
3. 在插件搜索框内输入 `typescript` (不要删除 `@builtin` 前缀)。
4. 点击“TypeScript and JavaScript Language Features”右下角的小齿轮，然后选择“Disable (Workspace)”。
5. 重新加载工作空间。Takeover 模式将会在你打开一个 Vue 或者 TS 文件时自动启用。
    ![](https://cdn.jsdelivr.net/gh/xihuanxiaorang/img/202312240032400.png)

完善 typescript 类型校验，首先需要使用 `yarn add -D miniprogram-api-typings @uni-helper/uni-app-types` 命令安装对应的类型声明文件，然后在 `tsconfig.json` 文件中添加如下配置信息：

```json
{
    // ... 省略 ...
    "compilerOptions": {
		// ... 省略 ...
        
		// 类型声明文件
		"types": [
			"@dcloudio/types", // uni-app API 类型
			"miniprogram-api-typings", // 原生微信小程序类型
			"@uni-helper/uni-app-types" // uni-app 组件类型
		],
	},
    // vue 编译器类型，校验标签类型
	"vueCompilerOptions": {
		// 原配置 `experimentalRuntimeMode` 现调整为 `nativeTags`
		"nativeTags": ["block", "component", "template", "slot"]
	},
}
```

## standard-version

> [GitHub - conventional-changelog/standard-version: :trophy: Automate versioning and CHANGELOG generation, with semver.org and conventionalcommits.org](https://github.com/conventional-changelog/standard-version)

首先使用 `yarn add -D standard-version` 安装 standard-version，然后在 `package.json` 中添加新的脚本：`"release": "standard-version"`。安装完成之后，只需执行 `npm run release` 命令就会自动生成 changelog，自动打 tag，自动 commit，而且提交信息是标准的 commitizen 规范。你只需要 push 即可。

> [!attention]
> **CHANGELOG.md 是追加写入内容的，如果你之前没有对应的内容或删了之前的内容，会导致生成的内容较少，或者不完整。**

## uni-ui 组件库

1. 使用 `yarn add sass -D` 命令安装 sass；
2. 使用 `yarn add @dcloudio/uni-ui` 命令安装 uni-ui 组件库；
3. 配置自动导入组件：打开项目根目录下的 `pages.json` 并添加 `easycom` 节点，如下所示：

   ```js
   {
   	// 组件自动引入规则
   	"easycom": {
   		// 是否开启自动扫描
   		"autoscan": true,
   		// 以正则方式自定义组件匹配规则
   		"custom": {
   			// uni-ui 规则如下配置
   			"^uni-(.*)": "@dcloudio/uni-ui/lib/uni-$1/uni-$1.vue"
   		}
   	},
   	// 其他内容
   	pages:[
   		// ...
   	]
   }
   ```

4. 使用 `yarn add -D @uni-helper/uni-ui-types` 命令安装类型声明文件，并在 `tsconfig.json` 文件中进行配置，如下所示：

   ```json
   {
     "compilerOptions": {
       // ...
       "types": [
         "@dcloudio/types", // uni-app API 类型
         "miniprogram-api-typings", // 原生微信小程序类型
         "@uni-helper/uni-app-types", // uni-app 组件类型
         "@uni-helper/uni-ui-types" // uni-ui 组件类型  
       ]
     },
     // vue 编译器类型，校验标签类型
     "vueCompilerOptions": {
       "nativeTags": ["block", "component", "template", "slot"]
     }
   }
   ```

## pinia 状态管理库

1. 使用 `yarn add pinia@2.0.36 pinia-plugin-persistedstate` 命令安装 [Pinia](https://pinia.vuejs.org/zh/) & [pinia-plugin-persistedstate](https://prazdevs.github.io/pinia-plugin-persistedstate/zh/)。
2. 在 src 目录下创建 store 文件夹并且在该文件夹下创建 `index.ts` 文件，在该文件中，创建一个 pinia 实例 (根 store)，然后将 pinia-plugin-persistedstate 插件应用到 pinia 实例上，内容如下所示：

   ```typescript
   import { createPinia } from 'pinia';
   import piniaPluginPersistedstate from 'pinia-plugin-persistedstate';
   
   // 创建 pinia 实例
   const pinia = createPinia();
   // 使用持久化存储插件
   pinia.use(piniaPluginPersistedstate);
   
   // 默认导出，给 main.ts 使用
   export default pinia;
   ```

3. 将 pinia 实例传递给应用，具体实现如下所示：

   ```typescript
   import { createSSRApp } from 'vue';
   import App from './App.vue';
   import pinia from './store';
   export function createApp() {
   	const app = createSSRApp(App);
   	app.use(pinia);
   	return {
   		app,
   	};
   }
   ```

4. 用法，如下所示：

   ```typescript
   import { defineStore } from 'pinia';
   import { ref } from 'vue';
   
   export const useMemberStore = defineStore(
   	'member',
   	() => {
   		// 会员信息
   		const profile = ref<any>();
   
   		// 保存会员信息，登录时使用
   		const setProfile = (val: any) => {
   			profile.value = val;
   		};
   
   		// 清理会员信息，退出时使用
   		const clearProfile = () => {
   			profile.value = undefined;
   		};
   
   		return {
   			profile,
   			setProfile,
   			clearProfile,
   		};
   	},
   	{
   		// 小程序端配置
   		persist: {
   			storage: {
   				getItem(key) {
   					return uni.getStorageSync(key);
   				},
   				setItem(key, value) {
   					uni.setStorageSync(key, value);
   				},
   			},
   		},
   	}
   );
   ```

## 参考资料

- 视频：
	- 【uniapp+vue3+ts 项目搭建】 https://www.bilibili.com/video/BV1Uu4y1P7vw/?share_source=copy_web&vd_source=84272a2d7f72158b38778819be5bc6ad
	- 【git commit 最佳实践，commitizen + husky + commitlint 规范化校验】 https://www.bilibili.com/video/BV193411C7XE/?share_source=copy_web&vd_source=84272a2d7f72158b38778819be5bc6ad
	- 【nodejs 项目工程化 eslint prettier husky lint-staged commitlint commitizen】 https://www.bilibili.com/video/BV1a8411i77L/?share_source=copy_web&vd_source=84272a2d7f72158b38778819be5bc6ad
- 文档：
	- [VUE3+TS项目内集成ESLint、Prettier、Stylelint、husky 、lint-staged以及Commitizen - 掘金](https://juejin.cn/post/7205094404415488058)
	- [从零搭建属于你自己的前端规范+自动化部署 - 掘金](https://juejin.cn/post/7207617774633107512)
	- [GitHub - dev-zuo/commitizen-practice-demo: Vue 项目 commitizen + husky + commitlint，git commit 提交信息规范校验 demo，conventional commits 实践](https://github.com/dev-zuo/commitizen-practice-demo)
