# zmk-config-CLine46

CLine46 の ZMK ファームウェアの設定です。

## ローカルでビルドする

GitHub Actions を使わずに、手元の PC で Docker を使ってビルドする手順です。Docker が使える状態になっている必要があります。

### 1. 準備

この「1. 準備」は、初めてビルドするときに1回だけ行います。2回目からは「2. ビルド」から始められます。

このリポジトリと、ZMK の作業フォルダ `zmk-ws` を、同じフォルダの中に並べて作ります。

```bash
mkdir cline46 && cd cline46
mkdir zmk-ws
git clone https://github.com/takamaru-fpv/zmk_config_CLine46.git zmk-config
```

```
cline46/
├── zmk-config/   # このリポジトリ
└── zmk-ws/       # ZMK 本体やビルドしたファイルが入る作業フォルダ
```

`cline46` フォルダで次のコマンドを実行して、ビルド用のコンテナを起動します。

```bash
docker run --rm -it \
  -v "$PWD/zmk-ws":/zmk-ws \
  -v "$PWD/zmk-config/config":/zmk-ws/config \
  -v "$PWD/zmk-config":/cline46 \
  -w /zmk-ws zmkfirmware/zmk-build-arm:stable bash
```

コンテナの中で次のコマンドを実行して、ZMK 本体とモジュールを `zmk-ws` にダウンロードします。

```bash
west init -l config
west update
```

`zmk-config/config/west.yml` を変更したときは、もう一度 `west update` を実行してください。

### 2. ビルド

`cline46` フォルダに移動し、上と同じ `docker run` のコマンドでコンテナを起動して、コンテナの中で次のコマンドを実行します。

```bash
# コンテナを起動するたびに1回実行する
west zephyr-export

# 右手側
west build -p -s zmk/app -d build/right -b seeeduino_xiao_ble -S studio-rpc-usb-uart -- \
  -DSHIELD="CLine46_R rgbled_adapter" -DZMK_CONFIG=/zmk-ws/config -DZMK_EXTRA_MODULES=/cline46

# 左手側
west build -p -s zmk/app -d build/left -b seeeduino_xiao_ble -- \
  -DSHIELD="CLine46_L rgbled_adapter" -DZMK_CONFIG=/zmk-ws/config -DZMK_EXTRA_MODULES=/cline46
```

ビルドしたファームウェアは次の場所にできます。

- 右手側: `zmk-ws/build/right/zephyr/zmk.uf2`
- 左手側: `zmk-ws/build/left/zephyr/zmk.uf2`

キーマップ（`zmk-config/config/CLine46.keymap`）だけを変更した場合は、右手側だけをビルドして書き込めば反映されます。

### 3. 書き込み

1. 電源スイッチを OFF にして、書き込む側を USB ケーブルで PC につなぐ
2. XIAO（マイコン）のリセットボタンをすばやく2回押す（`XIAO-SENSE` というドライブが表示される）
3. `zmk.uf2` を `XIAO-SENSE` ドライブにコピーする

macOS の場合は、`cline46` フォルダで次のコマンドを実行するとコピーできます。

```bash
# 右手側
cp zmk-ws/build/right/zephyr/zmk.uf2 /Volumes/XIAO-SENSE/

# 左手側
cp zmk-ws/build/left/zephyr/zmk.uf2 /Volumes/XIAO-SENSE/
```

コピーが終わると XIAO が自動で再起動し、ディスクが取り外されます。

電源を入れると新しいファームウェアが反映されいます。
