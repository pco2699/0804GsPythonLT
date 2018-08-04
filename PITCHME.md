### PythonでGRPCやってみた  

---
@snap[north]
@pco2699
@snapend
@snap[west]
![アイコン](assets/icon.jpg)
@snapend

@snap[east]
@size[20px](WealthNaviという会社でJavaエンジニア)
<br>
@size[20px](ジーズアカデミー DEVコース 7期卒業)
<br>
@size[20px](Python歴 : 2年ほど)

@snapend

---
@snap[north]
今回のお題目
@snapend
![アイコン](assets/python_book.jpg)

---

### めちゃくちゃ良書

---

### Pythonプロフェッショナルプログラミングのいいところ
- Pythonを公式言語として採用している株式会社ビープラウドさんが出版
- Pythonで実用に耐えうるDjangoアプリの設計方法やチーム開発の手法が載っている
- 要件定義->設計->開発までの一連の流れが書かれていて  他社さんの開発プロセスを知りたいエンジニアの方とかに非常におすすめ

---

### ただ、、、

---

### LTのネタとしては若干内容が発表しづらい

---

### 急遽、内容変更
### (本に興味ある方はぜひ、購入して読んでみてください!)

---

### PythonでgRPCしてみた

---

### What is gRPC?
- Googleが開発したAPI通信プロトコル
- REST API/JSONに代わるAPIの通信方法として最近注目されている
- JSONだと型が無いが、gRPCだと型をつけられて安心

---

### 今回つくるもの
![今回作るもの](assets/diagram1.png)

---

### 実際にPythonでgRPCを実装してみよう
1. .protoファイルを作成
1. grpc-toolsでPythonコードを生成
1. 生成されたコードからサーバー・クライアントのコードを生成
1. 動作確認

---

### .protoファイルの作成
.protoファイル=APIの仕様を決めるインターフェースのようなファイル
```
syntax = "proto3";

package helloworld;
// 実際にサーバで実装する処理のインプット・アウトプットを記載
service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}
// サーバ<->クライアントでやりとりするメッセージの仕様を記載
message HelloRequest {
  string name = 1;
}
// 上に同じ
message HelloReply {
  string message = 1;
}

```
---

### grpc-toolsでPythonコードを生成(1/2)
.protoファイルから実際にPythonコードを生成するためのGoogleお手製ツールをインストール
```
$ pip install grpcio-tools
$ pip install googleapis-common-protos
```

---

### grpc-toolsでPythonコードを生成(2/2)
インストールした.protoファイルからPythonコードを生成
```
$ python -m grpc_tools.protoc -Iproto --python_out=. --grpc_python_out=. protos/helloworld.proto
```

---

### 実際にインターフェースとしてPythonコードが生成されます
helloworld_pb2_grpc.py
```python
class GreeterServicer(object):
  # missing associated documentation comment in .proto file
  pass

  def SayHello(self, request, context):
    # missing associated documentation comment in .proto file
    pass
    context.set_code(grpc.StatusCode.UNIMPLEMENTED)
    context.set_details('Method not implemented!')
    raise NotImplementedError('Method not implemented!')

```

---

### 生成されたコードをimportして実装
サーバー側のコード
```
from concurrent import futures
import time

import grpc

# 生成したコードのインポート
import helloworld_pb2
import helloworld_pb2_grpc

_ONE_DAY_IN_SECONDS = 60 * 60 * 24

# .protoタイプで設定したインターフェースの内容をここで実装する
class Greeter(helloworld_pb2_grpc.GreeterServicer):
# 今回はシンプルにrequestからnameを取り出し、Hello, nameとReplyする
    def SayHello(self, request, context):
        print("name: " + request.name)
        return helloworld_pb2.HelloReply(message='Hello! %s!' % request.name)

# サーバーを立ち上げる処理
def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    helloworld_pb2_grpc.add_GreeterServicer_to_server(Greeter(), server)
    server.add_insecure_port('[::]:50051')
    server.start()
    try:
        while True:
            time.sleep(_ONE_DAY_IN_SECONDS)
    except KeyboardInterrupt:
        server.stop(0)


if __name__ == '__main__':
    serve()
```

---

### クライアントのコードを生成
クライアントのコード
```

from __future__ import print_function

import grpc

import helloworld_pb2
import helloworld_pb2_grpc


def run():
    # NOTE(gRPC Python Team): .close() is possible on a channel and should be
    # used in circumstances in which the with statement does not fit the needs
    # of the code.
    with grpc.insecure_channel('localhost:50051') as channel:
        # stubを作成 
        stub = helloworld_pb2_grpc.GreeterStub(channel)
        # stub経由でリクエストを生成しレスポンスを受け取る
        response = stub.SayHello(helloworld_pb2.HelloRequest(name='pco2699'))
    print("Greeter client received: " + response.message)


if __name__ == '__main__':
    run()
```
---

### デモ

---

### 実際にgRPCを実装してみて
- .protoファイルがAPIの仕様代わりとなるのがGood
- .protoファイルからいろんな言語の実装を作れるので、言語に依存しない！
- 型の効果はあんまり実感できなかったので、もっと複雑なアプリを作ってみたい！

---

### 現場からは以上です
