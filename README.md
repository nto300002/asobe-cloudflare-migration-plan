# ChatGPT Sites Cloudflare 移行計画

原資料: `cloudflare_migration_plan_v1.1.docx`（2026年11月26日）。この README は原資料の計画を保持し、同日に実施した GitHub ソース登録を反映した更新版です。Cloudflare への移行と本番データの回収は未実施です。

**更新日: 2026年11月26日。状態: ソースの GitHub 登録済み、移行は計画段階。**

原資料では未登録サイトを Private リポジトリにする案でしたが、所有者の後続指示に従い Public で公開しています。原資料中の課金条件・無料枠・日程は計画時点の前提であり、契約と実測で再確認します。

## 1. 目的と基本方針

2027年2月1日からのSites課金変更を契機に、業務アプリのホスティング・永続データをCloudflareへ移行する。5サイト合計の月額予算は5米ドル以下、目標は0米ドル。Google認証を採用し、既存業務・UI・データの継続性を優先する。移行完了目標は2027年1月24日、最終期限は1月31日とする。課金開始日・適用条件は前回計画とユーザー提示を前提とし、契約上の最新条件は切替前に再確認する。

## 2. 移行対象と優先順位

| 対象 | 優先度 | 2026年11月26日時点のソース管理 |
| --- | --- | --- |
| asobe在庫管理 | S | Public リポジトリへローカルソースのスナップショットを登録 |
| プリントオーダー | S | 既存 Public リポジトリを確認。現行配備との差分は未照合 |
| プリントマニュアル | S | Public リポジトリへ登録。埋込写真は除外 |
| 封入・納品管理 | 要確認 | Public リポジトリへ登録。移行の要否は利用状況で判断 |
| KIIYA見落としチェッカー | B・独立 | 既存 Public リポジトリの `main` にローカルソースを追加。現行配備との差分は未照合 |

## 3. 分離方針（今回の改訂）

asobeの最大4サイトは共通認証・共通ポータルを持つ。KIIYAは別Worker、別URL、別OAuthクライアント、別DB・R2・セッション、別GitHubリポジトリとする。相互のデータ参照やログイン共有は行わない。同一Cloudflareアカウント内で論理分離する案を基本とし、組織の所有権が異なる場合はアカウント自体も分離する。

現状の本番データ、全機能、画像原本、利用人数は未確認。ソースの GitHub 登録は実施したが、本番変更と Cloudflare への移行は実施していない。

## 4. 現行サイトと GitHub の対応

