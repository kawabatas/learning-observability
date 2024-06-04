# learning-observability
オブザーバビリティについての説明資料

### 推測するな、計測せよ
>ルール1: プログラムがどこで時間を消費することになるか知ることはできない。ボトルネックは驚くべき箇所で起こるものである。したがって、どこがボトルネックなのかをはっきりさせるまでは、推測を行ったり、スピードハックをしてはならない。
ルール2: 計測すべし。計測するまでは速度のための調整をしてはならない。コードの一部が残りを圧倒しないのであれば、なおさらである。

By Rob Pike (引用元 [wiki/UNIX哲学](https://ja.wikipedia.org/wiki/UNIX%E5%93%B2%E5%AD%A6))

### オブザーバビリティ (可観測性) とは
- システムの状態を外部から観測し、理解する能力を指します。具体的には、システムの動作状況、パフォーマンス、内部状態などを把握できることを意味します。
- :memo: Go言語による分散サービスの一文を紹介

### オブザーバビリティの 3本柱
- ログ
- メトリクス
>メトリックスは、マクロな統計情報。だいたい平均は100mSで終わってる、80パーセンタイルの処理時間は120mS、95パーセンタイルは400mSみたいな、「なんとなく」を掴む方法。メモリ使用量、CPUなどのシステムの状態を把握し、異常が起きそうかどうか、余裕があるかどうかを把握するために使うもので、SRE本ではかなり強調されているもの。
- トレース
>トレーシングはどの処理がどの順番で行われているか、どのぐらい処理時間がかかっているかをミクロに見ていくことによって、システムの状況を把握しやすくする。

(引用元 [OpenCensus(OpenTelemetry)とは](https://future-architect.github.io/articles/20190604/) 一読を進める)
([OpenTelemetryとgo-chiを繋げてみる](https://future-architect.github.io/articles/20211020a/) 続編のこちらも一読を進める)

### [OpenTelemetry](https://opentelemetry.io/docs/what-is-opentelemetry/) とは
>従来の3本柱ではメトリクス、ログ、トレースが別々のシステムとして扱われ、データが分断されているため相関関係を見つけるのが難しくなっています。
>
>OpenTelemetryではメトリクス、ログ、トレースを一つの一貫したグラフにまとめ、あらゆるテレメトリーのあらゆる形態を相関させることで、統一された分析を可能にし、遠く離れた重要な関連性をも見つけ出すことを可能にします。
>
>OpenTelemtryはメトリクス、ログ、トレースのようなテレメトリーデータを作成し、管理するために設計されたオブザーバビリティの標準仕様であり、ツールキットです。
OpenTelemetryでは、仕様に基づいて各言語向きライブラリが実装されており、アプリケーションやシステムを、その言語やインフラストラクチャー、ランタイム環境に関係なく、簡単に計装することができます。
OpenTelemetryではオブザーバビリティを①テレメトリーの作成と転送②分析の２つの段階に分けた時の、①テレメトリーの作成と転送を行うための各種ツール、API、SDKのコレクションを提供しています。

(引用元 [オブザーバビリティの最前線 OpenTelemetryで下げる認知負荷~活用事例4選~](https://findy-tools.io/articles/opentelemetry/12) 一読を進める)
([OpenTelemetryのこれまでとこれから](https://docs.google.com/presentation/d/e/2PACX-1vQdJTNtd8WLpGmL5KyaYzZtgmMYEroybE-dG-9FJ00mZ_a4-A_CXQhkoj2RBzFxIC2pSZHCgnLNseRZ/pub?resourcekey=0-tGqb05CaH1bETsKt11EdtA&slide=id.g48a57ebc11_0_0) こちらも一読を進める)

### Go のロガーをどうするか
>1.グローバル変数にloggerのインスタンスを入れておく
>
>2.contextにloggerのインスタンスを入れておく
>
>3.トレースIDなどを入れたloggerを適宜作ってcontextに格納する
>
>4.構造体のフィールドにloggerのインスタンスを入れておく（DI）

(引用元 [slog時代のGoではloggerをcontextで引きまわさなくて良い気がする](https://blog.arthur1.dev/entry/2024/01/19/093000))

その他参考
- [【OpenTelemetry】トレーサビリティを高めるLoggerを考えてみた【Go, Slog】](https://zenn.dev/yuki0920/articles/ee082adbea4769)
- [Go 1.21連載始まります＆slogをどう使うべきか](https://future-architect.github.io/articles/20230731a/)
- [なぜ Go ではロガーをコンストラクタ DI してはならないのか](https://zenn.dev/mpyw/articles/go-dont-inject-logger)

### 課題
各自のファイルアップローダーを観測できるようにする
```
                                          -----> Jaeger (trace)
App + SDK ---> OpenTelemetry Collector ---|
                                          -----> Prometheus (metrics)
```
- メトリクスを Prometheus で確認できるようにする
- トレースを Jaeger で確認できるようにする
  - HTTPリクエスト、MySQLにかかっている時間を計測可能にしたい
- アクセスログやアプリケーションログを標準出力に出力する
- ログとトレースをトレースIDで紐付ける

# (Other) References
- [A language-specific implementation of OpenTelemetry in Go](https://opentelemetry.io/docs/languages/go/)
```
Traces	Metrics	Logs
Stable	Stable	Alpha
```
- [OpenTelemetryの各言語の実装状況](https://github.com/open-telemetry/opentelemetry-specification/blob/main/spec-compliance-matrix.md)
- [Exporters](https://opentelemetry.io/docs/languages/go/exporters/)
>Send telemetry to the OpenTelemetry Collector to make sure it’s exported correctly. Using the Collector in production environments is a best practice.
- [open-telemetry/opentelemetry-go](https://github.com/open-telemetry/opentelemetry-go)
