# 実験003：第１弾（B〜D）のモデルと機能の再現範囲

この実験は[第１弾の搭載部品表](../../../hardware/prototype-01/README.md)を基準にする。部品を購入・実測した機体の再現ではなく、**設計重量12.3387kgと仮の部品配置を使い、飛行・航路実行・下向き測距の制御を検証するモデル**である。結果と判定条件は[実験記録](README.md)に記載する。

## 1. 物理モデル

| 項目 | 今回の扱い | 限界 |
| --- | --- | --- |
| 名目質量 | 12.3387kg、電池込み | 部品表の公表値・材料計算・仮割当の合計。実測値・保証上限ではない |
| 機体 | G06の対角1.172mを正方形Xで近似 | 実物の矩形配置・強度・脚の形状を再現するCADモデルではない |
| 推進系 | X6 Plus G2×4、MFP24×8.0。実験002と同じ推力係数 | 静止推力表の近似。モーターの熱、振動、故障は未再現 |
| 推進校正の基準質量 | `referenceMass=13.4946kg`のまま固定 | 今回の実質量12.3387kgとは別。機体を軽くしたことに合わせてモーターの能力を変えない |
| 質量配置・慣性 | 電池・推進系・追加トレー・機器群を分け、仮の座標と寸法から集計 | 取付位置・重心はまだ測定していない。実機の同定値ではない |
| 電池 | 6S 10Ah×2本直列、12S 10Ah、初期50.4V | 内部抵抗は実験002の仮定。補機ごとの電力変換や配線抵抗は未再現 |
| 非搭載 | タンク本体、水、散布装置、F9P、RTK無線・MCU | 段階E・Fの完成形をこの実験の重量に混ぜない |

部品表を反映しても、フレーム5.4kgの包含範囲、カメラの正味重量、追加配線、固定具の重量に関する不確実性は残る。推進能力・電流モデルの校正範囲も[実験002のモデル](../002-g06-payload/model.md)を引き継ぐ。特に電流と電池容量の表示だけから、実機の飛行時間やAS150/XT150端子の使用可否を判定しない。

