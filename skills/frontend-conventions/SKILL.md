---
name: frontend-conventions
description: >-
  按统一约定实现 React 前端。新建项目用官方 create-react-router 脚手架
  （pnpm create react-router@latest），不手写工程骨架。生成后按实际目录接入约定。
  默认纯客户端；文档要求 SSR、SEO 或 GEO 时再开服务端渲染。已有 Data Mode 仓库保持
  Data Mode。约定包含 Tailwind CSS 4 的 @theme inline 语义色、Base UI、包 cn、
  类型化 fetch、kebab-case 与 Vitest。在新建前端、加页面/组件/接口/类型/测试，或提到
  前端、React、Vite、React Router 时使用。React 侧是否使用 TanStack Query 必须先问用户。
  Zustand 只用于全局状态或过深的透传。在已有项目中只对照差异并报告；用户没明确要求修改时不改代码。
---

# 前端约定

新建项目先跑官方脚手架，再在生成结果上补技术栈、目录、风格、请求、类型和测试。扩展已有项目时保持它现有的路由模式。产品文档里的页面、权限和文案按该文档做；本 skill 不规定具体业务。

## 已有项目

用本约定对照当前实现，向用户说明差异。用户没有明确要求修改代码时，停在报告：不改文件，不顺手迁移，也不跑会改写代码的命令。下面的测试和浏览器验证只在用户要求实现或修改时执行。

已有仓库保持现有路由模式和 `ssr` 设置。存在 `app/routes.ts`、`react-router.config.ts` 或 `@react-router/dev` 时继续 Framework Mode。存在 `createBrowserRouter` 时继续 Data Mode，沿用现有 `src/` 入口，不改成脚手架，也不重写 `main.tsx` 和 `router.tsx`。

包管理器按本约定用 pnpm。用户只要求对照差异时，把非 pnpm 写进报告，不改 lockfile。用户要求实现或修改时，改用 pnpm：生成 `pnpm-lock.yaml`，去掉其他 lockfile，之后的安装和脚本都走 pnpm。

## 新建项目

用官方默认模板，包管理器只用 pnpm。不手写工程骨架，也不要从仓库抄模板文件。

```bash
pnpm create react-router@latest <dir> -- --yes --package-manager pnpm
```

`<dir>` 必填。`--yes` 使用默认模板并跳过提问。`--package-manager pnpm` 让脚手架用 pnpm 安装。不选带数据库、自定义服务器或 unstable 的模板，除非用户明确要求。

单体仓库在空的前端包目录里跑同一条命令。目录里已有文件时先问用户，同意后再加 `--overwrite`。

生成之后看实际目录，再接下文的分层、样式、请求和测试。入口、路由和配置保持脚手架的写法，缺的再补。

## 路由模式

新建项目用脚手架的 Framework Mode。入口、路由和脚本以生成结果为准。

纯客户端是默认：管理后台、内部工具，以及文档没提服务端或收录需求。文档写明服务端渲染、服务端数据、SEO 或 GEO 时再开服务端渲染。渲染开关改脚手架实际生成的配置。

已有 Data Mode 仓库继续用现有入口和路由。两种情况共用下文的目录分层、命名、样式、请求、类型和测试。

## 技术栈

| 领域 | 选型 |
| --- | --- |
| UI | React（跟脚手架） |
| 路由 | React Router（跟脚手架）。新建用 Framework Mode；已有 Data Mode 仓库保持 Data Mode |
| 构建 | Vite（跟脚手架） |
| 语言 | TypeScript strict，`verbatimModuleSyntax`。脚手架已有就沿用 |
| 样式 | Tailwind CSS 4；复杂样式用同目录 CSS Modules。脚手架已接入就沿用 |
| 交互原语 | Base UI（`@base-ui/react`）：Dialog、Menu、Select、Popover、Tabs 等无样式组件，外观用 Tailwind 或同目录 CSS Module |
| 图标 | `lucide-react` |
| 类名 | `cn`（包名 `cn`）。组件里 `import { cn } from "cn"` |
| 请求 | 自研 `fetch` 封装。是否再用 TanStack Query 由用户选择 |
| 客户端状态 | 局部状态留在组件。Zustand 只用于全局状态，或透传层级已经过多的状态 |
| 包管理器 | pnpm。安装与脚本都用 `pnpm add`、`pnpm add -D`、`pnpm dev`、`pnpm test`、`pnpm typecheck` |
| 测试 | Vitest + Testing Library + jsdom |

