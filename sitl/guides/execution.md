# SITLの環境構築と標準機の実行方法

この手順では、Macの中に仮想ドローンを起動し、離陸・位置保持・帰還・着陸を試します。実機、送信機、フライトコントローラーは不要です。

実行コードは[drone-sitl](https://github.com/m8k8/drone-sitl)、可視化の接続方法は[可視化ガイド](visualization.md)、確認済みの範囲は[実験001：標準機の基本飛行](../experiments/001-standard-quad/README.md)にあります。

環境構築は各実験で共通です。この文書の`run.sh`と`smoke-test.sh`は、総質量3kgの標準quad用です。G06の暫定モデルを使う`payload-test.sh`は別の実験であり、条件と再実行手順は[実験002](../experiments/002-g06-payload/README.md)を参照してください。[実験一覧](../README.md)で目的に合う手順を選べます。

## 1. 必要なものと構成

| 項目 | 使用するもの |
| --- | --- |
| OS・CPU | macOS / Apple Silicon |
| コンパイラ | Xcode Command Line Tools、または開発ツールを含むXcode |
| Python | Python 3.12、プロジェクト専用の`.venv` |
| 飛行制御 | ArduPilot `Copter-4.7.2` |
| 固定コミット | `fb773567896014da007bca58358704d4a0d8516f` |
| 操作画面 | MAVProxy 1.8.75のCLI |
| 機体モデル | 総質量3kgの標準quad（QUAD/PLUS）。G06専用モデルではない |
| 開始地点 | 公式サンプルのCMAC。利用者の実際の飛行場所ではない |

コードとPythonパッケージの初回取得にはインターネット接続が必要です。SITL向けの環境なので、実機ファームウェア用STM32クロスコンパイラは使いません。

## 2. ディレクトリを準備する

すでに`drone/sitl`があり、セットアップとビルドが完了している場合は「4. 起動する」へ進めます。以下のcloneは新しい環境で行う手順です。

作業先の親ディレクトリで、**macOSのターミナル**に入力します。

```bash
mkdir -p drone
cd drone
git clone https://github.com/m8k8/drone-sitl.git sitl
git clone https://github.com/m8k8/drone-docs.git docs
cd sitl
```

既存の`docs`または`sitl`がある場合は、その場所へ重ねてcloneせず、既存のディレクトリを使います。

## 3. 初回セットアップとビルド

開発ツールとPythonを確認します。

```bash
xcode-select -p
/opt/homebrew/opt/python@3.12/bin/python3.12 --version
```

不足している場合だけ、次を実行します。`brew`が未導入なら先に[Homebrew公式手順](https://brew.sh/)で導入してください。

```bash
xcode-select --install
brew install python@3.12
```

`sitl`ディレクトリで準備します。

```bash
./scripts/setup.sh
./scripts/build.sh
```

- `setup.sh`：固定版のArduPilotとサブモジュールを`vendor/ardupilot`へ取得し、`.venv`へ固定したCLI依存を導入します。
- `build.sh`：`waf configure --board sitl`と`waf copter`で、このMac用の実行ファイルを作ります。
- コードや依存のバージョンは`config/versions.env`と`requirements-cli.txt`で管理します。
- シェル設定ファイルやグローバルPythonは変更しません。

初回ビルド後の実行ファイルは`vendor/ardupilot/build/sitl/bin/arducopter`です。各コマンドの終了を待ち、エラーで停止した場合は次へ進む前に出力を確認してください。

## 4. 起動する

`sitl`ディレクトリで実行します。

```bash
./scripts/run.sh
```

保存済みパラメータを初期化して練習し直す場合は、SITLを終了してから次を使います。

```bash
./scripts/run.sh --reset
```

起動後はMAVProxyの入力画面になります。`Detected vehicle`、`ArduPilot Ready`などを確認し、GPSとEKF（姿勢・位置推定）の準備を待ちます。`STABILIZE>`などの表示が入力位置の目印です。

起動直後の`Connection refused sleeping`は、本体起動とのタイミング差で一時的に出ることがあります。続いて機体が検出されれば接続できています。

## 5. 最初の飛行

ここからのコマンドは**MAVProxyの入力欄**に入れます。macOSのシェルに入力するものではありません。

### GUIDEDで離陸する

```text
mode GUIDED
```

GUIDEDになったらアームします。

```text
arm throttle
```

`ARMED`を確認してから、10秒以内に離陸を指示します。

```text
takeoff 10
```

`PreArm: Need Position Estimate`などで拒否されたら、表示内容を確認し、GPS・EKFの準備が終わるまで待ちます。通常のアーム判定を無効化したり、強制アームしたりする手順ではありません。

### 高度を確認する

上昇後に入力します。

```text
status GLOBAL_POSITION_INT
```

`relative_alt`はmm単位です。約`10000`が離陸基準から約10mを表します。`alt`は別の基準の高度なので、ここでは`relative_alt`を見ます。コマンドはその時点の値を表示するため、繰り返して変化を確認できます。

### LOITERで位置・高度を保つ

高度が安定したら、仮想RCのスロットルを中立にして切り替えます。

```text
rc 3 1500
mode LOITER
```

数秒待って、もう一度`status GLOBAL_POSITION_INT`を入力します。位置と高度が大きく変わらないことを確認します。LOITERではスロットルが上昇・下降指示になるため、中立値1500を先に送っています。

### RTLで帰還・着陸する

```text
mode RTL
```

着陸し、`DISARMED`が表示されるまで待ちます。RTL設定によっては一度高度を上げます。この練習では離陸地点付近の上空にいるため、水平移動はほとんどありません。

その場への着陸を指示する場合は次を使います。

```text
mode LAND
```

### 終了する

着陸・ディスアーム後に`Ctrl+C`で終了します。シェルへ戻れば、次回は再び`./scripts/run.sh`で起動できます。シミュレーターを閉じる操作と、機体への着陸指示は別です。

## 6. 自動テストを実行する

macOSのシェルで、`sitl`ディレクトリから実行します。

```bash
./scripts/smoke-test.sh
```

独立したSITLを起動し、heartbeat、GPS・EKF・pre-arm、GUIDED、通常アーム、10m離陸、LOITER、RTL、着陸・自動ディスアームを確認します。`PASS`と終了コード0が成功です。

自動テストは5倍速・専用の実行ディレクトリを使います。通常の対話起動とは別のプロセスで、既存のTCP 5760の機体には接続しません。この自動テストの通信をQGroundControlへ転送する設定は含まれていません。可視化したいときは`run.sh`を使います。

## 7. ログの保存先

以下は`drone-sitl`リポジトリのルートからの相対パスです。

| 記録 | 保存先 | 内容 |
| --- | --- | --- |
| 保存パラメータ・状態 | `runs/default/` | 通常起動の状態。`--reset`でパラメータ初期化 |
| 機体ログ | `runs/default/logs/*.BIN` | 機体側の記録 |
| 通信ログ | `runs/default/mav.tlog` | MAVProxyが受信したテレメトリー |
| 自動試験の結果 | `runs/smoke-*/summary.json` | 成否、各ステップ、測定値 |
| 自動試験の起動ログ | `runs/smoke-*/sitl.log` | 試験専用SITLの標準出力・エラー |
| 対話起動時の本体メッセージ | `/tmp/ArduCopter.log` | 公式headless launcherの出力。次回起動で上書き |

必要な実験ログは次の起動前に別の場所へ保存してください。`runs/`はGit管理対象外です。公開するテスト記録は、[実験001の結果](../experiments/001-standard-quad/README.md)のように試験条件と観測値を整理し、実験ごとのディレクトリに保存します。

## 8. うまく動かないとき

| 状態 | 確認すること |
| --- | --- |
| Python 3.12が見つからない | Homebrewの`python@3.12`を確認。別の場所なら`SITL_PYTHON=/path/to/python3.12 ./scripts/setup.sh`を使用 |
| 実行ファイルがない | `./scripts/build.sh`が成功したか確認 |
| TCP 5760が使用中 | 先に起動したSITLをそのターミナルで終了してから再実行 |
| 接続拒否が繰り返される | `/tmp/ArduCopter.log`で本体の起動失敗を確認 |
| アームできない | `PreArm`の原因とGPS・EKFの準備状態を確認 |
| 離陸しない | GUIDED、ARMED、アームから離陸指示までの時間を確認 |
| LOITERで降下する | `rc 3 1500`を送ってスロットルを中立にする |
| 地図が出ない | 現在の起動はCLI。別途[可視化ガイド](visualization.md)でQGroundControlを接続 |

## 参照

- [ArduPilot：macOSビルド環境](https://ardupilot.org/dev/docs/building-setup-mac.html)
- [ArduPilot：SITLの使い方](https://ardupilot.org/dev/docs/using-sitl-for-ardupilot-testing.html)
- [ArduPilot：Copter SITL/MAVProxyチュートリアル](https://ardupilot.org/dev/docs/copter-sitl-mavproxy-tutorial.html)
- [MAVLink：GLOBAL_POSITION_INT](https://mavlink.io/en/messages/common.html#GLOBAL_POSITION_INT)
