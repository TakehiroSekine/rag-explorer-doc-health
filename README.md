# rag-explorer-doc-health

RAG に登録する前の文書の集まりを読み、誤答の原因になりうる**文書の側の問題**を拾う CLI です。
LLM ・ 埋め込みを使わず、実行中は通信しません（入手のときを除く。下の「通信について」）。

## 使い方

```
npx rag-explorer-doc-health --dir <入力のフォルダ> --out <出力のフォルダ> [--no-excerpt]
```

- 入力のフォルダの下の file（下のフォルダも含む）を読みます。入力のフォルダは読むだけで、書き換えません。
- 読む形式: Markdown ・ テキスト ・ JSON ・ Word（DOCX）・ PowerPoint（PPTX）・ Excel（XLSX）・ PDF。旧形式の Word（DOC）は読みません。
  - それ以外の形式は読まずに、読まなかったことを出力に記録します。
  - PDF のうち、スキャンした PDF ・ 文字が化けている PDF ・ 画像の多い PDF は診断しません（出力に理由を記録します）。

## 出力

`--out` のフォルダに 3 つの file を書きます。

| file | 中身 |
|---|---|
| `report.html` | 人の読むレポート（日本語 ・ 1 file ・ 外部の資源を読まないので、オフラインで開けます） |
| `manifest.json` | 読んだ文書の一覧と、診断しなかった文書とその理由 |
| `findings.json` | 所見の一覧（文書と行の範囲。本文は含みません） |

- `report.html` は、既定で所見の箇所の本文の抜粋を含みます。`--no-excerpt` を付けると、本文を 1 文字も出しません。
- 同じ入力には同じ内容の出力を返します（時刻を含みません）。

## 通信について

- **入手のとき**: `npx rag-explorer-doc-health` は、初回に npm の registry からこの package を取得します（ここで通信します）。
- **実行中**: 通信しません。起動の最初に、通信の口（`net` ・ `tls` ・ `dns` ・ `http` ・ `https` ・ `http2` ・ `dgram` ・ `fetch` ・ `WebSocket`）と、
  外の process ・ thread を作る口（`child_process` ・ `worker_threads`）を塞ぎます。呼ばれたら、その場で止まります。
- 入手のときの通信も避けたい場合は、別の環境で入手した package の tarball（`npm pack rag-explorer-doc-health` で作れます）を持ち込み、
  `npx --offline --package=<tarball> rag-explorer-doc-health --dir <入力のフォルダ> --out <出力のフォルダ>` で動かしてください。
  Windows なら、下の「Windows 版」も使えます（Node も npm も要りません）。

## Windows 版

Node の入っていない Windows で動く単体の実行ファイルを、GitHub の Releases に zip で置いています（`rag-explorer-doc-health-<版>-win-x64.zip`）。
展開すると、実行ファイル（`rag-explorer-doc-health-<版>-win-x64.exe`）と、この README ・ `LICENSE` ・ `THIRD_PARTY_LICENSES.txt` が出てきます。

```
rag-explorer-doc-health-<版>-win-x64.exe --dir <入力のフォルダ> --out <出力のフォルダ> [--no-excerpt]
```

- 中身は npm の版と同じ CLI です（Node の単体の実行ファイルの仕組みで、Node と一緒に 1 file にしています）。
- **署名していません。** そのため、次の 2 か所で警告が出ることがあります。
  - **入手のとき**: ブラウザが、zip を「一般的にはダウンロードされない」ファイルなどとして警告することがあります（表示の語はブラウザと版で変わります）。
    下の sha256 を確かめた上で、ファイルを保持してください。
  - **初めて実行するとき**: Windows の SmartScreen が「Windows によって PC が保護されました」と警告することがあります。
    「詳細情報」を押すと「発行元: 不明な発行元」と「実行」のボタンが出ます。下の sha256 を確かめた上で、「実行」で進めてください。
- 入手した file が配った物と同じかは、Releases の `SHA256SUMS.txt` の値と見比べて確かめてください。zip と、展開した実行ファイルの両方の値が載っています（PowerShell）:

  ```
  Get-FileHash .\rag-explorer-doc-health-<版>-win-x64.zip -Algorithm SHA256
  Get-FileHash .\rag-explorer-doc-health-<版>-win-x64.exe -Algorithm SHA256
  ```

- Windows PowerShell 5.1 で、画面の表示を file にリダイレクト（`>`）したり、パイプで渡したりすると、日本語が化けます
  （PowerShell が外部のコマンドの出力を Shift_JIS として読むため）。先に次を実行してください。出力の 3 file には影響しません。

  ```
  [Console]::OutputEncoding = [Text.Encoding]::UTF8
  ```

## 処理の時間

- Markdown ・ テキストの文書は、数百の file でも数秒で終わります（試した例: 172 file で 1 秒未満）。
- PDF は 1 本あたり数秒 ・ 数百 MB のメモリを使うことがあります（試した例: 約 90 頁の PDF 2 本で 9 秒 ・ 約 600 MB）。PDF の多いフォルダは、最初は数本で試してください。

## 試した文書

公開の前に、性質の違う 3 組の公開文書で、誤検出の傾向を確かめました（文書の本文や抜粋はどこにも載せていません）。

| 文書 | 型 | 出典と条件 | 加工 |
|---|---|---|---|
| 厚生労働省「モデル就業規則」の 2 つの版（令和 7 年 12 月版 ・ 令和 5 年 7 月版） | 規程集（PDF） | 厚生労働省（https://www.mhlw.go.jp/stf/seisakunitsuite/bunya/koyou_roudou/roudoukijun/zigyonushi/model/index.html）。政府標準利用規約に準拠 | ファイル名だけを変えました |
| 国税庁「タックスアンサー」の所得控除の節の 23 頁 | FAQ | 国税庁（https://www.nta.go.jp/taxes/shiraberu/taxanswer/code/index.htm）。公共データ利用規約（第 1.0 版）に準拠 | HTML の本文を Markdown に変換しました |
| Kubernetes の日本語ドキュメント（`content/ja/docs/concepts/`） | 技術文書（Markdown） | The Kubernetes Authors（https://github.com/kubernetes/website）。CC BY 4.0 | 変更していません |

- 識別子（型番 ・ エラーコード）の多い製品マニュアルの型では、試していません。条件の合う公開の文書が見つからなかったためです。

## 対応する Node の版

保守の続く LTS に限ります: **Node 22 ・ Node 24**。

## 問い合わせ

support@mind-f.com

## license

MIT（`LICENSE`）。束ねてある第三者の package の license は `THIRD_PARTY_LICENSES.txt` にあります。