路径别名沿用脚手架。已有 Data Mode 项目的 `~/*` 指向 `./src/*`。Vite 与 Vitest 都打开 `resolve.tsconfigPaths`，没有再加。

## 目录

业务代码放进脚手架的源码目录，按实际路径接入，不重排入口和路由。

```text
components/
├── ui/
├── biz/<domain>/
└── shared/
hooks/
stores/                  # 只有全局状态或过深透传才建
lib/
├── api/                 # api-client.ts + 按资源拆分的具名函数
└── utils/
types/
```

不设 `lib/classname.ts`。类名直接使用 `cn`。路由文件名跟脚手架和当前 React Router 约定。已有 Data Mode 的路由文件用 kebab-case，例如 `item-detail.tsx`。

| 目录 | 可以依赖 |
| --- | --- |
| `ui/` | Base UI、Tailwind、`cn`、同目录 CSS Module、少量 `lib/utils` |
| `shared/` | `ui/`、hooks、通用类型 |
| `biz/<domain>/` | `ui/`、`shared/`、本域 store 和 API |

`ui/` 不引用业务模块、store 或具体 API。`shared/` 不承载单一路由的业务语义。跨域复用下沉到 `shared/`。

## 客户端状态

Zustand 只在两类场景使用：

- 全局状态：登录态、主题偏好（`system | light | dark`）
- 组件之间透传层级已经过多

其余状态放在拥有它的组件里。服务端资源放在路由 `loader`，或放在用户选中的 TanStack Query。不要为每个功能默认建 store。组件只订阅自己渲染用到的字段。

主题解析为 light 或 dark 后，只在根元素切换 `.dark`。组件通过语义色工具类取色，不在组件里写深浅两套颜色。

## 代码风格

| 对象 | 约定 | 示例 |
| --- | --- | --- |
| 文件名、目录名 | kebab-case | `message-bubble.tsx`、`use-leave-confirm.ts`、`auth-store.ts` |
| CSS Module | 与组件同名 | `message-bubble.module.css` |
| 组件导出 | PascalCase 具名函数 | `export function MessageBubble` |
| 函数、变量 | camelCase | `getConversation`、`isSaving` |
| 类型 | PascalCase，用 `type` | `ConversationDetail`、`UpdateConversationInput` |

组件、hooks、store、API 使用具名导出。Framework Mode 的 route module 按 React Router 默认导出页面组件，并另外导出 `loader`。类型导入写 `import type`。

```ts
import { cn } from "cn";
import { MessageBubble } from "~/components/biz/conversation/reader/message-bubble";
```

样式：

1. 布局、间距、颜色、排版和简单状态用 Tailwind。
2. 复杂选择器、多步动画、伪元素和第三方结构，用同目录 `*.module.css`，以 `styles.xxx` 引用。
3. 设计 token 只放在脚手架的全局样式入口。
4. 颜色使用主题工具类：`border-subtle`、`bg-page`、`text-ink`。`[…]` 只用于非主题值，如 `rounded-[4px]`。
5. Base UI 的外观用 data-attribute + Tailwind，或包进 `ui/*` 再用 Module 限定作用域。
6. 布局断点写 Tailwind 响应式前缀或 Module 里的 `@media`。`matchMedia` 只用于主题等非布局行为。

### 颜色

每个项目都做浅色和暗色。`:root` 写基底色，`.dark` 覆盖基底色，语义主题色放在 `@theme inline`。组件使用由此生成的工具类。

