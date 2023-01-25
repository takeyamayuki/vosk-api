# Vosk Speech Recognition Toolkit

VOSK Cの検証を行うリポジトリ．

## 使い方

1. kaldiのインストール

```bash
# kaldhi build
$ sudo su

# travis/Dockerfile.manylinuxと同じです．
$ cd /opt \
    && git clone -b vosk --single-branch https://github.com/alphacep/kaldi \
    && cd /opt/kaldi/tools \
    && git clone -b v0.3.20 --single-branch https://github.com/xianyi/OpenBLAS \
    && git clone -b v3.2.1  --single-branch https://github.com/alphacep/clapack \
    && make -C OpenBLAS ONLY_CBLAS=1 DYNAMIC_ARCH=1 TARGET=NEHALEM USE_LOCKING=1 USE_THREAD=0 all \
    && make -C OpenBLAS PREFIX=$(pwd)/OpenBLAS/install install \
    && mkdir -p clapack/BUILD && cd clapack/BUILD && cmake .. && make -j 10 && find . -name "*.a" | xargs cp -t ../../OpenBLAS/install/lib \
    && cd /opt/kaldi/tools \
    && git clone --single-branch https://github.com/alphacep/openfst openfst \
    && cd openfst \
    && autoreconf -i \
    && CFLAGS="-g -O3" ./configure --prefix=/opt/kaldi/tools/openfst --enable-static --enable-shared --enable-far --enable-ngram-fsts --enable-lookahead-fsts --with-pic --disable-bin \
    && make -j 10 && make install \
    && cd /opt/kaldi/src \
    && ./configure --mathlib=OPENBLAS_CLAPACK --shared --use-cuda=no \
    && sed -i 's:-msse -msse2:-msse -msse2:g' kaldi.mk \
    && sed -i 's: -O1 : -O3 :g' kaldi.mk \
    && make -j $(nproc) online2 lm rnnlm \
    && find /opt/kaldi -name "*.o" -exec rm {} \;

$ exit
```

2. Voskのインストール
```
# build vosk
$ cd workspce # 任意のディレクトリ
$ git clone https://github.com/alphacep/vosk-api
$ cd vosk-api/src
$ nano Makefile
```
MakefileのKALDI_ROOTを以下のように変更する．
```Makefile
KALDI_ROOT?=/opt/kaldi/
```

```bash
# src内のビルド
$ make 

# testコードのビルド
$ cd ../c
$ make 
```

3. テストコードの実行

```bash
$ ./test_vosk
```

出力例

```bash
{
  "result" : [{
      "conf" : 1.000000,
      "end" : 4.770000,
      "start" : 4.500000,
      "word" : "今日"
    }, {
      "conf" : 1.000000,
      "end" : 4.920000,
      "start" : 4.770000,
      "word" : "も"
    }, {
      "conf" : 0.998977,
      "end" : 5.160000,
      "start" : 4.950000,
      "word" : "一"
    }, {
      "conf" : 0.999090,
      "end" : 5.460000,
      "start" : 5.190000,
      "word" : "日"
    }, {
      "conf" : 0.993345,
      "end" : 6.090000,
      "start" : 5.490000,
      "word" : "がんばろう"
    }],
  "text" : "今日 も 一 日 がんばろう"
}
```


## メモ
以下の２行を追加して，認識結果を出力する．
検出単語，コンフィデンス，認識始まり，認識終わりの時間を出力する．

```c
vosk_recognizer_set_partial_words(recognizer,1);
vosk_recognizer_set_words(recognizer,1);
```
