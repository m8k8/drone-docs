# 第１弾（B〜D）のSITLを実行する

Mac上で12.3387kgの第１弾モデルを動かし、実機なしで基本飛行・AUTO航路・下向き測距を試験する手順です。設定と再現範囲は[モデル説明](../experiments/003-prototype-bd/model.md)、既存の11回の結果は[実験003](../experiments/003-prototype-bd/README.md)を参照してください。

## 1. 必要な環境

| 項目 | 使用するもの |
| --- | --- |
| OS・CPU | macOS / Apple Silicon |
| 開発ツール | Xcode Command Line Tools、または開発ツールを含むXcode |
| Python | Python 3.12、プロジェクト専用の`.venv` |
| 飛行制御 | ArduPilot Copter 4.7.2 |
| 固定コミット | `fb773567896014da007bca58358704d4a0d8516f` |
| 機体モデル | `config/prototype-01/models/nominal.json`、12.3387kg、X配置 |
| 開始地点 | CMACの平地。利用者の実際の飛行場所ではない |

初回はコードとPythonパッケージを取得するため、インターネット接続が必要です。以下のコマンドはmacOSのターミナルへ入力します。

## 2. 新しい環境を準備する

すでに`drone/sitl`と`drone/docs`がある場合はcloneを省略します。

```bash
mkdir -p drone
cd drone
git clone https://github.com/m8k8/drone-sitl.git sitl
git clone https://github.com/m8k8/drone-docs.git docs
cd sitl
```

開発ツールとPythonを確認します。

```bash
xcode-select -p
/opt/homebrew/opt/python@3.12/bin/python3.12 --version
```

