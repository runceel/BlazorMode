# ASP.NET Core Blazor Web App (.NET 8以降) - レンダーモード選択ガイド

.NET 8 以降の Blazor Web App では、コンポーネントレベルでレンダーモードを選択できるようになりました。従来のホスティングモデルという概念から、**レンダーモード**という概念に変わり、より柔軟にアプリケーションを構築できます。

## Blazor Web App の選択肢

### 1. **Static SSR（静的サーバーサイドレンダリング）**

**特徴：**
- サーバーで HTML を生成し、インタラクティブ機能なし
- 最も高速な初期読み込み
- SEO に最適
- イベントハンドラーは動作しない

**使うべき場面：**
- 静的なコンテンツ表示
- SEO が重要なページ
- 高速な初期読み込みが必要
- データ表示のみのページ

### 2. **Interactive Server**

**特徴：**
- SignalR を使用したサーバーサイドでのインタラクティブ処理
- 完全な .NET API アクセス
- サーバーリソースを活用
- リアルタイム通信

**使うべき場面：**
- サーバーリソースへの直接アクセスが必要
- 完全な .NET API が必要
- セキュリティが重要（コードがサーバー側に保護される）
- データベースアクセスが頻繁
- リアルタイム機能が必要

### 3. **Interactive WebAssembly**

**特徴：**
- クライアントサイドで実行
- オフライン動作可能
- サーバー負荷軽減
- 静的ホスティング可能

**使うべき場面：**
- オフライン動作が必要
- サーバー負荷を軽減したい
- 静的サイトホスティング（CDN）を使用
- クライアント側で重い処理を実行
- PWA (Progressive Web App) を構築

### 4. **Interactive Auto**

**特徴：**
- 初回はサーバーサイドで実行
- バックグラウンドで WebAssembly をダウンロード
- 2回目以降の訪問時は WebAssembly で実行
- 両方の利点を組み合わせ

**使うべき場面：**
- 初回の高速読み込みとその後の高性能を両立したい
- ユーザーが繰り返し訪問するアプリケーション
- 最適なユーザーエクスペリエンスを提供したい

## Blazor WebAssembly Standalone

### **Blazor WebAssembly Standalone**

**特徴：**
- 完全にクライアントサイドで実行される単独アプリケーション
- ASP.NET Core サーバーが不要
- 静的ファイルとして配信可能
- CDN やスタティックホスティングサービスで配信可能

**使うべき場面：**
- サーバーを持たないアプリケーション
- 完全にオフラインで動作するアプリケーション
- GitHub Pages や Azure Static Web Apps での配信
- シンプルな SPA (Single Page Application) を構築
- サーバーコストを削減したい

**制限事項：**
- .NET API のサブセットのみ利用可能
- 初期読み込み時間が長い
- サーバー側機能は Web API 経由でのみアクセス可能

## 選択フロー

```
アプリケーションを作成したい
│
├─ サーバーが不要 / 静的ホスティングのみ
│  └─ Blazor WebAssembly Standalone
│
├─ サーバーありの Web アプリケーション
│  │
│  ├─ インタラクティブ機能が不要
│  │  └─ Static SSR
│  │
│  ├─ インタラクティブ機能が必要
│  │  │
│  │  ├─ 単一のレンダーモードで統一したい
│  │  │  │
│  │  │  ├─ サーバーリソース重視 / セキュリティ重視
│  │  │  │  └─ Interactive Server（グローバル）
│  │  │  │
│  │  │  ├─ オフライン対応 / サーバー負荷軽減
│  │  │  │  └─ Interactive WebAssembly（グローバル）
│  │  │  │
│  │  │  └─ 最適なUX（初回速度 + 継続性能）
│  │  │     └─ Interactive Auto（グローバル）
│  │  │
│  │  └─ ページごとに最適化したい
│  │     └─ Blazor Web App（混在モード）
│  │        ├─ 静的ページ → Static SSR
│  │        ├─ DB操作ページ → Interactive Server
│  │        ├─ オフライン対応ページ → Interactive WebAssembly
│  │        └─ 汎用ページ → Interactive Auto
```

