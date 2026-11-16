# 農業ドローンの設計・シミュレーション資料

実機を購入する前に、ArduPilot SITLで制御を学ぶための手順・検証記録をまとめています。

入口は[SITLの実験一覧](sitl/README.md)です。目的や機体モデルが変わるたびに、実験用のディレクトリを作って条件・結果をまとめます。

| 読みたいこと | 資料 |
| --- | --- |
| 実験の目的、現在の検証範囲、結果 | [SITLの実験一覧](sitl/README.md) |
| 環境構築、標準機の起動、最初の飛行、終了 | [実行ガイド](sitl/guides/execution.md) |
| 地図、姿勢、高度、テレメトリーを見る | [可視化ガイド](sitl/guides/visualization.md) |

## リポジトリの役割

- [drone-docs](https://github.com/m8k8/drone-docs)：この資料リポジトリ。
- [drone-sitl](https://github.com/m8k8/drone-sitl)：セットアップ・起動・自動テストのコード。

手元では同じ親ディレクトリに配置します。

```text
drone/
├── docs/   # drone-docs
└── sitl/   # drone-sitl
```

SITLの資料は、共通の手順書と実験記録に分けて保存します。

```text
docs/
├── README.md
└── sitl/
    ├── README.md
    ├── guides/
    │   ├── execution.md
    │   └── visualization.md
    └── experiments/
        ├── 001-standard-quad/
        │   └── README.md
        └── 002-g06-payload/
            └── README.md
```

実施日・モデル・判定条件・結果・未検証項目は各実験の記録を参照してください。QGroundControlの共通ガイドは公式資料を調べて記載しており、GUIでの接続・表示は未検証です。

## 更新するとき

実験の目的やモデルを変えた場合は、`sitl/experiments/`の下に番号と短い名前を付けたディレクトリを作ります。使ったコードの版、実行環境、試験条件、測定値、未検証項目を記録します。同じ実験を繰り返す場合も実施日を残し、失敗した試験を成功した記録で上書きしません。

このリポジトリは手順書と公開用に整理した実験記録を管理します。手元にある購入計画Excelや旧結果JSONはGit管理対象外です。依存環境や生の実行ログは、実行コード側のGit管理対象外ディレクトリに保存します。
