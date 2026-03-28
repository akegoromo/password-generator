# パスワード生成ツール

[English](./README.en.md) | 日本語

文字構成、除外ルール、連続文字制約を細かく制御できる、クライアントサイド完結型のパスワード生成ツールです。バニラJavaScriptと暗号学的に安全な乱数生成で実装しています。

**🔗 デモサイト:** https://pswd-gen.pages.dev

## 設計思想

本ツールは以下の原則に基づいて実装されています：

- **ゼロトラスト型クライアントサイド設計**: すべての処理がブラウザ内で完結し、サーバー通信は一切なし
- **暗号学的に安全な乱数生成**: `Math.random()` ではなく `window.crypto.getRandomValues()` を使用
- **厳格なポリシー適用**: 生成後バリデーションにより全制約の100%遵守を保証
- **堅牢なUI設計**: ドロップダウンのみのインターフェースで入力検証の複雑性を排除

## 技術的特徴

**パスワード生成ルール:**
- 設定可能な長さ（8〜32文字、デフォルト: 13文字）
- 必ず小文字で開始
- 大文字・小文字・数字を必ず含む
- 数字は最低2つ必須
- 記号の使用をカスタマイズ可能な除外設定付きで選択可能
- 大文字・小文字それぞれ独立した連続文字数制限

**実装のハイライト:**
- ビルドプロセス不要の単一HTMLファイル
- 偏りのない文字分布を実現するFisher-Yatesシャッフル
- 制約検証付きリトライメカニズム（最大1000回試行）
- モバイル最適化を含むレスポンシブデザイン

## コア実装

### 暗号学的に安全な文字選択

```javascript
function getRandomChar(charSet) {
    if (charSet.length === 0) return '';
    const array = new Uint32Array(1);
    window.crypto.getRandomValues(array);
    return charSet[array[0] % charSet.length];
}

厳格なバリデーションループ

生成されたパスワードは、事後バリデーションによる厳密な検証を受けます：

function validatePassword(pwd, config, upperPool, lowerPool, numberPool) {
    // 1. 小文字で開始することを確認
    if (!lowerPool.includes(pwd[0])) return false;
    
    // 2. 最小数字数の検証
    let numberCount = 0;
    for (let char of pwd) {
        if (numberPool.includes(char)) numberCount++;
    }
    if (numberCount < 2) return false;
    
    // 3. 文字種要件の確認
    let hasUpper = pwd.split('').some(c => upperPool.includes(c));
    let hasLower = pwd.split('').some(c => lowerPool.includes(c));
    if (!hasUpper || !hasLower) return false;
    
    // 4. 連続文字制限の確認
    return !hasExcessiveConsecutive(pwd, upperPool, config.maxConsecutiveUpper) &&
           !hasExcessiveConsecutive(pwd, lowerPool, config.maxConsecutiveLower);
}

アーキテクチャ決定: なぜ単一ファイルなのか？

    依存関係ゼロ: package.json不要、ビルドツール不要、外部ライブラリ不要
    最大の可搬性: あらゆる静的ホスティングプラットフォームで動作
    即時デプロイ: Cloudflare Pages/GitHub Pagesへ直接アップロード可能
    明確な境界: すべてのロジックが1つのレビュー可能なファイルに集約

使い方
エンドユーザー向け

    デモサイトURLにアクセス
    ドロップダウンメニューで生成パラメータを設定
    30〜60個のパスワードを一度に生成
    ワンクリックで全パスワードをクリップボードにコピー

開発者向け

ローカル開発:

git clone https://github.com/akegoromo/password-generator.git
cd password-generator
# ブラウザでindex.htmlを開くか、任意の静的サーバーで配信
python -m http.server 8000  # Pythonを使った例

デプロイ:

    任意の静的ホスティングサービスに index.html をアップロード
    ビルド設定は不要
    Cloudflare Pages、GitHub Pages、Netlify、Vercelで動作確認済み

カスタマイズ: すべての生成パラメータはJavaScript設定オブジェクトで外部化されています：

// index.html内のカスタマイズポイント例
const CONFIG = {
    passwordLength: 13,        // デフォルト長
    includeSymbols: true,      // 記号使用
    excludedSymbols: "!$%&=",  // 除外する記号
    maxConsecutiveUpper: 4,    // 大文字最大連続数（3+1）
    maxConsecutiveLower: 4,    // 小文字最大連続数（3+1）
    excludedChars: "0OI1l"     // 紛らわしい文字
};

セキュリティに関する考慮事項
本ツールが提供するもの

✅ 暗号学的に安全な乱数生成
✅ クライアントサイド処理（ネットワーク送信なし）
✅ 設定可能な文字除外機能
✅ 厳格な制約遵守
✅ セキュリティレビュー可能なオープンソースコード
本ツールが提供しないもの

❌ 実際の攻撃に対するパスワード強度評価
❌ クリップボード監視からの保護
❌ 特定のセキュリティポリシーへの準拠保証
❌ パスワードの保存や管理機能
❌ ネットワークセキュリティや送信保護
ブラウザ互換性

必要要件:

    ES6+ JavaScript対応
    Web Crypto API（window.crypto.getRandomValues）
    CSS GridおよびFlexbox対応

動作確認済み:

    Chrome 90+、Firefox 88+、Safari 14+、Edge 90+

パフォーマンス

    最新ハードウェアで60個のパスワードを100ms未満で生成
    初回読み込み後のネットワークリクエストはゼロ
    メモリフットプリント最小（合計1MB未満）
    典型的な制約充足は10回未満の再試行で完了

免責事項

⚠️ 重要: 無保証および責任の制限

本ソフトウェアは、明示的または黙示的を問わず、いかなる種類の保証もなく**「現状のまま」**提供されます。

利用者の責任:

    パスワード評価: 生成されたパスワードがセキュリティ要件を満たすかどうかの判断は、利用者の単独責任です
    リスクの引き受け: パスワードの使用、保存、管理に関連するすべてのリスクは利用者が負います
    ポリシー準拠: パスワードが組織のセキュリティポリシーに準拠するかは、利用者自身が検証する必要があります
    セキュリティ評価: 作成者は、特定の攻撃手法に対するパスワード強度について一切の主張を行いません

責任の制限: 作成者は、以下に起因するいかなる損害、損失、またはセキュリティ侵害に対しても責任を負いません：

    本ツールで生成されたパスワードの使用
    本リポジトリからコピーまたは改変されたコードの使用
    本ツールの改変版のデプロイ
    直接的、間接的、偶発的、または結果的な損害

本ツールまたはコードを使用することにより、これらの条件を認識し、受け入れたものとみなされます。
ライセンス

MIT License

Copyright (c) 2026 akegoromo

詳細は LICENSE ファイルを参照してください。