语义色优先、并且只使用 `@theme inline`。普通 `@theme` 会在 `:root` 生成 `--color-test: var(--success)`，`var(--success)` 在根上算成具体值再继承；子元素之后改 `--success`，`bg-test` 仍读 `--color-test`，不会跟着变。`inline` 把 `var(--success)` 直接写进工具类，子元素重写基底色才会生效。

基底色用 `--paper`、`--ink` 这类名字。`--color-*` 只出现在 `@theme inline` 里，用来生成 `bg-` / `text-` / `border-`。

`@theme inline` 的值使用 `var(--基底)`，或包含 `var()` 的 `color-mix`、`light-dark()`。语义色跟着基底变，不为每个语义色再写一套深色色值。`light-dark()` 跟随 `color-scheme`；用类名切换时，`.dark` 里写 `color-scheme: dark`。

浅色基底用独立变量保住。`.dark` 优先用 `color-mix` 从浅色基底推出深色基底；mix 表达不了的色相再写深色值。

`@theme inline` 的主题色会直接写进工具类，不会再生成同名自定义属性。CSS Module 需要颜色时，引用基底变量或在 Module 里写 `color-mix`。

```css
@import "tailwindcss";

@custom-variant dark (&:is(.dark *));

:root {
  --paper-base: oklch(0.93 0.03 85);
  --ink-base: oklch(0.28 0.02 70);
  --accent-base: oklch(0.52 0.09 55);
  --danger-base: oklch(0.48 0.12 35);

  --paper: var(--paper-base);
  --ink: var(--ink-base);
  --accent: var(--accent-base);
  --danger: var(--danger-base);
  color-scheme: light;
}

.dark {
  --paper: color-mix(in oklab, var(--paper-base) 16%, black);
  --ink: color-mix(in oklab, var(--ink-base) 18%, white);
  --accent: color-mix(in oklab, var(--accent-base) 72%, white);
  --danger: color-mix(in oklab, var(--danger-base) 70%, white);
  color-scheme: dark;
}

@theme inline {
  --color-page: var(--paper);
  --color-panel: color-mix(in oklab, var(--ink) 6%, var(--paper));
  --color-ink: var(--ink);
  --color-muted: color-mix(in oklab, var(--ink) 62%, var(--paper));
  --color-subtle: color-mix(in oklab, var(--ink) 12%, transparent);
  --color-strong: color-mix(in oklab, var(--ink) 22%, transparent);
  --color-accent: var(--accent);
  --color-danger: var(--danger);
}
```

上面的 `--color-subtle` 同时生成 `border-subtle`、`text-subtle` 和 `bg-subtle`。语义色沿用 Tailwind 的 `bg-` / `text-` / `border-` 前缀，`cn` 才能让后写的颜色覆盖先写的。

```tsx
import { cn } from "cn";
import styles from "./message-bubble.module.css";

export function MessageBubble({ variant, children }: Props) {
  return (
    <div className={cn(styles.bubble, styles[variant], "max-w-prose border border-subtle")}>
      {children}
    </div>
  );
}
```

## 接口请求

`lib/api/api-client.ts` 是唯一的 `fetch` 入口。每个资源一个文件，导出具体类型的具名函数。组件、loader 和 Query 的 `queryFn` / `mutationFn` 都调用这些函数。

函数名以动词开头，读得出动作：

| 动作 | 命名 |
| --- | --- |
| 读取一条 | `getConversation` |
| 读取列表 | `getConversations` 或 `listConversations` |
| 创建 | `createConversation` |
| 更新 | `updateConversation` |
| 删除 | `deleteConversation` |
| 上传 | `uploadImage` |

名字里要能看出是创建还是更新。`saveConversation`、`adminImageFile` 这种看不出动作的名字不用。

在把请求接到 React 组件之前，先让用户选择要不要使用 TanStack Query。用提问让用户二选一，得到选择后再安装。未选择时不引入 `@tanstack/react-query`。

