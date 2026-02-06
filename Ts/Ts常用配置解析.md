## TypeScript 配置详解（基于最新版本）

本指南以 TypeScript 最新版本（特别是其现代的模块系统策略）为背景，详解常用配置项。核心配置可分为 **基础与输出**、**模块解析**、**项目结构**及**类型检查**四大类。

### 一、基础与输出配置

此类配置定义了编译的代码目标和输出形式。

#### 1. `target`
*   **特点**：指定编译生成的 JavaScript 代码遵循的 ECMAScript 语言标准版本（如 ES2020, ES2022）。这决定了代码中可以使用哪些语法（如箭头函数、可选链），并直接影响最终代码在目标运行环境中的兼容性。
*   **必要条件**：无。通常与 `lib` 配置配合使用，以引入对应版本的标准库类型定义。
*   **示例**：
    ```json
    {
      "compilerOptions": {
        "target": "ES2022"
      }
    }
    ```

#### 2. `module`
*   **特点**：此配置至关重要，它决定了代码的**模块格式**（如 CommonJS, ES2015）和**模块解析策略**的行为基础。它不仅是输出格式，还影响着 TypeScript 如何理解文件是模块（有独立作用域）还是脚本（全局作用域）。
*   **必要条件**：必须与目标运行环境（即“宿主”，Host）匹配。
    *   对于 **Node.js（v12+）**，应使用 `nodenext` 或 `node16`，以准确模拟其 ESM/CJS 互操作规则。
    *   对于使用 **Webpack、Vite 等打包工具**的项目，通常使用 `esnext` 并配合 `moduleResolution: bundler`。
    *   对于 **浏览器原生 ES 模块**，目前也推荐使用 `nodenext` 来获得最严格的解析检查。
*   **示例**（Node.js 项目）：
    ```json
    {
      "compilerOptions": {
        "module": "nodenext"
      }
    }
    ```

#### 3. `lib`
*   **特点**：指定编译环境（宿主环境）可用的 API 类型定义，如 `DOM`, `ES2022`。它确保 TypeScript 能正确识别 `document.querySelector` 或 `Promise` 等全局对象和方法的类型。
*   **必要条件**：无，但必须根据你的代码运行环境来设置。例如，浏览器项目需要包含 `DOM`，纯 Node.js 项目则不需要。
*   **示例**：
    ```json
    {
      "compilerOptions": {
        "lib": ["ES2022", "DOM"]
      }
    }
    ```

#### 4. `outDir`
*   **特点**：指定编译后所有输出文件（`.js`, `.d.ts` 等）的根目录。与 `rootDir` 配合使用，可以保持源码的目录结构。
*   **必要条件**：通常需要与 `rootDir` 一起设置，以清晰地界定输入和输出的结构。
*   **示例**：
    ```json
    {
      "compilerOptions": {
        "rootDir": "./src",
        "outDir": "./dist"
      }
    }
    ```

#### 5. `rootDir`
*   **特点**：指定 TypeScript 源文件的公共根目录。编译器会根据此目录计算输出文件在 `outDir` 中的相对路径。在**项目引用**（`composite`）中，如果未明确设置，其默认值会变为包含 `tsconfig.json` 的目录。
*   **必要条件**：当使用 `outDir` 时强烈建议设置，以确保输出结构的可预测性。
*   **示例**：见 `outDir` 示例。

### 二、模块解析配置

此类配置指导 TypeScript 如何查找并理解导入的模块。

#### 1. `moduleResolution`
*   **特点**：控制编译器遵循何种策略来查找 `import` 或 `require` 语句所指向的模块文件。这是理解项目如何“引入”外部代码的关键。不同的 `module` 设置通常会隐式指定不同的 `moduleResolution`。
*   **必要条件**：与 `module` 设置紧密耦合。
    *   `module: nodenext` 隐含 `moduleResolution: nodenext`，用于 Node.js 的现代模块系统。
    *   `module: esnext` 配合 `moduleResolution: bundler`，用于打包工具环境。
*   **示例**（使用打包工具）：
    ```json
    {
      "compilerOptions": {
        "module": "esnext",
        "moduleResolution": "bundler"
      }
    }
    ```

