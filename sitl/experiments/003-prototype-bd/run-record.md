# 実験003：実行記録と追跡情報

## 固定した環境

実行コードは [b114b11](https://github.com/m8k8/drone-sitl/commit/b114b11a92de9dace9dc0cbf7043cd659ed3a8e1)。本試験の全実行で、このコミットから変更のない作業ツリーを使用したことをrun.jsonで確認済み。

| 項目 | 値 |
| --- | --- |
| OS / CPU | macOS-26.2-arm64-arm-64bit |
| Python | 3.12.12 |
| ArduPilot | Copter 4.7.2、`fb773567896014da007bca58358704d4a0d8516f` |
| 元の部品表 | docs `83dfb0be7130008cc49d900d6633d6adef8740dd` |
| 起動地点 | CMAC、緯度−35.363261・経度149.165230・標高584m・方位353° |
| 実行方式 | instance 40、速度5倍、独立UDP通信、新しい保存状態 |
| 本試験の保存先 | sitl/`runs/prototype-final-20261116/` |

ArduPilot作業ツリーの差分は、SIM_Frame.cpp/.hのreferenceMass分離パッチ。機体質量を変えても推力校正を固定するための変更で、パッチ済みソースのハッシュをprepare-prototype.shが照合する。現在のパッチと校正資料は`config/prototype-01/`へ集約している。

## 入力とバイナリのSHA-256

| 対象 | SHA-256 |
| --- | --- |
| ArduCopterバイナリ | `4c41933f395d83255b2b49304300df4fca5c5c8775ce4a28bf74f5280e658d29` |
| model.json（nominal） | `c87861cfd45aa5b16f56a361f779505873116e8aeab3cac540c63adc266249ca` |
| experiment.parm | `aec8cc893c2983d8ea13c51c421bc67f4e4ec9b9439cd904c8c9c6020e6f902d` |
| base.parm | `5e01345b45d1c6190b28bece5638bbdd4cf1cce35e05bbbf480ab24d2b51aa0e` |
| manifest.json | `3e0ac3f2e48e4b36ca32b60347ac83c801ec991c2e6bb94576411d50fa8707e8` |
| runner.py | `3220a7e5ff8e0aede71097afe6075a5387ef5df0fac19294650c096bbb98819d` |
| payload_test.py | `fceb8eb128715f8110626a8c413e1556129ac78294dd9e4b25701103f6e67f51` |
| components.json | `12376a2faf984d0446432b79c4f0033b779f93e53b1bd6bfb25a425bfae15e88` |
| mass-properties.json | `197baa35d5fe227606ffd98ab273952ca6076941ce745b937222ee69a0b49933` |
| calibration.json | `7d2d900b7df498eef930e75429f7c097e9578933db59b492ae525a308d7d1ea9` |
| reference-mass.patch | `56d9bd28677a0e723cf01340a9f93b0e39d136a04a3bdf37f9dba1adf5619cfb` |
| model.json（mass-low） | `a4004e2b2c80c7ffdbfd1e828b550e8c38d4610fe899def23576794fbd0fe1c8` |
| model.json（mass-high） | `dc64fedeedd4fada644518b02a93ec974aa7cdeae72ae66bf1edb8af9a8a9b3a` |

入力コピーとCSV・BINはローカルの各runに保存し、公開リポジトリへ生ログを含めない。下記のハッシュは記録を照合する識別子であり、このページだけから生ログの再解析ができるという意味ではない。

## 本試験前の修正と記録の整理

本試験の前に、RTL項目の未使用yawへNaNを送る問題と、MISSION_STARTに固定版Copterが受け付けない開始・終了項目番号を渡す問題を修正した。未使用引数を0、MISSION_STARTの項目指定を0,0に統一し、正常な指令手順で試験した。帰還中の期待モード・観測の新しさと、全シナリオの非空BIN保存も成功条件に含めている。

2026-11-16の整理では、利用者の指定により予備試験のディレクトリと他のシミュレーション結果を削除した。このページで追跡する生ログは、11回の本試験を含む`runs/prototype-final-20261116/`である。本試験結果・入力コピー・ハッシュ・図は変更していない。過去のコードはGit履歴から確認できる。

このページの`payload_test.py`や旧manifestのハッシュは、本試験当時の入力コピーを指す。現在のコードを旧版に戻したり、整理後のハッシュとして読み替えたりしない。

## 本試験の確定記録

実行開始：2026-11-16T18:52:58.119892+09:00。終了：2026-11-16T18:58:44.907072+09:00。suite全体の実時間は346.8秒。11回の試験結果と終了コードはいずれも成功。全runの実行コードはb114b11、作業ツリーは変更なし。

suite.json SHA-256：`25ae85f1efa874ef0134433a99b9c1e3ecc2ac0ecb6507fd68e6a04fdb84617c`

入力10ファイルずつ・全BINのハッシュを再計算し、保存した値と照合済み。帰還区間はflight/rangefinderでRTL（6）、missionでAUTO（3）を継続し、新しい真値・heartbeatを受信した。以下は各runのsummary・CSV・BINを特定する値。

| run | summary.json SHA-256 | telemetry.csv SHA-256 |
| --- | --- | --- |
| nominal-flight-repeat-1 | `3cecae049ad6a5f4a4e65097f643f772618ba3d51d9d096fc38f3f7c915df387` | `ed615be9ecfdd70450b7b84d5b60a7b11fdee18447500e500a612af6949db8e7` |
| nominal-flight-repeat-2 | `a80c8d9912fae2d78f65571329f15013472faad94ac2fb772f7e8a9bd229d11b` | `a57f395029fb0ccff96ad2f84e8cb702811a9ff8a1770f7e0b8ead4db3452892` |
| nominal-flight-repeat-3 | `3ac3d345a10a2c5e6fe26c5f91b4d4b0a9c34b2768b2e90db58ef7464429bbfd` | `d3961177ecd56e5db07d929f81ed9f34d5036a16d970794a6e82ac559071fd20` |
| nominal-mission-repeat-1 | `26ea50d7abe01151559d524275bc6f0268dab82f77ed53a5a3902b18d974c4ee` | `ce1423e84d7e11b00232a70eea339ab6bba7b6ce94c593a4426d01c789ab2ebe` |
| nominal-mission-repeat-2 | `8fdaf2c07505009383d8c48d6dda559e52834da2754f7b4891776c4f9f2d93bc` | `4668a45b6c8d597cfb54e4be9f95d2fccd81c432f744df0167ba1324af17ba53` |
| nominal-mission-repeat-3 | `ed501365c15feffb38f9560ea4417bfa781b321aac7b91e99002fbbba07725ac` | `d9c8d9249a9bd0b24a5733917902c89c7f77a6a8a8eebf0bef8193d153fdb6f9` |
| nominal-rangefinder-repeat-1 | `764261ca35e4d75f73776064550fc27064a2204549e45d407bd94395e4f61942` | `3c5857222c5e19951abb61158801409d6126a88c8d6cbae36b04547c3a8e620d` |
| nominal-rangefinder-repeat-2 | `5d140950066475bc1a12a48710682a40f947829c9e93d07c2a4023cef1430368` | `eb00b0698c27fe610898486dbd1d461f9aed3005d1928970cf0d2bca7f7a66b3` |
| nominal-rangefinder-repeat-3 | `a035aa8bf9fd92002f70f914353e1bde624848840f517bb1fabca8e1ddacd5e0` | `8d9fe8570aa4c4a598a5303f2b6eccd40aa7dc26621567ce3cda60f1b8b1d69f` |
| mass-low-flight-repeat-1 | `991cbdaadd810f1893adbb06103040cba1c3486c50321bcba7a910d3776d97e2` | `cce3b6187428b31c899f41212605d147d41f0bcc8f46921c40b4207d769a1399` |
| mass-high-flight-repeat-1 | `712b2bf8cdd16647d40740d4a86a20a8bdc0e56abfc063275da09aaed09c2849` | `da938f6eedb0febe26620205007573125e6994904b0cb497d933d65f334348fc` |

| run / ログ | BIN SHA-256 |
| --- | --- |
| nominal-flight-repeat-1/logs/00000001.BIN | `1f34d46104403c343fd8e9899825f5892f13d3ae8fe2156b1a58c7013431bfbc` |
| nominal-flight-repeat-2/logs/00000001.BIN | `1e8c310219760abfec3317f37d9d8ef3105133edbeca3e907cbd160901341cbc` |
| nominal-flight-repeat-3/logs/00000001.BIN | `b20224136373403720f181db19847e6a285ad69921116c4a6e375d9852e465d1` |
| nominal-mission-repeat-1/logs/00000001.BIN | `d95ea4f76ed842b457edb3ac6f5faf27aee518fee3ecb66996d0a5de4d12d4d7` |
| nominal-mission-repeat-2/logs/00000001.BIN | `177abcc6fb0113dd0d2b3cb5603b6d9f84544497b329195dd82711f8ce417006` |
| nominal-mission-repeat-3/logs/00000001.BIN | `c3aaec59731348e923efc240b0456c8f84394fa2809c572151d43060da3a097b` |
| nominal-rangefinder-repeat-1/logs/00000001.BIN | `479d37b284fe51dbb8b2e12769e54581e70fe5480ea5d43db5b12877b23bb8ec` |
| nominal-rangefinder-repeat-2/logs/00000001.BIN | `e72d764ff3a3822432323b2a8aee67c98ef1b0a2f61ce798a671ef10621aca4b` |
| nominal-rangefinder-repeat-3/logs/00000001.BIN | `e5b2ab4e670b70fa4e13ce8b36b68e6b8924ce6bb63f0564754463bf0293188b` |
| mass-low-flight-repeat-1/logs/00000001.BIN | `e8eaa6551aa90ac45afb27b5aaba42ff691182e15b36de4b4a8b3136dbc0762f` |
| mass-high-flight-repeat-1/logs/00000001.BIN | `3b5a93673b7be4ea869571c5a3a31772b097fc5c65dad07a24277523cabfb2d6` |

## 整理後の動作確認

2026-11-16に、他のモデル・旧試験コードを削除し、第１弾の共通処理と推進校正を集約した後の動作を確認した。整理内容を固定したコードは[1ddbd77](https://github.com/m8k8/drone-sitl/commit/1ddbd772b03601539b9be6941f7a75c8f45fefe3)。機体モデルの数値とパラメータは維持した。

自動試験は`runs/prototype-cleanup-20261116/`へ保存し、`prototype-suite.sh --repeats 1 --no-mass-sensitivity`で基本飛行・AUTO・測距を各1回実行した。開始は19:26:33、終了は19:28:10（JST）。**3/3成功、全終了コード0**で、通常アーム・帰還・着陸・ディスアームと機体BIN保存を確認した。

| 試験 | 結果 | 主な確認値 | 真値着地ずれ |
| --- | --- | --- | ---: |
| flight | 成功 | 北20mのホバーで真値水平ずれ最大0.172m | 0.0287m |
| mission | 成功 | 4地点到達。各WP実行中の真値最近接距離の最大0.0213m | 0.0454m |
| rangefinder | 成功 | 1m短い測距に対する真値高度変化+1.0065m。範囲外・復旧のBIN状態も一致 | 0.0287m |

実行時のHEADは`b114b11`で、整理中の未コミット差分を含んでいた。`run.json`の変更状態はそのまま保存しており、最終コミットのclean状態で実行したとは扱わない。実行時にコピーした入力33ファイルは、その後確定した`1ddbd77`の対応ファイルと照合して一致した（`base.parm`は固定版ArduPilotの原本と照合）。3件のBINハッシュも保存値と一致した。

| 対象 | SHA-256 |
| --- | --- |
| 整理後suite.json | `88d3aa55e4ec465a8ad726271fcf596fcd1cce29ab6103f251c086686da3f62e` |
| 整理後manifest.json | `1da4606e92f2ce9bb5fb70986552ef21af0a1ab0d6f86efcc82b20b587582da3` |

対話起動は`runs/prototype-manual/sessions/20261116T102747Z-zwu63l00/`で確認した。実行中の質量12.3387kg・校正基準13.4946kgの照合、`FRAME_TYPE=1`、`RNGFND1_TYPE=27`、`ARMING_SKIPCHK=0`の読戻し、RC3の1500受信、GUIDED/STABILIZE切替の受理を確認した。**この対話起動試験ではアーム・離陸をしていない。** Ctrl+C後に自身の2子プロセスの終了と専用TCP/UDPポートの解放を確認した。起動スクリプトの終了コード130はこの停止操作によるものである。QGroundControlのGUI表示は未検証のまま。

整理時に旧起動・旧モデル・予備試験の12ディレクトリ（約170.9MiB）を削除した。当初の本試験`runs/prototype-final-20261116/`の210ファイルは全ハッシュ不変で、上記11回の測定値・図・追跡情報をそのまま残している。

## 結果の位置づけ

試験の判定条件・測定値・図は[実験結果](README.md)、質量・配置・センサーの仮定は[モデル説明](model.md)を参照する。このページは保持した本試験の履歴であり、削除した別モデルの結果は含めない。
