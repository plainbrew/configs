# @plainbrew/tsconfig

plainbrew プロジェクト向けの strict な TypeScript 設定。

## 使い方

```jsonc
// ベース
{ "extends": "@plainbrew/tsconfig" }

// React
{ "extends": "@plainbrew/tsconfig/react" }
```

## 設計判断

`@tsconfig/strictest`, `@sindresorhus/tsconfig`, `@total-typescript/tsconfig` 等の大規模 OSS を参考に設計。

### Strict 系

| オプション                                                 | 採用理由                                                                                                                                    |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `strict`                                                   | `strictNullChecks`, `noImplicitAny` 等を一括有効化                                                                                          |
| `exactOptionalPropertyTypes`                               | `undefined` の明示的な使用を強制。`@tsconfig/strictest` と同方針                                                                            |
| `noFallthroughCasesInSwitch`                               | switch 文の break 忘れを防止                                                                                                                |
| `noImplicitOverride`                                       | `override` キーワードの明示を強制し、継承元の変更検知を容易に                                                                               |
| `noImplicitReturns`                                        | 全コードパスでの return を強制                                                                                                              |
| `noPropertyAccessFromIndexSignature`                       | index signature へのドットアクセスを禁止し、bracket notation を強制                                                                         |
| `noUncheckedIndexedAccess`                                 | 配列・オブジェクトのインデックスアクセスに `undefined` を含める                                                                             |
| `noUncheckedSideEffectImports`                             | `import "./styles.css"` 等の副作用 import の存在チェック (TS 5.6+)。`@sindresorhus/tsconfig` が採用                                         |
| `noUnusedLocals` / `noUnusedParameters`                    | 未使用変数・引数を検出                                                                                                                      |
| `allowUnusedLabels: false` / `allowUnreachableCode: false` | 到達不能コードや未使用ラベルをエラーに                                                                                                      |
| `erasableSyntaxOnly`                                       | `enum`, `namespace` 等の型消去不可能な構文を禁止 (TS 5.8+)。Node.js の `--experimental-strip-types` と互換。`@sindresorhus/tsconfig` が採用 |

### Module 系

| オプション                    | 採用理由                                                                                                                      |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| `module: "preserve"`          | import/export をそのまま保持しバンドラーに委ねる (TS 5.4+)                                                                    |
| `moduleResolution: "bundler"` | Vite, webpack, Next.js 等のバンドラー環境に最適化                                                                             |
| `moduleDetection: "force"`    | 全ファイルをモジュールとして扱いグローバルスコープ汚染を防止。`@total-typescript/tsconfig`, `@sindresorhus/tsconfig` が採用   |
| `isolatedModules`             | ファイル単位のトランスパイルを保証。esbuild, SWC 等との互換性確保                                                             |
| `esModuleInterop`             | CJS モジュールの default import を安全に扱う。`verbatimModuleSyntax` は CJS 環境で問題があるため不採用（ESLint ルールで代替） |
| `resolveJsonModule`           | JSON ファイルの型安全な import を許可                                                                                         |
| `skipLibCheck`                | `.d.ts` のチェックをスキップしビルド高速化。ほぼ全ての主要パッケージが採用                                                    |

### 意図的に含めないオプション

| オプション                                     | 不採用理由                                                                                                          |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `target` / `lib`                               | 実行環境に依存するため拡張先に委ねる                                                                                |
| `declaration` / `declarationMap` / `sourceMap` | output 系は strictness と無関係。app と library で要件が異なる                                                      |
| `verbatimModuleSyntax`                         | CJS 出力環境で `import` を `require()` に変換不可。ESLint (`@typescript-eslint/consistent-type-imports`) で代替可能 |
| `forceConsistentCasingInFileNames`             | TS 5.0 以降デフォルト `true`                                                                                        |
| `noEmit`                                       | バンドラー/フレームワーク設定に依存                                                                                 |

### React 設定 (`react.json`)

ベースを継承し、React 固有のオプションのみ追加:

- `jsx: "react-jsx"` — React 17+ の JSX Transform を使用
- `lib: ["dom", "dom.iterable", "esnext"]` — ブラウザ API の型定義
