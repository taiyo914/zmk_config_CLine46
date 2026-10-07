# zmk-config-CLine46

CLine46 の ZMK ファームウェアの設定です。

## ローカルでビルドする

GitHub Actions を使わずに、手元の PC でビルドする手順です。Docker が使える状態になっている必要があります。

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

`zmk-config` フォルダに移動します。

```bash
cd zmk-config
```

次のコマンドで、コンテナの起動からビルドまでをまとめて行います。

```bash
make build-right  # 右手側
make build-left   # 左手側
make build-both   # 左右両方
```

ビルドしたファームウェアは次の場所にできます。

- 右手側: `zmk-ws/build/right/zephyr/zmk.uf2`
- 左手側: `zmk-ws/build/left/zephyr/zmk.uf2`

`CLine46.keymap` や右手の設定ファイルのみを変更した場合は、右手側だけをビルドして書き込めば反映されます。

左手の設定ファイルや `config/west.yml` などを変更したときは、左手側もビルドして書き込んでください。

また、リセットしたいときは `settings_reset` を両方に書き込み、そのあとで左右両方にファームウェアを書き込み直します。

### 3. 書き込み

1. 書き込む側を USB ケーブルで PC につなぐ
2. `zmk-config` フォルダで書き込む側のコマンドを実行する

    ```bash
    make flash-right  # 右手側
    make flash-left   # 左手側
    ```

3. コマンドが `XIAO-SENSE` ドライブが表示されるのを待つので、書き込む側で次のどちらかの操作をする
    - `&bootloader` キーを押す
    - XIAO（マイコン）のリセットボタンをすばやく2回押す
4. `XIAO-SENSE` ドライブが表示されると、コマンドが `zmk.uf2` をドライブにコピーする

コピーが終わると XIAO が自動で再起動し、ドライブが取り外されます。電源を入れると新しいファームウェアが反映されています。

`&bootloader` キーが入っていないファームウェアを使っているときは、リセットボタンを使ってください。

### 4. ビルドと書き込みを続けて行う

`zmk-config` フォルダで次のコマンドを実行すると、ビルドが成功したあと、そのまま「3. 書き込み」の手順 3 に進みます。

```bash
make build-flash-right  # 右手側
make build-flash-left   # 左手側
```

### 5. コマンドの一覧

`make help` を実行すると、使えるコマンドの一覧が表示されます。
