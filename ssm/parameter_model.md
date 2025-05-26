## パラメータ管理の考え方

LINE Bot のように複数の外部サービスと連携する構成では、認証キーや API エンドポイントなどの機密情報を安全に管理する必要があります。
今回は AWS Systems Manager Parameter Store（以下 SSM）を使い、環境ごとに切り替え可能な形にしました。

## 管理している項目

以下のような値を SSM Parameter Store で管理しています。

- /cooksnap/{ENV}/LINE_CHANNEL_SECRET
- /cooksnap/{ENV}/LINE_CHANNEL_ACCESS_TOKEN
- /cooksnap/{ENV}/CLAUDE_API_KEY
- /cooksnap/{ENV}（環境識別用の文字列：dev, prod など）

名前空間を `/cooksnap/` としてまとめることで、他のプロジェクトと衝突しないようにしています。

## パラメータの種類とポリシー

- 全ての値は「SecureString」で保存し、KMS により暗号化
- 読み取りは Lambda 関数内で実行時に取得

## 本番環境とステージングの切り替え

ENV という変数を用意しておき、Lambda 関数の中で動的に SSM のパスを切り替えています。

```python
import os
ENV = os.environ.get("ENV", "dev")
ssm_path = f"/cooksnap/{ENV}/LINE_CHANNEL_SECRET"
```

このようにして、デプロイ環境によって読み取るパラメータを変えられるようにしています。

## 運用時の注意点

- パラメータの権限は Lambda の IAM ロールに最小限で許可
- 値の更新履歴は SSM 側で管理されるが、意図しない更新を防ぐため、更新手順は明示しておく
- 誤って SSM を削除しないよう、タグや保護設定を付ける運用も可能

## まとめ

SSM を使えば、機密情報をコードに書かずに管理できます。
また、ステージングや本番で異なる設定を切り替える際にも、構成を変えずに安全に反映できるので、個人開発でも導入して良かったと感じています。
