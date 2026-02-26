# @plainbrew/tsconfig

plainbrew プロジェクト向けの strict な TypeScript 設定。

## 使い方

> **TypeScript >= 5.8 が必要です** (`erasableSyntaxOnly` は TS 5.8+、`noUncheckedSideEffectImports` は TS 5.6+)

```jsonc
// ベース
{ "extends": "@plainbrew/tsconfig" }

// React
{ "extends": "@plainbrew/tsconfig/react" }
```

## 設計判断

`@tsconfig/strictest`, `@sindresorhus/tsconfig`, `@total-typescript/tsconfig` 等の大規模 OSS を参考に設計。

### Strict 系

| オプション                                                                                                                                                                  | 採用理由                                                                                                                                    |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| [`strict`](https://www.typescriptlang.org/tsconfig/#strict)                                                                                                                 | `strictNullChecks`, `noImplicitAny` 等を一括有効化                                                                                          |
| [`exactOptionalPropertyTypes`](https://www.typescriptlang.org/tsconfig/#exactOptionalPropertyTypes)                                                                         | `undefined` の明示的な使用を強制。`@tsconfig/strictest` と同方針                                                                            |
| [`noFallthroughCasesInSwitch`](https://www.typescriptlang.org/tsconfig/#noFallthroughCasesInSwitch)                                                                         | switch 文の break 忘れを防止                                                                                                                |
| [`noImplicitOverride`](https://www.typescriptlang.org/tsconfig/#noImplicitOverride)                                                                                         | `override` キーワードの明示を強制し、継承元の変更検知を容易に                                                                               |
| [`noImplicitReturns`](https://www.typescriptlang.org/tsconfig/#noImplicitReturns)                                                                                           | 全コードパスでの return を強制                                                                                                              |
| [`noPropertyAccessFromIndexSignature`](https://www.typescriptlang.org/tsconfig/#noPropertyAccessFromIndexSignature)                                                         | index signature へのドットアクセスを禁止し、bracket notation を強制                                                                         |
| [`noUncheckedIndexedAccess`](https://www.typescriptlang.org/tsconfig/#noUncheckedIndexedAccess)                                                                             | 配列・オブジェクトのインデックスアクセスに `undefined` を含める                                                                             |
| [`noUncheckedSideEffectImports`](https://www.typescriptlang.org/tsconfig/#noUncheckedSideEffectImports)                                                                     | `import "./styles.css"` 等の副作用 import の存在チェック (TS 5.6+)。`@sindresorhus/tsconfig` が採用                                         |
| [`noUnusedLocals`](https://www.typescriptlang.org/tsconfig/#noUnusedLocals) / [`noUnusedParameters`](https://www.typescriptlang.org/tsconfig/#noUnusedParameters)           | 未使用変数・引数を検出                                                                                                                      |
| [`allowUnusedLabels`](https://www.typescriptlang.org/tsconfig/#allowUnusedLabels) / [`allowUnreachableCode`](https://www.typescriptlang.org/tsconfig/#allowUnreachableCode) | `false` に設定。到達不能コードや未使用ラベルをエラーに                                                                                      |
| [`erasableSyntaxOnly`](https://www.typescriptlang.org/tsconfig/#erasableSyntaxOnly)                                                                                         | `enum`, `namespace` 等の型消去不可能な構文を禁止 (TS 5.8+)。Node.js の `--experimental-strip-types` と互換。`@sindresorhus/tsconfig` が採用 |

### Module 系

| オプション                                                                                   | 採用理由                                                                                                                      |
| -------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| [`module`](https://www.typescriptlang.org/tsconfig/#module): `"preserve"`                    | import/export をそのまま保持しバンドラーに委ねる (TS 5.4+)                                                                    |
| [`moduleResolution`](https://www.typescriptlang.org/tsconfig/#moduleResolution): `"bundler"` | Vite, webpack, Next.js 等のバンドラー環境に最適化                                                                             |
| [`moduleDetection`](https://www.typescriptlang.org/tsconfig/#moduleDetection): `"force"`     | 全ファイルをモジュールとして扱いグローバルスコープ汚染を防止。`@total-typescript/tsconfig`, `@sindresorhus/tsconfig` が採用   |
| [`isolatedModules`](https://www.typescriptlang.org/tsconfig/#isolatedModules)                | ファイル単位のトランスパイルを保証。esbuild, SWC 等との互換性確保                                                             |
| [`esModuleInterop`](https://www.typescriptlang.org/tsconfig/#esModuleInterop)                | CJS モジュールの default import を安全に扱う。`verbatimModuleSyntax` は CJS 環境で問題があるため不採用（ESLint ルールで代替） |
| [`resolveJsonModule`](https://www.typescriptlang.org/tsconfig/#resolveJsonModule)            | JSON ファイルの型安全な import を許可                                                                                         |
| [`skipLibCheck`](https://www.typescriptlang.org/tsconfig/#skipLibCheck)                      | `.d.ts` のチェックをスキップしビルド高速化。ほぼ全ての主要パッケージが採用                                                    |

### 意図的に含めないオプション

| オプション                                                                                                                                                                                                              | 不採用理由                                                                                                          |
| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| [`target`](https://www.typescriptlang.org/tsconfig/#target) / [`lib`](https://www.typescriptlang.org/tsconfig/#lib)                                                                                                     | 実行環境に依存するため拡張先に委ねる                                                                                |
| [`declaration`](https://www.typescriptlang.org/tsconfig/#declaration) / [`declarationMap`](https://www.typescriptlang.org/tsconfig/#declarationMap) / [`sourceMap`](https://www.typescriptlang.org/tsconfig/#sourceMap) | output 系は strictness と無関係。app と library で要件が異なる                                                      |
| [`verbatimModuleSyntax`](https://www.typescriptlang.org/tsconfig/#verbatimModuleSyntax)                                                                                                                                 | CJS 出力環境で `import` を `require()` に変換不可。ESLint (`@typescript-eslint/consistent-type-imports`) で代替可能 |
| [`forceConsistentCasingInFileNames`](https://www.typescriptlang.org/tsconfig/#forceConsistentCasingInFileNames)                                                                                                         | TS 5.0 以降デフォルト `true`                                                                                        |
| [`noEmit`](https://www.typescriptlang.org/tsconfig/#noEmit)                                                                                                                                                             | バンドラー/フレームワーク設定に依存                                                                                 |

### React 設定 (`react.json`)

ベースを継承し、React 固有のオプションのみ追加:

- [`jsx`](https://www.typescriptlang.org/tsconfig/#jsx): `"react-jsx"` — React 17+ の JSX Transform を使用
- [`lib`](https://www.typescriptlang.org/tsconfig/#lib): `["dom", "dom.iterable", "esnext"]` — ブラウザ API の型定義
