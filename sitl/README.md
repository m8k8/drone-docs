# 第１弾（B〜D）のSITL

現在は第１弾の部品表に基づく**12.3387kgの暫定モデル**を使います。部品の仮配置から重心・慣性を計算し、ArduCopterによる飛行制御・AUTO航路・下向き測距を試験します。

| 資料 | 内容 |
| --- | --- |
| [実験003の結果](experiments/003-prototype-bd/README.md) | 基本飛行・AUTO・測距を各3回、重量感度を2回。2026-11-16の本試験は11/11成功 |
| [モデル説明](experiments/003-prototype-bd/model.md) | 質量・配置・推進校正・センサー・再現していない機能 |
| [実行記録](experiments/003-prototype-bd/run-record.md) | 本試験の版・ハッシュと、整理後の自動試験3/3成功・対話起動確認 |
| [実行ガイド](guides/execution.md) | セットアップと第１弾モデルの実行 |
| [可視化ガイド](guides/visualization.md) | 保存したCSVから軌跡・高度・測距応答の図を作る |

## 使用する設定

実行リポジトリの`config/prototype-01/`にまとめています。通常は`models/nominal.json`を使います。`mass-low.json`と`mass-high.json`は、同じ第１弾の質量を±0.5kg変える感度比較用です。別の搭載構成や散布機体ではありません。

準備済みの`drone/sitl`で、次を実行できます。

```bash
./scripts/prepare-prototype.sh
./scripts/prototype-test.sh --scenario flight
```

全11件の自動試験は次のコマンドです。

```bash
./scripts/prototype-suite.sh --repeats 3
```

MAVProxyで対話操作する場合も、`./scripts/run.sh`で同じ第１弾の基準モデルを起動します。手動で離陸・LOITER・RTLを操作する手順は[実行ガイド](guides/execution.md)を参照してください。

自動試験は毎回、新しい`runs/prototype-*`へ結果を保存します。`--output`で保存先を指定した場合も、既存ディレクトリへの上書きを拒否します。対話起動は固定の`runs/prototype-manual/`を使い、保存パラメータを継続します。起動ごとのログは、その中の`sessions/`へ分けて保存します。

## 記録の読み方

「成功」は、記録したモデルと判定条件を満たしたという意味です。重量・取付位置には仮定があり、FPV映像・G12の無線伝搬・Remote ID送信・実配線の電気特性などは再現していません。部品重量の反映と、その製品の全機能の再現を区別してください。

以前の別モデルの実験ページと起動手順は削除しました。実験番号003は、本試験のファイル名・ハッシュ・履歴との対応を保つためそのまま使います。

[資料の入口へ戻る](../README.md) · [実行コード：drone-sitl](https://github.com/m8k8/drone-sitl)