機械可読の部品表・配置・生成条件は実行リポジトリの[`config/prototype-01/`](https://github.com/m8k8/drone-sitl/tree/b114b11a92de9dace9dc0cbf7043cd659ed3a8e1/config/prototype-01)で管理する。元のExcelの重量表を読み替えた単一の「実測値」にはしていない。

名目ケースでは、電池2本の中心をモーター面から180mm下、左右±35.5mmに置く。各電池は170×61×58mmの箱として扱い、横方向のすき間は10mmと仮定した。フレーム5.4kgは中央の箱3.0kgと各アーム途中の0.6kg×4へ分配する。BOMの30行を45個の質点・箱に展開しており、その他の部品も重量・仮配置を計算に含める。

この仮配置から得る重心は、モーター面中心より前方1.48mm・下方55.44mm。重心まわりの慣性対角成分は約`[0.72593, 0.75059, 1.31983] kg·m²`となる。各モーター位置をこの重心基準へ変換してSITLへ渡す。ネイティブモデルは慣性の対角成分のみを扱い、積慣性と一般剛体の角速度結合項は再現しない。

`models/nominal.json`のほか、中央フレームの仮質量だけを±0.5kg変えた`mass-low.json`（11.8387kg）と`mass-high.json`（12.8387kg）を用意する。これは重量が想定と違う場合の感度確認条件であり、実機重量の下限・上限や水積載量ではない。どの条件を実際に試験したかは実験記録で区別する。

ネイティブの接地高さは0.1m固定で、脚・下吊り電池・トレーの衝突形状はない。離着陸に成功しても、実機の地上クリアランスや電池トレーの取付適合が確認できたことにはならない。また、電池モデルの電流は推進系の簡易値であり、カメラ・無線・FC・センサー・DC-DCの消費電力を個別加算していない。

## 2. 部品と機能の対応

| 第１弾の部品・機能 | SITLへ反映する内容 | 今回再現しない内容 |
| --- | --- | --- |
| G06・取付板・固定具 | 質量、仮の分布・慣性 | 実形状、変形、締結強度、プロペラ干渉 |
| X6 Plus G2・プロペラ | 4基の推力・トルクによる飛行運動 | 個体差、温度上昇、実際のESC通信・保護動作 |
| 6S 10Ah電池2本（直列で12S 10Ah） | 質量・配置、簡易電圧低下と容量消費 | 電池の実測放電曲線、セル差、端子発熱 |
| Pixhawk 6C | 実機と同系列のArduCopter制御ソフト、EKF3、航路実行 | Pixhawk基板のCPU・電源・配線・センサー個体特性 |
| M10・IMU・コンパス・気圧計 | 仮想GPS・慣性・方位・気圧データによる姿勢・位置推定 | M10の型番固有誤差、磁気干渉、RTK補正 |
| TF02-Pro UART | Benewake系のシリアル距離パケットを受け、下向き距離によるSurface Tracking | 光学系、反射率・照度依存の誤差、実センサーの全更新周期・温度情報 |
| プログラム制御 | 機体へ登録したAUTOミッションと、試験プログラムからの指令 | Luaを追加しない試験はLuaの動作確認にはならない |
| G12送受信機 | 受信機・アンテナの重量。操縦指令はSITL/MAVLink経路で代替 | G12の無線伝搬、電波認証、SBUS配線、映像伝送遅延 |
| FPVカメラ | 設計上の重量・配置 | カメラ映像、画角、Ethernet映像ストリーム。地図・姿勢表示はFPV映像ではない |
| PM08・FCHUB・ハーネス・ヒューズ | 質量、簡易電池テレメトリー | 各5V/12V系、保護協調、過電流・突入・断線、電源ノイズ |
| FRID-1・microSD | 質量、SITL上でのログ記録 | Remote ID送信・実カードの書込み性能 |

## 3. TF02-Proに対応するシリアル距離センサー

ArduPilotの現行資料は、TF02-ProのUART接続を`RNGFND1_TYPE=27`（BenewakeTF03）としている。旧TF02用の19とは区別する。[ArduPilotのBenewake接続資料](https://ardupilot.org/copter/docs/common-benewake-tf02-lidar.html)

SITLにはTF02-Pro専用の光学モデルがないため、ネイティブの`benewake_tf03`を使う。これは共通の`0x59 0x59`ヘッダー、cm単位の距離、チェックサムを持つ9バイトのUARTパケットを実際のArduPilotドライバーへ渡すもの。TF02-Proの製品特性全体をTF03の仕様に置き換える意味ではない。

| 設定 | 値・意味 |
| --- | --- |
| 起動引数 | `--serial5=sim:benewake_tf03`。SITLの仮想UARTを使用 |
| `SERIAL5_PROTOCOL` / `SERIAL5_BAUD` | 9（Rangefinder）/ 115（115200baud） |
| `RNGFND1_TYPE` | 27 |
| `RNGFND1_MIN` / `RNGFND1_MAX` | 0.1m / 13.5m。4.7.2のパラメータ単位はm |
| `RNGFND1_ORIENT` | 25（下向き） |
| `SURFTRAK_MODE` | 1（地面を追従） |
| `EK3_SRC1_POSZ` / `EK3_RNG_USE_HGT` | 1（気圧計）/ −1（距離センサーへの高さ源切替を無効） |
| 通常時の測距原点 | シミュレーターと機体設定の距離センサー位置を原点に置く仮定。実機のTF02取付位置とは別 |

13.5mは、[Benewake A02データシート](https://en.benewake.com/uploadfiles/2024/04/20240426135509321.pdf)の10%反射率の標準板に対する範囲を試験設定として採った値である。同資料では90%反射率の標準板に対して40m。SITLは反射率や照度を計算しないため、13.5mを超えると使用不能にする判定範囲を設定しただけである。

実センサーの既定更新は100Hz、距離分解能は1cm。SITLネイティブのこのUARTモデルは**5Hz**であり、実センサーの帯域・遅延を忠実に再現しない。`SIM_SONAR_RND`で加えるノイズも試験上の仮定であり、メーカーの精度・再現性と同じ統計分布ではない。MAVLinkを10Hzで配信しても、新しいセンサー測定が10Hzで得られるわけではない。

今回の`SIM_SONAR_RND=0.02`はネイティブモデルで距離へ加える±0.02mの一様乱数雑音である。メーカー精度の±5cm（0.1〜5m）・±1%（5〜40m）をそのまま再現した設定ではない。

`RNGFND1_GNDCLR`は着地時距離の設定であり、機体の実際の脚の高さを生成しない。ネイティブSITLの平地測距は初期機体高度との差から作られるため、実機のセンサー取付位置と地上クリアランスを再現したとは扱わない。SITLのSERIAL5指定も、実機のTELEM端子への配線指示ではない。

## 4. Surface Trackingと測距異常の試験方法

Surface Trackingは、通常の気圧計・IMUを使うEKF高度推定を維持しながら、距離センサーで地面の高さを推定して高度制御目標を補正する。EKFの高度源を距離センサーへ切り替える試験ではない。[ArduPilotのSurface Tracking](https://ardupilot.org/copter/docs/terrain-following-manual-modes.html)、[EKFの高さ源](https://ardupilot.org/copter/docs/common-ekf-sources.html)

平地で距離を表示するだけでは、測距を制御に使っている証明にならない。そこで、LOITER中にシミュレーター側だけの`SIM_SONAR_POS_Z`を変更し、測距に既知の偏りを与える。機体側の取付位置設定は変更しない。

| 試験操作 | 入力に起きる変化 | 確認すること |
| --- | --- | --- |
| 通常値0 | 平地までの距離 | 測距と真値の対応、ドライバーの正常状態 |
| `SIM_SONAR_POS_Z=+1` | 水平時に距離が約1m短くなる | 地面が近づいたと解釈し、高度制御が約1m上昇方向へ応答すること |
| 0へ戻す | 偏りを解消 | 上記の制御応答が元へ戻ること |
| `SIM_SONAR_POS_Z=-50` | 水平時に距離が約50m長くなり、設定上限を超える | 測距を使用しなくなっても、通常のEKFを使うLOITERで高度を維持できること |
| 0へ戻す | 使用範囲内の測距へ復帰 | 測距状態の回復と、復帰時の高度変化 |

これは**合成した距離偏り・範囲外値の注入**であり、地形生成、センサーの実移設、UART通信断、雨・水面での反射不良の再現ではない。範囲外の間もUARTパケットは届く。`SIM_SONAR_GLITCH`はアナログ測距の電圧生成側に作用し、このシリアルモデルへの故障注入には使わない。

ArduCopterのSurface Trackingは、良好な測距が途絶えて約1秒経つと地面追従の補正をリセットする。測距が復帰すると、その時点の地面距離を基準に追従を再開する。この動作は測距異常で自動的にRTLへ移行する機能ではない。最後のRTLを試験プログラムが指示した場合、その指示と異常時の自動動作を区別して結果へ記録する。

## 5. ログで確認する値

- `DISTANCE_SENSOR`：距離・最小値・最大値・方向。時刻はMAVLinkの送信時刻であり、センサーの新しい測定時刻とは限らない。`signal_quality=0`はこのモデルでは品質情報なしの意味で、異常の直接証拠にはしない。
- DataFlash `RFND.Stat`：0=未接続、1=データなし、2=範囲未満、3=範囲超過、4=正常。今回の範囲外注入では3を確認する。
- DataFlash `SURF.St`：bit1は測距が制御に使える健康状態でないことを示す。`SURF.D`は距離、`SURF.TO`は地面高さの推定オフセット。
- DataFlash `CTUN.DSAlt`：有効なSurface Tracking中の目標距離。使用しない状態ではNaNとなるため、異常値として一律に捨てず状態の証拠に使う。
- `SYS_STATUS`のLASER_POSITION健康ビット：この版では「データが届いているか」で決まる。**範囲外でも立ったままになり得る**ため、これだけで測距が制御に使われていると判定しない。

## 6. 実装根拠

参照するArduPilotは固定版`fb773567896014da007bca58358704d4a0d8516f`。今回の解釈は最新版Web説明だけでなく、実行する版の以下のコードと照合した。

- [シリアルBenewakeシミュレーター](https://github.com/ArduPilot/ardupilot/blob/fb773567896014da007bca58358704d4a0d8516f/libraries/SITL/SIM_RF_Benewake.cpp)と[TF03側のパケット項目](https://github.com/ArduPilot/ardupilot/blob/fb773567896014da007bca58358704d4a0d8516f/libraries/SITL/SIM_RF_Benewake_TF03.h)、[5Hz更新の基底クラス](https://github.com/ArduPilot/ardupilot/blob/fb773567896014da007bca58358704d4a0d8516f/libraries/SITL/SIM_SerialRangeFinder.h)。
- [距離・センサー位置・ノイズの生成](https://github.com/ArduPilot/ardupilot/blob/fb773567896014da007bca58358704d4a0d8516f/libraries/SITL/SIM_Aircraft.cpp#L552)、[平地高度の基準](https://github.com/ArduPilot/ardupilot/blob/fb773567896014da007bca58358704d4a0d8516f/libraries/AP_HAL_SITL/SITL_State.cpp#L384)。
- [Benewake受信ドライバー](https://github.com/ArduPilot/ardupilot/blob/fb773567896014da007bca58358704d4a0d8516f/libraries/AP_RangeFinder/AP_RangeFinder_Benewake.cpp)、[距離範囲による状態分類](https://github.com/ArduPilot/ardupilot/blob/fb773567896014da007bca58358704d4a0d8516f/libraries/AP_RangeFinder/AP_RangeFinder_Backend.cpp#L54)。
- [Surface Trackingの更新・復帰](https://github.com/ArduPilot/ardupilot/blob/fb773567896014da007bca58358704d4a0d8516f/ArduCopter/surface_tracking.cpp)、[距離の健康状態・グリッチ処理](https://github.com/ArduPilot/ardupilot/blob/fb773567896014da007bca58358704d4a0d8516f/libraries/AP_SurfaceDistance/AP_SurfaceDistance.cpp)。
- [SYS_STATUSの測距ビット](https://github.com/ArduPilot/ardupilot/blob/fb773567896014da007bca58358704d4a0d8516f/ArduCopter/GCS_Copter.cpp#L72)、[DISTANCE_SENSORの送信](https://github.com/ArduPilot/ardupilot/blob/fb773567896014da007bca58358704d4a0d8516f/libraries/GCS_MAVLink/GCS_Common.cpp#L449)。
