# Kokkos　コア文書ウェブサイト
Kokkos ドキュメンテーションレポジトリにようこそ。  これは、 https://kokkos.github.io/kokkos-core-wiki/.のソースです。

## ローカル環境で　HTML　ページを作成するための要件

これはドキュメントのローカルレンダリング専用に必要なもので、プッシュ前に確認できるようになります。
要件は、 `build_requirements.txt`にあります。
以下を使って、インストール可能です: `pip install -r build_requirements.txt`

## ビルド

```
cd docs
make html
```

クリーンのため:
```
cd docs
コードをきれいにする
```

## サイトをローカルで表示

ウェブブラウザで、`docs/generated_docs/index.html` を開くことが可能で、または、その代わりに、python　に組み込まれた  http サーバーを使用できます:

```bash
cd docs/generated_docs
python3 -m http.server
```

次に、 http://localhost:8000　に移動してください。

その代わりに、あるいは、make　を実行するたびに自動更新を希望する場合は、[httpwatcher](https://pypi.org/project/httpwatcher/) と組み合わせて、ドキュメンテーションを利用できます。