#### 2. `baseUrl`
*   **特点**：设置解析**非相对模块导入**（如 `import "jquery"`）的基准目录。所有非相对导入都将被视为相对于此路径。
*   **必要条件**：通常与 `paths` 配置一起使用。如果单独设置 `paths`，则必须设置 `baseUrl`。
*   **示例**：
    ```json
    {
      "compilerOptions": {
        "baseUrl": "./src",
        "paths": {
          "@utils/*": ["utils/*"]
        }
      }
    }
    ```

#### 3. `paths`
*   **特点**：提供模块名到基于 `baseUrl` 的实际路径的映射。常用于为项目内部模块创建别名（例如 `@components`），实现更清晰和可维护的导入路径。
*   **必要条件**：必须同时设置 `baseUrl`。
*   **示例**：见 `baseUrl` 示例。

#### 4. `types` / `typeRoots`
*   **特点**：
    *   `typeRoots`：指定查找全局类型声明（`.d.ts` 文件）的目录列表，默认包含 `node_modules/@types`。
    *   `types`：更精确地指定需要包含的全局声明包名。如果指定，则只会包含列出的包，而不自动包含所有 `@types` 下的包。
*   **必要条件**：无。
*   **示例**（仅包含特定类型包）：
    ```json
    {
      "compilerOptions": {
        "types": ["node", "react"]
      }
    }
    ```

#### 5. `esModuleInterop` 与 `allowSyntheticDefaultImports`
*   **特点**：这两个选项主要处理 CommonJS 模块（通常使用 `module.exports` 导出）与 ES 模块（使用 `export default` 导出）之间的互操作性。启用 `esModuleInterop` 后，TypeScript 会生成辅助代码，使你可以使用 ES 模块的默认导入语法（`import React from 'react'`）来导入 CommonJS 包。`allowSyntheticDefaultImports` 是一个更宽松的类型检查选项，允许此类语法，但不改变输出代码。
*   **必要条件**：在现代配置中，当 `module` 设置为 `nodenext` 或 `node16` 时，`esModuleInterop` 会自动启用。对于打包工具项目，也常显式设置为 `true`。
*   **示例**：
    ```json
    {
      "compilerOptions": {
        "esModuleInterop": true
      }
    }
    ```

#### 6. `verbatimModuleSyntax`
*   **特点**：这是 TypeScript 5.0+ 引入的推荐选项。它强制 `import/export` 语句在输出 JavaScript 时**完全按原样**保留，禁止编译器为了兼容性而重写它们。这能避免因编译器重写导入而导致的、在不同用户环境下的歧义问题，是编写可移植库的理想选择。
*   **必要条件**：无。当启用时，它取代了 `isolatedModules` 的作用。
*   **示例**：
    ```json
    {
      "compilerOptions": {
        "verbatimModuleSyntax": true
      }
    }
    ```

### 三、项目结构配置（项目引用）

此类配置用于管理多项目（Monorepo）结构的 TypeScript 代码库。

#### 1. `references`
*   **特点**：`tsconfig.json` 的顶级属性（不在 `compilerOptions` 内），用于声明当前项目所依赖的其他 TypeScript 项目。它能实现跨项目的智能构建和精确的类型检查。
*   **必要条件**：所引用的子项目必须在其配置中启用 `composite: true`。
*   **示例**：
    ```json
    {
      "references": [
        { "path": "../packages/shared" },
        { "path": "../packages/utils" }
      ],
      "compilerOptions": { ... }
    }
    ```

#### 2. `composite`
*   **特点**：启用此选项表明该项目是一个“复合”项目，可以被其他项目引用。它会自动启用某些限制和优化，以便 TypeScript 能快速定位其输出。
*   **必要条件**：如果项目被其他项目在 `references` 中引用，则**必须**启用。启用后，会自动或强制进行以下设置：
    1.  `declaration: true`（必须生成 `.d.ts` 文件）。
    2.  所有源文件必须被 `include` 或 `files` 显式包含。
    3.  若未设置 `rootDir`，则默认为包含 `tsconfig.json` 的目录。
*   **示例**（被引用的子项目配置）：
    ```json
    {
      "compilerOptions": {
        "composite": true,
        "declaration": true,
        "rootDir": "src",
        "outDir": "dist"
      },
      "include": ["src/**/*"]
    }
    ```

#### 3. `declaration` & `declarationMap`
*   **特点**：
    *   `declaration`：生成 `.d.ts` 类型声明文件，这是库发布和项目引用所必需的。
    *   `declarationMap`：生成 `.d.ts.map` 文件，使得在支持此功能的编辑器中，可以对来自 `.d.ts` 的符号（如从依赖库导入的函数）进行“转到定义”操作，并跳转到原始的 `.ts` 源码，极大提升开发体验。
