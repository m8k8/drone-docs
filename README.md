# 農業ドローンの設計・シミュレーション資料

第１弾（段階B〜D）の部品選定と、同じ構成を使うArduPilot SITLの手順・検証記録をまとめています。現在のシミュレーションは、電池込み12.3387kgの暫定モデルに統一しています。

| 読みたいこと | 資料 |
| --- | --- |
| 搭載部品、選定理由、重量 | [第１弾の部品表](hardware/prototype-01/README.md) |
| 現在のモデルと実行方法の入口 | [第１弾のSITL](sitl/README.md) |
| 基本飛行・AUTO航路・下向き測距の試験結果 | [実験003](sitl/experiments/003-prototype-bd/README.md) |
| 環境構築と試験の実行 | [実行ガイド](sitl/guides/execution.md) |
| 飛行軌跡・高度・測距応答を見る | [可視化ガイド](sitl/guides/visualization.md) |

## リポジトリの役割

- [drone-docs](https://github.com/m8k8/drone-docs)：部品表・手順書・公開用の実験記録。
- [drone-sitl](https://github.com/m8k8/drone-sitl)：モデル・セットアップ・起動・自動試験のコード。

手元では同じ親ディレクトリに配置します。

```text
drone/
├── docs/   # drone-docs
└── sitl/   # drone-sitl
```

資料の構成は次のとおりです。

```text
docs/
├── hardware/prototype-01/  部品表・選定根拠
└── sitl/
    ├── README.md
    ├── guides/
    │   ├── execution.md
    │   └── visualization.md
    └── experiments/003-prototype-bd/
        ├── README.md       条件・結果・再実行方法
        ├── model.md        モデルの仮定と再現範囲
        ├── run-record.md   実行版・ハッシュ
        └── figures/        軌跡・測距応答の図
```

2026-11-16の整理で、他のシミュレーション用コード・実験ページ・生ログを削除し、第１弾B〜Dの設定と本試験結果を残しました。過去のGitコミットは書き換えていません。

## 更新するとき

機体の部品・重量・配置・制御条件を変更した場合は、モデルの仮定と結果を一緒に更新します。再試験は新しい保存先を使い、既存の本試験結果へ上書きしません。現時点の11/11成功は記録したモデルと条件に対する結果で、実機の飛行性能や全搭載機器の動作を保証するものではありません。

購入計画Excel、依存環境、生の実行ログはGit管理対象外です。部品表の内容や元のExcelは、今回のシミュレーション整理では変更していません。