| 用户选择 | 做法 |
| --- | --- |
| 使用 TanStack Query | 组件中的读取、缓存、变更、重试和失效走 Query。`queryFn` / `mutationFn` 只调用具名 API |
| 不使用 | 路由读取留在 `loader`，页面从 `loaderData` 读取。写入成功后 `revalidate` 或导航刷新 |

选定之后，同一个资源只走一条缓存路径。

其余规则：

- `apiRequest` 的调用处显式传入 `TResponse`。`TBody` 默认为 `never`；`POST` / `PUT` / `PATCH` 传入具体 Input 类型。
- 对外契约使用具体类型。基础设施才使用泛型。
- 成功返回 JSON；`204` 返回 `undefined`。失败解析 `{ code, message, details?, requestId? }` 并抛 `ApiError`。
- 默认 `credentials: "include"`。有 CSRF token 时，非 GET 附带后端约定的头。
- query 里的 `null` 和 `undefined` 不写入 URL。
- JSON body 与 `FormData` 分开。上传单独写函数，例如 `uploadImage(file: File): Promise<MediaAsset>`。
- 时间字段在类型里是 `string`，值为 ISO 8601 UTC。展示转换集中在 `lib/datetime.ts`。
- `401` 抛出带后端 `code` 的 `ApiError`，由根布局一处处理。

```ts
export async function apiRequest<TResponse, TBody = never>(
  path: string,
  options?: {
    method?: "GET" | "POST" | "PUT" | "PATCH" | "DELETE";
    body?: TBody;
    query?: Record<string, string | number | boolean | null | undefined>;
    signal?: AbortSignal;
    formData?: FormData;
    headers?: HeadersInit;
  },
): Promise<TResponse>;
```

```ts
export function getConversation(id: string): Promise<ConversationDetail> {
  return apiRequest<ConversationDetail>(`/conversations/${id}`);
}

export function updateConversation(
  id: string,
  input: UpdateConversationInput,
): Promise<ConversationDetail> {
  return apiRequest<ConversationDetail, UpdateConversationInput>(
    `/conversations/${id}`,
    { method: "PUT", body: input },
  );
}
```

浏览器里发的请求 base URL 用 `/api`，由 Vite 代理到后端。loader 跑在服务端、`window` 不存在时，改用环境变量里的绝对 API 地址。

## Types

DTO 放在 `types/<resource>.ts`，字段名与后端 camelCase JSON 一致。有 OpenAPI 时可生成，否则手写。

- 响应用资源名：`ConversationListItem`、`ConversationDetail`。
- 写入用 `CreateXInput`、`UpdateXInput`。
- 枚举用字符串联合类型。
- 列表信封与后端一致。后端使用游标分页时，共享：

```ts
export type Paginated<T> = {
  items: T[];
  nextCursor: string | null;
  total: number;
  page: number | null;
};
```

`ApiError` 与 `ApiErrorBody` 放在 `types/api.ts`。组件里的字段错误从 `ApiError.details` 映射，保留用户已输入的内容。

## 测试和验证

测试文件与源文件同目录：`api-client.spec.ts`、`message-bubble.spec.tsx`。

| 层级 | 范围 |
| --- | --- |
| 单元 | `apiRequest` 的成功 JSON、204、错误体、query 省略、CSRF 头；日期和其它纯函数 |
| API 函数 | mock `apiRequest`，断言 path、method、body、query |
| 组件 | Testing Library 覆盖条件渲染、提交和关键键盘操作 |

接口失败的测试要断言调用方仍保留用户输入。

用户要求实现或修改时，完成后运行 `pnpm test` 和 `pnpm typecheck`。`typecheck` 用脚手架已有脚本；没有该脚本时用 `pnpm exec tsc --noEmit`。改了 UI、布局、路由或展示数据时，在浏览器里走通主路径、空状态和错误状态；断点有变化时同时看桌面宽度和窄屏。相关路由若读取同一状态，一并核对。主题改动要同时看浅色和暗色。
