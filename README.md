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

**⚠️ 注意：実装とコストの複雑さ**
- **開発コスト**: Server と WebAssembly の両方に対応する必要があり、実装難易度が大幅に上がる
- **運用コスト**: サーバーリソースとクライアントリソースの両方を管理する必要がある
- **デバッグ**: 2つの異なる実行環境でのテストと問題の切り分けが必要
- **バンドルサイズ**: WebAssembly のダウンロードが必要で、初期表示は早いが総データ量は多い

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

```mermaid
flowchart TD
    Start([アプリケーションを作成したい]) --> ServerChoice{サーバーが必要？}
    
    ServerChoice -->|サーバーが不要<br/>静的ホスティングのみ| Standalone[Blazor WebAssembly<br/>Standalone]
    
    ServerChoice -->|サーバーありの<br/>Web アプリケーション| Interactive{インタラクティブ機能が必要？}
    
    Interactive -->|インタラクティブ機能が不要| StaticSSR[Static SSR]
    
    Interactive -->|インタラクティブ機能が必要| RenderMode{レンダーモードの選択}
    
    RenderMode -->|単一のレンダーモードで統一したい| GlobalMode{グローバルモードの選択}
    
    GlobalMode -->|サーバーリソース重視<br/>セキュリティ重視| ServerGlobal[Interactive Server<br/>グローバル]
    
    GlobalMode -->|オフライン対応<br/>サーバー負荷軽減| WasmGlobal[Interactive WebAssembly<br/>グローバル]
    
    GlobalMode -->|最適なUX<br/>初回速度 + 継続性能| AutoGlobal[Interactive Auto<br/>グローバル]
    
    RenderMode -->|ページごとに最適化したい| MixedMode[Blazor Web App<br/>混在モード]
    
    MixedMode --> StaticPage[静的ページ → Static SSR]
    MixedMode --> DBPage[DB操作ページ → Interactive Server]
    MixedMode --> OfflinePage[オフライン対応ページ → Interactive WebAssembly]
    MixedMode --> GeneralPage[汎用ページ → Interactive Auto]
    
    %% 警告の追加
    AutoGlobal -.->|⚠️ 注意| CostWarning[開発・運用コストが高い<br/>実装難易度が大幅に上がる]
    GeneralPage -.->|⚠️ 注意| CostWarning
    
    %% スタイリング
    classDef warning fill:#fff2cc,stroke:#d6b656,stroke-width:2px
    classDef autoMode fill:#ffe6e6,stroke:#ff6b6b,stroke-width:2px
    classDef decision fill:#e1f5fe,stroke:#0288d1,stroke-width:2px
    classDef endpoint fill:#e8f5e8,stroke:#4caf50,stroke-width:2px
    
    class CostWarning warning
    class AutoGlobal,GeneralPage autoMode
    class ServerChoice,Interactive,RenderMode,GlobalMode decision
    class Standalone,StaticSSR,ServerGlobal,WasmGlobal,StaticPage,DBPage,OfflinePage endpoint
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

| 要件 | 推奨選択肢 | 実装難易度 |
|------|-----------|-----------|
| 高速な初期読み込み | Static SSR | 低 |
| SEO 最適化 | Static SSR | 低 |
| オフライン動作 | Interactive WebAssembly / Blazor WebAssembly Standalone | 中 |
| 完全な .NET API アクセス | Interactive Server | 低 |
| リアルタイム通信 | Interactive Server | 中 |
| サーバー負荷軽減 | Interactive WebAssembly / Blazor WebAssembly Standalone | 中 |
| 静的ホスティング | Interactive WebAssembly / Blazor WebAssembly Standalone | 中 |
| 最適なUX（初回＋継続） | Interactive Auto | **高** |
| セキュリティ重視 | Interactive Server | 低 |
| サーバーレス | Blazor WebAssembly Standalone | 中 |

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
- **選択肢**: Blazor Web App（Interactive Server グローバル）
- **推奨理由**: 業務アプリケーションは通常、データベースアクセスが頻繁でセキュリティが重要。Interactive Auto は開発・運用コストが高いため、特別な要件がない限り Interactive Server を推奨
- **備考**: 繰り返し利用される業務アプリで、かつ開発・運用リソースが十分にある場合のみ Interactive Auto を検討

### 4. **ポートフォリオサイト**
- **選択肢**: Blazor WebAssembly Standalone
- **GitHub Pages で配信**: 静的ホスティング
- **完全にオフライン動作**: サーバー不要

### 5. **デモアプリケーション**
- **選択肢**: Blazor WebAssembly Standalone
- **CDN で配信**: コスト削減
- **シンプルな SPA**: サーバーレス

### 6. **Interactive Auto を選択すべき場面**
- **エンタープライズアプリケーション**で、開発・運用チームが充実している
- **頻繁に利用されるアプリケーション**で、UX への投資対効果が高い
- **長期運用が前提**で、初期投資コストを回収できる見込みがある
- **技術的負債を管理できる体制**が整っている

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
- **Interactive Auto の場合**: Server と Client の両方のプロジェクトを管理する必要があり、依存関係の管理が複雑化

### 4. **Interactive Auto の実装コスト**
- **開発時間**: 単一モードの約2倍の実装時間が必要
- **テスト工数**: Server と WebAssembly の両方での動作確認が必要
- **保守性**: 2つの実行環境での問題の切り分けと対応が必要
- **チーム要件**: Server 側と Client 側の両方の技術スタックに精通したメンバーが必要

## 参考ドキュメント

- [ASP.NET Core Blazor render modes](https://learn.microsoft.com/en-us/aspnet/core/blazor/components/render-modes?view=aspnetcore-9.0)
- [ASP.NET Core Blazor hosting models](https://learn.microsoft.com/en-us/aspnet/core/blazor/hosting-models?view=aspnetcore-9.0)
- [Tooling for ASP.NET Core Blazor](https://learn.microsoft.com/en-us/aspnet/core/blazor/tooling?view=aspnetcore-9.0)
- [ASP.NET Core Blazor project structure](https://learn.microsoft.com/en-us/aspnet/core/blazor/project-structure?view=aspnetcore-9.0)