*   **必要条件**：对于库项目或作为项目引用一部分的项目，`declaration` 应为 `true`。`declarationMap` 是可选的，但推荐开启。
*   **示例**：
    ```json
    {
      "compilerOptions": {
        "declaration": true,
        "declarationMap": true
      }
    }
    ```

### 四、类型检查严格度配置

此类配置通过不同程度的约束来提升代码的类型安全。

#### 1. `strict`
*   **特点**：这是一个总开关，启用后即同时激活所有严格类型检查选项（如 `strictNullChecks`, `noImplicitAny` 等）。强烈建议在所有新项目中启用。
*   **必要条件**：无。对于库项目，启用 `strict` 是**最佳实践**，可以确保库的类型在其使用者启用严格模式时也保持安全。
*   **示例**：
    ```json
    {
      "compilerOptions": {
        "strict": true
      }
    }
    ```

#### 2. `strictNullChecks`
*   **特点**：当启用时，`null` 和 `undefined` 具有自己独立的类型，不再被自动视为任何类型的子类型。这能有效避免“未定义不是对象”这类常见的运行时错误。是 `strict` 模式包含的核心选项之一。
*   **必要条件**：无，但强烈建议启用。
*   **示例**（`strict: false` 时单独启用）：
    ```json
    {
      "compilerOptions": {
        "strict": false,
        "strictNullChecks": true
      }
    }
    ```

#### 3. `noUnusedLocals` / `noUnusedParameters`
*   **特点**：
    *   `noUnusedLocals`：报告未使用的局部变量错误。
    *   `noUnusedParameters`：报告未使用的函数参数错误。
    这两个选项有助于保持代码简洁，消除无效代码。
*   **必要条件**：无。
*   **示例**：
    ```json
    {
      "compilerOptions": {
        "noUnusedLocals": true,
        "noUnusedParameters": true
      }
    }
    ```

### 配置组合示例

以下是针对不同场景的推荐配置组合，请注意替换注释中的“其他重要选项”。

#### 1. 使用打包工具的现代前端应用（如 Vite, Webpack）
```json
{
  "compilerOptions": {
    // 模块与解析
    "module": "esnext",
    "moduleResolution": "bundler",
    "esModuleInterop": true,
    // 类型检查
    "strict": true,
    "verbatimModuleSyntax": true,
    // 输出
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "outDir": "./dist",
    "rootDir": "./src",
    // 路径别名（按需）
    "baseUrl": "./src",
    "paths": {
      "@/*": ["./*"]
    },
    // 其他重要选项...
    "skipLibCheck": true // 通常建议为true以加速编译
  }
}
```

#### 2. Node.js 服务端项目（使用原生 ES 模块）
```json
// package.json 中需要设置 "type": "module"
{
  "compilerOptions": {
    // Node.js 现代模块系统
    "module": "nodenext",
    // "nodenext" 隐含了 moduleResolution, esModuleInterop 等设置
    // 类型检查
    "strict": true,
    "verbatimModuleSyntax": true,
    // 输出
    "target": "ES2022",
    "lib": ["ES2022"],
    "outDir": "./dist",
    "rootDir": "./src",
    // 其他重要选项...
  }
}
```

#### 3. 用于发布的库（Library）
```json
{
  "compilerOptions": {
    // 使用 node16/nodenext 以获得最广泛的兼容性（尤其是Node.js环境）
    "module": "node16",
    "target": "es2020", // 设定你支持的最低ES版本
    // 类型与输出
    "strict": true,
    "verbatimModuleSyntax": true, // 关键！确保导入/导出声明的可移植性
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    // 其他重要选项...
  }
}
```

#### 4. Monorepo 子包配置
```json
// 子包（如 packages/utils）的 tsconfig.json
{
  "extends": "../../tsconfig.base.json", // 继承共享的基础配置
  "compilerOptions": {
    "composite": true, // 关键！允许被其他项目引用
    "declaration": true, // composite 会强制要求此项
    "declarationMap": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "references": [
    // 声明此子包所依赖的其他内部子包
    { "path": "../shared" }
  ],
  "include": ["src/**/*"] // composite 要求所有文件必须被显式包含
}
```