## インタラクティブ性の適用範囲

### グローバルインタラクティブ性
- アプリケーション全体で同じレンダーモードを使用
- 管理が簡単で、一貫性のあるユーザーエクスペリエンス
- 単一機能のアプリケーションに適している

### ページ/コンポーネント単位のインタラクティブ性
- ページやコンポーネントごとに異なるレンダーモードを選択可能
- 各ページの要件に応じた最適化が可能
- より複雑だが、柔軟性が高い

## レンダーモードの選択基準

| 要件 | 推奨選択肢 |
|------|-----------|
| 高速な初期読み込み | Static SSR |
| SEO 最適化 | Static SSR |
| オフライン動作 | Interactive WebAssembly / Blazor WebAssembly Standalone |
| 完全な .NET API アクセス | Interactive Server |
| リアルタイム通信 | Interactive Server |
| サーバー負荷軽減 | Interactive WebAssembly / Blazor WebAssembly Standalone |
| 静的ホスティング | Interactive WebAssembly / Blazor WebAssembly Standalone |
| 最適なUX（初回＋継続） | Interactive Auto |
| セキュリティ重視 | Interactive Server |
| サーバーレス | Blazor WebAssembly Standalone |

## 実践的な使い分け例

### 1. **コーポレートサイト**
- **選択肢**: Blazor Web App（混在モード）
- **静的ページ（会社情報、製品紹介など）**: Static SSR（SEO重視）
- **お問い合わせフォーム**: Interactive Server（データベースアクセス）

### 2. **Eコマースサイト**
- **選択肢**: Blazor Web App（混在モード）
- **商品一覧ページ**: Static SSR（SEO重視）
- **ショッピングカート**: Interactive WebAssembly（オフライン対応）

### 3. **業務アプリケーション**
- **選択肢**: Blazor Web App（Interactive Auto グローバル）
- **ダッシュボード**: Interactive Auto（バランス重視）
- **データ編集画面**: Interactive Auto（サーバー↔クライアント自動切り替え）

### 4. **ポートフォリオサイト**
- **選択肢**: Blazor WebAssembly Standalone
- **GitHub Pages で配信**: 静的ホスティング
- **完全にオフライン動作**: サーバー不要

### 5. **デモアプリケーション**
- **選択肢**: Blazor WebAssembly Standalone
- **CDN で配信**: コスト削減
- **シンプルな SPA**: サーバーレス

## 注意点

### 1. **レンダーモードの伝播**
- 親コンポーネントのレンダーモードが子コンポーネントに伝播される
- 異なるインタラクティブモード間での切り替えは不可

### 2. **パラメータの制限**
- Static 親から Interactive 子にパラメータを渡す場合、JSON シリアライズ可能である必要がある
- RenderFragment や子コンテンツは渡せない

### 3. **プロジェクト構成**
- WebAssembly や Auto モードを使用する場合、`.Client` プロジェクトが必要
- WebAssembly コンポーネントは `.Client` プロジェクトに配置
- Blazor WebAssembly Standalone は単一プロジェクト構成

## 参考ドキュメント

- [ASP.NET Core Blazor render modes](https://learn.microsoft.com/en-us/aspnet/core/blazor/components/render-modes?view=aspnetcore-9.0)
- [ASP.NET Core Blazor hosting models](https://learn.microsoft.com/en-us/aspnet/core/blazor/hosting-models?view=aspnetcore-9.0)
- [Tooling for ASP.NET Core Blazor](https://learn.microsoft.com/en-us/aspnet/core/blazor/tooling?view=aspnetcore-9.0)
- [ASP.NET Core Blazor project structure](https://learn.microsoft.com/en-us/aspnet/core/blazor/project-structure?view=aspnetcore-9.0)