| 対象 | Sites URL | GitHub URL | ソース登録状況 |
| --- | --- | --- | --- |
| 在庫管理 | [silk-screen-inventory](https://silk-screen-inventory.hherocl278.chatgpt.site/session?return_to=%2F) | [asobe-inventory-site](https://github.com/nto300002/asobe-inventory-site) | 2026-10-16 のローカルコミットを登録 |
| プリントオーダー | [asobe-print-order-20261012](https://asobe-print-order-20261012.hherocl278.chatgpt.site/) | [asobe-print-order-20261012](https://github.com/nto300002/asobe-print-order-20261012) | 既存リポジトリを確認 |
| プリントマニュアル | [asobe-silk-screen-skill-check](https://asobe-silk-screen-skill-check.hherocl278.chatgpt.site/) | [asobe-silk-screen-manual-site](https://github.com/nto300002/asobe-silk-screen-manual-site) | 2026-11-25 のローカルコミットを登録。埋込写真と確認用画像を除外 |
| 封入・納品管理 | [funnou-nouhin-kanri](https://funnou-nouhin-kanri.hherocl278.chatgpt.site/) | [asobe-packing-delivery-site](https://github.com/nto300002/asobe-packing-delivery-site) | 2026-11-12 のローカルコミットを登録 |
| KIIYA見落としチェッカー | [kiiya-cleaning-checker](https://kiiya-cleaning-checker.hherocl278.chatgpt.site/) | [kiiya-cleaning-checker-site](https://github.com/nto300002/kiiya-cleaning-checker-site) | 2026-11-04 のローカルコミットを `sites-source/` に追加し、[PR #16](https://github.com/nto300002/kiiya-cleaning-checker-site/pull/16) を `main` に統合。大型フォントを除外 |

上記5つの GitHub リポジトリは Public。Sites URL は原資料の記載を引き継ぎ、2026年11月26日に各画面へアクセスできることを確認した。GitHub のソースと現在の配備版が同一かは未検証。GitHub のソースには本番 DB、画像原本、秘密鍵、OAuth secret、Chatwork トークンを含めない。

### 管理者ログインとレコードダウンロードの調査

2026年11月26日、依頼者の ChatGPT アカウントで5つの Sites URL を開き、サインイン後の画面を確認した。業務データの追加・変更・削除やダウンロードは行っていない。確認した管理画面には、業務レコード全件をファイルとして取得する操作は見当たらなかった。画面にある PDF 化や個別報告の出力は、DB・画像を含む移行用エクスポートとは別の機能として扱う。

| サイト | ログイン・管理画面の確認結果 | レコード保管先とエクスポート方針 |
| --- | --- | --- |
| 在庫管理 | サインイン成功。画面に「管理者」と表示され、職員台帳・操作記録を開けた | D1 の品目、日別在庫、購入依頼を ID・日時・参照関係ごと JSON/CSV へ。公開済みソースは職員台帳がある配備版より古いため、実装前に現行ソースを回収・照合 |
| プリントオーダー | サインイン成功。画面に「管理」と表示され、職員台帳・操作記録を開けた | D1 の `app_state` にある版番号と業務状態の完全な JSON を優先し、注文・制作実績の CSV を補助出力。現行配備との差分を照合 |
| プリントマニュアル | サインイン成功。設定とアカウント管理画面を開けた。編集には登録済みパスキーでの追加確認が必要 | D1 のマニュアル本文、動画情報、利用者進捗などを JSON/CSV へ。R2 と元ソースに埋め込まれた写真の原本を一覧・MIME・サイズ・SHA-256 とともに別ファイルとして回収。ダウンロードにパスキーを要求するか権限設計時に決める |
| 封入・納品管理 | サインイン成功。職員画面で依頼者アカウントの役割が「管理」、有効と表示された | D1 の番号、工程別残数、納品記録、明細を ID・日時・参照関係ごと JSON/CSV へ。利用継続の判断と独立にバックアップを用意 |
| KIIYA 見落としチェッカー | サインイン成功。職員台帳で依頼者アカウントの役割が「アプリ管理者」と表示された | D1 の作業、報告、画像メタデータ、マスタ履歴を JSON/CSV へ。R2 の画像原本も別途回収。公開済みソースは 2026-11-04 のローカル版なので、現行配備との差分を先に照合 |

### 全件ダウンロード機能の確定仕様

**目的と範囲:** 管理者が各サイトの管理画面から、既存の業務レコードを1件も省かずに取得し、同じ時点のテーブル一覧と構造を確認できるようにする。画面上の検索・日付・表示件数による絞り込みは全件ダウンロードに適用しない。対象はエクスポート作成時の配備先 D1 を調べ、GitHub のスキーマだけを正本としない。CSV は閲覧用の補助形式であり、復元用の原本は型と null を保つ JSON Lines または SQL とする。

**管理画面の操作:** 「業務データ全件とテーブル構造をダウンロード」を押すと、対象 DB、含まれる業務テーブルと件数見込み、画像の有無、秘密情報の扱いを表示する。管理者が開始するとジョブ ID と状態（準備中・完成・失敗）を表示し、完成後だけ「ダウンロード」を有効にする。生成中の再操作は既存ジョブを案内し、失敗時は不完全なファイルをダウンロードさせない。プリントマニュアルでは既存のパスキーによる追加確認をダウンロードにも適用する。その他のサイトもサーバー側でその時点の管理者権限を再検査し、権限不足は 403、未ログインは 401 とする。開始とダウンロードの両方を監査ログに記録する。

**出力物:** サイトごとに次の構成の ZIP を作る。ファイル名にはサイト名と UTC の生成時刻を含め、ZIP 内は UTF-8 とする。複数のアーカイブに分割した場合も共通のエクスポート ID を付ける。

```text
<site>-<UTC timestamp>.zip
  manifest.json
  schema/schema.sql
  schema/tables.json
  data/<table>.jsonl
  assets/manifest.json             # 画像を持つサイトのみ
  assets/<object key>              # 画像を持つサイトのみ、分割可
```

| ファイル | 必須内容 |
| --- | --- |
| `schema/schema.sql` | 配備中 D1 から取得したテーブル、ビュー、インデックス、トリガーの `CREATE` 定義。順序と依存関係を保ち、復元に不要な内部オブジェクトの扱いを明記 |
| `schema/tables.json` | **全ユーザーテーブル**の名前、列名、型、NULL 可否、既定値、主キー順、生成列、外部キー、ユニーク制約、インデックス、トリガー、現在件数。値を出さない技術・認証テーブルも一覧と構造には含め、値の除外理由を記録 |
| `data/<table>.jsonl` | 対象業務テーブルの**全行**。1行1レコードで列名と元の値を保持し、ID・日時・削除状態・外部キーを変換しない。BLOB があれば base64 と型を明記。0件のテーブルにも空ファイルを作る |
| `assets/manifest.json` | R2 とソース埋込写真の対象一覧、参照元レコード・キー・MIME・サイズ・SHA-256。ファイルが欠けた場合は完了扱いにしない |
| `manifest.json` | エクスポート ID、サイト URL、生成日時、ソース版、DB 識別子、スキーマのハッシュ、対象・除外テーブル、各テーブル件数、各ファイルのサイズと SHA-256、画像件数、整合性検査結果、完全／失敗の状態。秘密値は記録しない |

テーブル定義は D1 の `sqlite_schema` と対応する `PRAGMA table_xinfo`、`foreign_key_list`、`index_list`、`index_xinfo` 等で照合する。D1 がサポートする SQL と `PRAGMA` の範囲は[公式 SQL 文書](https://developers.cloudflare.com/d1/sql-api/sql-statements/)に従い、実装時に各配備 DB で確認する。運用者による別経路の検証・復元用には、[Wrangler の `d1 export --remote`](https://developers.cloudflare.com/d1/best-practices/import-export-data/)でスキーマとデータの SQL ダンプを取得できる。管理画面に Cloudflare API トークンを渡したり、Public GitHub に出力物を保存したりしない。

**サイト別の業務データ:** 在庫管理は `items`、`daily_inventory`、`shortage_requests` を最低限含める。プリントオーダーは `app_state` の版番号と JSON 全体を原本とし、その中の注文・制作実績等を CSV に展開できるようにする。プリントマニュアルは `users`、`manual_documents`、`manual_videos`、`manual_video_preview_checks`、`progress`、`progress_archives` と写真原本を含める。封入・納品管理は `variants`、`balances`、`deliveries`、`delivery_lines` を含める。KIIYA は `app_users`、`works`、`work_changes`、`images`、`report_versions`、マスタ変更履歴、報告配送履歴と画像原本を含める。これは現時点の公開ソースから得た最低一覧であり、配備 DB に追加の業務テーブルがあれば自動検出して対象へ加える。

**機密テーブル:** セッション、パスキー、認証チャレンジ、OAuth トークン、Chatwork 認証設定などの値は通常の業務ダウンロードに入れない。ただし、そのテーブルの名前・列・制約・件数と除外理由は `schema/tables.json` と `manifest.json` に必ず残す。認証状態を含めた完全な DB 復旧が必要な場合は、管理画面の通常ダウンロードとは別に、運用者が全 DB の SQL ダンプを取得し、暗号化して保管する。業務上必要な職員・利用者の対応関係は、秘密値を除いた移行用の表として定義し、管理者だけが取得する。

**整合性と完了条件:** 全件取得中の更新で行が欠けたり重複したりしないよう、単一の整合した時点のダンプを使うか、更新を停止して取得する。ページ分割で取得する場合は開始・終了時の版と件数を比較し、変化があれば失敗として再作成する。各テーブルの `COUNT(*)` と出力行数、全ファイルの SHA-256、外部キー参照、画像件数を照合する。空の検証 DB にスキーマとデータを復元し、元の件数・ID・日時・参照関係が一致して初めて受入合格とする。画像が大きいマニュアルと KIIYA は非公開の一時保管と分割ダウンロードを使い、期限切れ後に一時ファイルを消す。実装前に配備版ソース、D1・R2 の実体と容量を確認する。

**受け入れケース:** 管理者が全件を取得し、全業務テーブルの行数・構造・ハッシュが一致すること。0件のテーブルも構造と空ファイルがあること。閲覧・編集権限の利用者と未ログイン利用者は API を直接呼んでも取得できないこと。生成中の更新、R2 の欠損、期限切れダウンロードは成功と表示されないこと。復元先で ID・日時・外部キーと画像参照が一致すること。各ケースについて操作・HTTP 応答・マニフェスト照合結果を証跡として残す。

## 5. 移行先アーキテクチャ

| 項目 | asobe | KIIYA |
| --- | --- | --- |
| ホスティング | asobe-tools Worker（仮） | kiiya Worker（仮） |
| URL | workers.dev／アプリ別パス | 独立workers.dev URL |
| 認証 | Google OIDC 共通認証 | 独立Google OIDC |
| DB | auth + 4業務DB | 専用auth + 業務DB |
| 画像 | asobe専用R2 | KIIYA専用R2（必要時） |
| Git | 既存注文repo＋新規repo等 | 既存kiiya-site repo |
| デプロイ | 独立CI／Wrangler | 独立CI／Wrangler |

D1は合計7 DBを想定（封入を除けば6）。同一アカウントの無料枠10 DBの残りを検証用に使う。asobeのパスは /inventory、/print-order、/manual、/delivery、/admin を暫定案とする。既存アプリの独立デプロイが必要ならasobe内でWorkerを分け、共通認証の安全な連携を別途設計する。

### 技術選定

プリントオーダーは既存Vinext構成を優先。Sites専用の認証ヘッダー、.openai/hosting.json、ビルド・環境変数依存を調査し、Cloudflare本番用設定とD1/R2 bindingを整備する。pnpm lockfileを尊重し、Node.js 22.13以上でビルド検証する。その他は回収コードに合わせ、必要な場合のみReact/TypeScript/Vite/Hono/Drizzleを採用する。全面書き換えやモノレポ統合は必須にしない。

### Google認証・権限

Authorization Code Flow + PKCE、state、nonce、署名・iss・aud・exp検証を行う。Google subを永続識別子とし、既存SitesユーザーIDとは管理者承認の対応表で紐付ける。許可済みユーザーのみアクセス可能とし、管理者・職員・作業担当者の権限はAPI側で検査する。CookieはSecure/HttpOnly/SameSite、CSRF対策、退職時のセッション失効を実装。R2は非公開とし認証済みAPIで配信する。

### KIIYA固有の検証

既存設計にChatwork連携、毎分監視・遅延照合・日次処理の候補がある。現行サイトの実装有無を確認し、必要ならWorkers Cron等で再現性・重複送信防止・実行時間・費用を検証する。外部Cloud Tasks等は費用未確定のため採用保留。

## 6. 費用計画・上限制御

Workers Freeを基本とし、有料化は承認制とする。以下は前回計画で参照した公式仕様に基づく設計値で、契約時点に再確認する。同一アカウント内の全アプリで枠を共有する。

| 項目 | 無料枠（設計値） | 警戒ライン |
| --- | --- | --- |
| Workers | 10万動的要求/日、CPU 10ms/要求 | 7万要求/日 |
| 静的配信 | 無料・無制限（該当配信） | 動的処理と区別 |
| D1読込 | 500万行/日 | 350万行/日 |
| D1書込 | 10万行/日 | 7万行/日 |
| D1容量 | 合計5GB、個別500MB、10 DB | 個別350MB |
| R2 Standard | 10GB-month、Class A 100万/月、B 1000万/月 | 7GB、操作枠70% |

試算：15人×50操作/日×3 API = 2,250要求/日、各API100行読込なら225,000行/日、操作あたり2行書込なら1,500行/日。画像2GBと仮定すれば上記主要枠内。ただし認証・定期処理・バックアップ・CPUは別途実測する。

R2は無料枠超過で従量課金される。保存量・アップロードサイズ・操作回数をアプリ側で制御し、上限接近時は管理者通知と不要な書込み停止を行う。予算通知は強制課金停止を保証しない。Workers Paidは基本5ドル/月に超過料金・税が加わる可能性があるため、総額5ドルの厳守には不向き。独自ドメインは購入しない前提。

### 運用と所有権

本番アカウントは事業所の管理権限を確保する。GitHubとCloudflareの最小権限を設定し、KIIYAのデプロイトークンはasobeの資源を変更できないようにする。組織が別でアカウントも分離する場合、費用は両アカウントの合計で管理する。

## 7. データ・ソースコード回収

優先順：①現行ソースとエクスポートの取得、②管理者限定のエクスポート機能追加、③取得不可の場合は画面・仕様から再実装。現行本番への変更はバックアップと承認の後に限る。

| 回収対象 | 確認・保全内容 |
| --- | --- |
| コード | フロント、API、依存・lockfile、設定、テスト、スキーマ |
| DB | 全レコード、ID、日時、外部キー、件数、容量、削除状態 |
| ファイル | 写真原本、MIME、サイズ、SHA-256、参照先 |
| 認証 | 既存ID、権限、所有者対応表（秘密値除外） |
| 仕様 | 全画面、操作、エラー、外部連携、スクリーンショット |

### プリントオーダー

既存 Public リポジトリの存在を確認した。次に現在の Sites デプロイと差分を照合し、移行用ブランチで既存コード・履歴を維持する。DBスキーマと本番データは別物として回収する。

### KIIYA：既存リモートへのソース追記

既存 README/docs/issues を保持したまま、ローカルの `kiiya-ui-release` コミットのスナップショットを `sites-source/` に追加し、PR #16 で `main` に統合した。現在配備中の Sites コードと同一かは未照合。大型フォントは未登録で、そのファイルを読むテストは復元まで実行できない。秘密・業務データの除外と、最新配備コードとの差分確認を続ける。

### 原資料で GitHub 未登録だった3サイト

在庫・マニュアル・封入のローカルソースを、それぞれ新しい Public リポジトリに登録した。登録したのは既知のローカルコミットのスナップショットであり、現行配備との差分は未照合。マニュアルの埋込写真と確認用画像は公開対象から除外したため、写真原本の所在確認と保全を優先する。

### Sites作成元チャットへの調査指示

現行アプリを変更せず、ソースコードの取得方法、DBの種類・スキーマ・件数・容量、全データのエクスポート可否、画像一覧・原本取得方法、認証方式、全画面・API、Sites依存箇所を調査する。取得できない情報は推測せず理由を記載する。削除・再デプロイは禁止。

## 8. スケジュールと完了判定

| 期間 | 作業 | 完了条件 |
| --- | --- | --- |
| 11/26–12/2 | 現行調査・対象確定 | ソース・データ所在を特定 |
| 11/29–12/13 | 全サイトデータ回収 | 復元可能な退避データ |
| 12/1–12/15 | asobe・KIIYA基盤 | 各Google認証・DBが動作 |
| 12/11–12/20 | KIIYA先行検証 | 独立環境で動作・連携確認 |
| 12/18–12/31 | 在庫管理 | 在庫整合性確認 |
| 12/24–1/7 | プリントオーダー | 既存機能と受注記録一致 |
| 12/29–1/10 | マニュアル | 本文・画像照合 |
| 1/6–1/13 | 封入・納品 | 必要時のみ移行 |
| 1/10–1/20 | 統合・受入テスト | 必須Gate合格 |
| 1/20–1/24 | 本番切替 | 利用開始・旧更新停止 |
| 1/24–1/31 | 予備期間 | 不具合対応・旧課金確認 |

工数は前回の43–70時間（コード再利用）、70–120時間（再実装）を暫定参考値とする。KIIYAの独立認証・定期処理等の追加工数は未算定。回収後に再見積もりし、期限に間に合わない場合は優先度Sの3サイトを保護する。

### 必須受入Gate

移行前後の件数と項目値・ID・日時・参照関係を照合。画像SHA-256と全表示を確認。ログイン、未許可ユーザー拒否、CRUD、再読込、別端末、同時更新、権限、asobe/KIIYA間のアクセス拒否を実機検証する。Playwrightのスクリーンショット、HTTP応答、DB比較JSON、テストログを保存。KIIYAの外部連携があれば重複・失敗復旧も検証する。AIの完了報告だけでは合格にしない。

## 9. 切替・復旧・未決事項

### 本番切替

初回移行 → 試験運用 → 旧サイト更新停止 → 最終差分取得・取込 → 件数・内容・画像照合 → 新URL周知 → 新環境での業務開始 → 旧Sites停止・課金状態確認。切替失敗時は新環境の更新分を退避・照合して旧環境へ復帰する。旧環境のデータ削除は復元可能性と管理者承認を確認した後に限る。

### バックアップ

D1は毎日SQLエクスポート、R2は毎週・変更時に原本バックアップ、コード・スキーマは変更ごとにGit管理。暗号化した外部ストレージに週次保管し、月1回空の検証DBへ復元する。D1 Time Travel（無料7日、有料30日）は補助として利用する。

例：npx wrangler d1 export inventory-db --remote --output=./backups/inventory.sql

例：npx wrangler d1 execute inventory-restore-test --remote --file=./backups/inventory.sql

未決事項・確認責任

| 事項 | 確定条件 |
| --- | --- |
| コード完全性 | 現行配備とGitの差分照合 |
| 写真原本 | 手元／Sites／その他の所在確認 |
| 封入管理 | 利用者への確認で継続判断 |
| 本番データ・権限 | 事業所承認、容量・ユーザー・権限実測 |
| KIIYA分離範囲 | Cloudflareアカウント自体の分離要否 |
| 予算 | 税・外部API・バックアップ費用の扱い |
| 課金条件 | OpenAI最新の契約・停止条件確認 |

## 10. 参考資料

https://github.com/nto300002/asobe-inventory-site  
https://github.com/nto300002/asobe-print-order-20261012  
https://github.com/nto300002/asobe-silk-screen-manual-site  
https://github.com/nto300002/asobe-packing-delivery-site  
https://github.com/nto300002/kiiya-cleaning-checker-site  
https://developers.cloudflare.com/workers/platform/pricing/  
https://developers.cloudflare.com/d1/platform/limits/  
https://developers.cloudflare.com/d1/platform/pricing/  
https://developers.cloudflare.com/r2/pricing/  
https://developers.cloudflare.com/d1/reference/time-travel/  
https://developers.google.com/identity/openid-connect/openid-connect  
https://learn.chatgpt.com/docs/sites/usage-and-cost