不足している場合だけ、Command Line ToolsとHomebrewのPythonを導入します。Homebrew自体が未導入なら[公式手順](https://brew.sh/)を使ってください。

```bash
xcode-select --install
brew install python@3.12
```

`sitl`ディレクトリで実行します。

```bash
./scripts/setup.sh
./scripts/prepare-prototype.sh
```

`setup.sh`は固定版ArduPilotとサブモジュール、CLI用のPython依存を用意します。`prepare-prototype.sh`はモデルの生成整合性と推進校正用パッチを確認し、このMac用のSITLをビルドします。単なるビルドに加え、この準備処理まで済ませてから試験します。

実行ファイルは`vendor/ardupilot/build/sitl/bin/arducopter`です。シェル設定やグローバルPythonを変更せず、取得ソース・依存・ログをこのプロジェクト内に置きます。

## 3. 基本飛行を1回試す

準備済みの環境では、`drone`から次を実行します。

```bash
cd sitl
./scripts/prototype-test.sh --scenario flight
```

専用SITLを起動し、通常のアームチェック、10m離陸、LOITER、北20m移動、RTL、着陸・自動ディスアームを確認します。`PASSED`と終了コード0が成功です。途中の拒否・タイムアウト・観測欠落は成功扱いにしません。

その他のシナリオは次のとおりです。

```bash
./scripts/prototype-test.sh --scenario mission
./scripts/prototype-test.sh --scenario rangefinder
```

`mission`は高度10mで20m四方の4地点を回り、ミッション内のRTLで帰還します。`rangefinder`は正常測距、1m短い合成測距への応答、範囲外での高度維持、正常値への復帰を調べます。範囲外値の試験であり、UART通信断の再現ではありません。

## 4. 全11件の試験を実行する

```bash
./scripts/prototype-suite.sh --repeats 3
```

基準機の3シナリオを各3回、同じ推進系で質量を±0.5kg変えた基本飛行を各1回行います。実行は5倍速で、各回を新しい保存状態から始めます。

保存先を指定する場合は、未使用のディレクトリ名を使います。

```bash
./scripts/prototype-suite.sh --repeats 3 --output runs/prototype-next
```

保存済みの本試験は`runs/prototype-final-20261116/`です。上書きせず、新しい試験結果は別名で保存してください。自動試験はQGroundControlへ転送せず、[保存CSVの可視化](visualization.md)を使います。

## 5. MAVProxyで対話操作する

`run.sh`も同じ第１弾の基準モデルを起動します。自動試験とは別のインスタンス41を使います。

```bash
./scripts/run.sh
```

起動後はMAVProxyの入力画面です。`ArduPilot Ready`や機体の検出を確認し、GPS・EKFの準備を待ちます。以下はmacOSのシェルではなく、MAVProxyへ入力します。

```text
mode GUIDED
```

GUIDEDへの切替を確認してからアームします。

```text
arm throttle
```

`ARMED`の通知を確認してから、10秒以内に離陸を指示します。

```text
takeoff 10
```

上昇後に高度を確認します。`GLOBAL_POSITION_INT.relative_alt`の単位はmmで、約10000が相対高度10mです。

```text
status GLOBAL_POSITION_INT
```

高度が安定したらスロットルを中立にしてLOITERへ切り替えます。

```text
rc 3 1500
mode LOITER
```

帰還・着陸は次の指示です。接地して`DISARMED`の通知が出るまで待ちます。

```text
mode RTL
```

終了は`Ctrl+C`。対話用の保存先は`runs/prototype-manual/`です。通常起動では保存パラメータを継続し、設定を初期化する場合だけ、SITLを終了してから次を使います。

```bash
./scripts/run.sh --reset
```

`--reset`は保存パラメータの`eeprom.bin`を初期化する指定です。機体モデルは常に第１弾のnominalを使います。アームが拒否された場合はPreArmの理由を確認し、チェックを無効化せずに対処します。

対話起動は機体のTCP 6170と仮想RCのUDP 5911を使い、MAVProxyからQGroundControl向けUDP 14550へ転送します。機体ログ・通信ログは対話用ディレクトリ内に保存します。機体側の起動・終了処理は、この起動で作ったプロセスを対象とします。

## 6. 結果とログ

| 保存物 | 内容 |
| --- | --- |
| suite直下の`suite.json` | 全試験の成否・終了コード・summaryのハッシュ |
| 各runの`summary.json` | 判定条件・測定値・エラー・BINハッシュ |
| `run.json` | コードの版、入力ハッシュ、起動コマンド |
| `model.json`・`experiment.parm`など | 実行時の設定とプログラムのコピー |
| `telemetry.csv` | 推定位置と真値、モード、高度、PWM、電流、測距の時系列 |
| `sitl.log` | SITLの起動・動作メッセージ |
| `logs/*.BIN` | 機体内部のDataFlashログ |

結果表は次で表示できます。

```bash
.venv/bin/python scripts/summarize_prototype.py runs/prototype-final-20261116
```

生ログはGit管理対象外です。現在の本試験の入力コピーには、当時使った共通コードの旧ファイル名も残っています。これは結果の追跡用スナップショットで、現在の実行コードは`scripts/`、モデルは`config/prototype-01/`を使います。

## 7. うまく動かないとき

| 状態 | 確認すること |
| --- | --- |
| Python 3.12が見つからない | Homebrewの`python@3.12`を確認。別の場所なら`SITL_PYTHON=/path/to/python3.12 ./scripts/setup.sh`を使用 |
| 実行ファイルがない・パッチ検査に失敗 | `./scripts/prepare-prototype.sh`の出力を確認 |
| ポート使用中 | 対話起動はinstance 41、自動試験は既定instance 40。同じインスタンスの既存起動を終了してから再実行する |
| アームできない | 各runの`sitl.log`と`summary.json`でPreArm・GPS・EKF状態を確認 |
| 試験が失敗した | `summary.json`のerrorとCSV/BINを確認。アームチェックや合格条件を無効化しない |
| 図が出ない | [可視化ガイド](visualization.md)で、成功済みsuiteと描画用依存を確認 |

[第１弾のSITLへ戻る](../README.md)
