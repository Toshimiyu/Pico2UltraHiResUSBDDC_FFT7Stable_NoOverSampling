# Pico2UltraHiResUSBDDC_FFT7Stable_NoOverSampling
## Description
- 7-band FFT spectrum display version without oversampling and FIR filtering.
- This project is based on the original work by ArqAlice (MIT License).
- Originally developed for the PICO_AUDIO_PACK environment (RP2040), this firmware has been reworked for Raspberry Pi Pico2 (RP2350).
- It does not support RP2040-based boards.
- GPIO assignments have been remapped to GP9 / GP10 / GP11 to maintain compatibility with the original PICO_AUDIO_PACK hardware.
- Additional modifications have been implemented to improve audio quality, stability, and compatibility.
- Operation has been verified with PCM5100 on PICO_AUDIO_PACK hardware.

## 概要

- 本プロジェクトは、RP2350（Raspberry Pi Pico2）上で動作する USB Audio Class 1.0 準拠の USB-DDC（Digital to Digital Converter）です。
- USB経由で入力された2ch PCMオーディオ信号をオーバーサンプリングやFIRフィルタ処理を行わず、そのままI²Sインターフェイスへ出力します（NOS化）。
- OLEDディスプレイ上にリアルタイム7バンドFFTスペクトラム表示機能を搭載しています。

## 特長

- USB Audio Class 1.0 準拠
- Raspberry Pi Pico2 (RP2350) 専用
- No Oversampling (NOS)
- No FIR Filtering
- ダイレクトPCM出力
- Bit-Perfect Playback Compatible
- OLEDリアルタイム7バンドFFT表示
- DMA + PIOによる低遅延I²S伝送
- Dual Core RP2350対応
- PICO_AUDIO_PACK互換
- PICO Display PACK2.0互換

## 表示仕様（Pico Display Pack 2.0）
ディスプレイ上では、以下のリアルタイム情報およびオーディオビジュアライザーが描画されます。
* **L/R レベルメーター**: ステレオ音声の音量レベルをリアルタイム表示
* **L・R 独立 7バンド FFT 表示**: 左右チャンネルそれぞれの周波数特性を7バンドで分離して視覚化
* **オーディオ情報表示**: 現在の「ビットレート」および「サンプリング周波数 (Fs)」をリアルタイム表示
* **描画に関する注意**: 本ファームウェアは**音質優先設計**となっているため、オーディオ処理を最優先した結果として画面の描画にちらつき（フリッカー）が生じる場合がありますが、これは仕様です。

## 対応DAC

- TI PCM5100

※ PCM5100はPICO_AUDIO_PACK環境で動作確認済み

## USB入力仕様

- USB Audio Class 1.0
- フォーマット: I²S 32bit（左右チャンネル交互）
- チャンネル数: 2ch Stereo
- ビット深度: 16bit / 24bit PCM
- サンプルレート: 44.1kHz / 48kHz / 88.2kHz / 96kHz

## I²S出力仕様
- I²S 32bit Format
- Stereo
- Input Sample Rate Follow Mode
- No Sample Rate Conversion
- No Oversampling
- Bit-Perfect Playback

※　ビットパーフェクト再生を行う場合は、Windows側の再生デバイス音量を100%に設定してください。
- Windowsの音量を下げるとデジタルボリューム処理が適用され、USBオーディオデバイスへ送られるPCMデータが変更される場合があります。
- より正確な再生を行うために、WASAPI Exclusive Mode（排他モード）の使用を推奨します。

## Playback Notes
- 高サンプリングレート再生時は、ご使用のPC環境や負荷状況により音切れが発生する場合があります。
- 音切れが発生する場合は、再生ソフトウェアまたはOSのサンプリングレート設定を下げてご使用ください。
- システム負荷を低減することで再生が安定する場合があります。

- Audio dropouts may occur during high sample-rate playback depending on the host PC performance and system load.
- If audio dropouts occur, reduce the playback sample rate in the operating system or audio player settings.
- Lower sample rates may improve playback stability.

## FFT Spectrum Display
### Real-time 7-band FFT spectrum analyzer.

表示・解析デバイス: Pico Display Pack 2.0 (ドーターボード経由接続)

Center frequencies:

- 100Hz
- 220Hz
- 460Hz
- 1kHz
- 2.7kHz (2k7)
- 7.4kHz (7k4)
- 16kHz (16k)

### FFT Configuration:
- FFT_LEN = 1024

The OLED display provides a real-time 7-band spectrum visualization optimized for RP2350 performance and low display latency.

## 使用技術・構成

- ハードウェア: RP2350 (Raspberry Pi Pico2)

- 表示・解析デバイス: Pico Display Pack 2.0 (ドーターボード経由接続)

- データ転送:
  DMA + PIO による低遅延 I²S 出力

- マルチコア処理:
  Core0
  - OLED描画処理
  - Power Mode管理
  - クロック切り替え制御
  - DACミュート制御
  - タイマー管理処理

  Core1
  - I²S出力処理
  - DMA転送処理
  - FFT解析処理

- USB制御:
  LUFAベース USB Audio Class 1.0 実装

- クロック同期:
  USB Audio Asynchronous Feedback Endpoint を利用したフィードバック制御

- タイミング制御:
  Repeating Timer による周期管理

## ピンアサイン詳細
- I2S DATA : `GP9`
- I2S BCLK : `GP10`
- I2S LRCK : `GP11`
- I2C SDA  : `GP6`
- I2C SCL  : `GP7`
- DAC ENABLE : `GP5`
- POWERMODE SW : `GP0`
- Pico Display (SPI/Ctrl) : `GP16` - `GP20`
- Pico Display カラーLED : ピンアサインの衝突を防ぐため**停止状態（非使用）**に設定
- Pico Display ボタン (A, B, X, Y) : 未使用

## ビルド・使用方法
1. 機器の接続は**ドーターボード経由**で行ってください。
2. Visual Studio Code 上で Raspberry Pi Pico 拡張機能をインストールする。
3. 本リポジトリをクローンする。
4. VSCode上でビルドを実行する（`build/src` にバイナリが生成されます）。
5. Pico 2 の **BOOTSEL ボタン** を押しながらPCに接続し、ドライブ（`RP2350`）として認識させる。
6. 生成されたファームウェア（`.uf2`）をドラッグ＆ドロップして書き込む。
7. 接続後、OS標準のUSBオーディオデバイスとして自動認識されます。

## ライセンス
- MIT License
- Original Copyright (c) 2025 ArqAlice
- Additional modifications for RP2350 support, PICO_AUDIO_PACK compatibility, OLED FFT display, and No Oversampling version by Toshimiyu.

## 参考文献
- USB Audio Class 1.0 Specification
- Raspberry Pi Pico SDK
- LUFA USB Framework
