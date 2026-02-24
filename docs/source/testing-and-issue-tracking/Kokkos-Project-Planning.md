# Kokkosプロジェクトプラニング

## 要件定義

コアプロジェクトには、4つの要件カテゴリが存在します:

- 安定した、十分にテスト済みの　API　を提供し、互換性の問題を回避
- 導入時点において、関連するすべてのコンピューティングプラットフォームをサポート
- パフォーマンスの移植性を実現するプログラミングモデルの機能を提供
- 将来の　ISO C++　機能への導入経路の可能化

Kokkos API　の安定性という、別途の包括的な要件があります。

関連する具体的な実行可能なタスクはすべて、GitHubの問題およびプルリクエストにて記録および追跡されています。

### Kokkos API 安定性

堅牢性とAPIの安定性は、テスト駆動開発および既存機能の明示的な非推奨化と削除プロセスを通じて、保証されます。

既存の機能が、時代遅れ、または有用でなくなったと判断された場合、Kokkos　チームはその機能を非推奨とし、次期メジャーリリース（3年ごとに発生）での削除対象としてマークいたします。
さらに、Kokkos　の設定オプションにより、非推奨機能の実質的な削除が可能となり、顧客がそれらを信頼しているかどうかのテストを可能にします。

非推奨機能の廃止サイクルでは、ユーザーに対し最低6か月間の警告期間を設けております。
廃止の段階において、お客様からのフィードバックにより廃止の決定を見直すことが可能です。

#### この要件を支援するための活動:

- 偶発的な障害が発生しないことを保証するために、既存の機能に対して完全なテストを提供
- 機能の継続的な有用性と根本的な欠陥を評価
- 非推奨/廃止の過程にある機能には、非推奨としてタグ付け
- 非推奨機能は、メジャーリリースバージョン変更時のみ削除

### プラットフォームサポート

Kokkos　プロジェクトの主な要件は、現行および将来のコンピューティングプラットフォーム向けに、堅牢なパフォーマンス・移植性ソリューションを提供することです。
目標は、システム間のコードのシームレスな移行を可能にし、既存の
Kokkos　ベースのコードが希望するコンピューティングプラットフォームを活用できない状況を回避することです。

この要件を満たすため、Kokkosチームは、お客様による実地試験が行われる前に、新たなハードウェアプラットフォームを予測する必要があります。
Kokkos　プロジェクトでは、更新されたソフトウェアスタック（コンパイラ、実行時ライブラリ）の機能性についても、
Kokkosチームが利用可能になり次第（理想的には顧客プラットフォームへの展開前）、プラットフォーム上で検証する必要があります。

したがって、Kokkosチームは、ハードウェアベンダーと共同設計の取り組みを、独立して、また資金提供機関のシステム調達活動と連携して、進める必要があります。

#### 本要件をサポートする活動:

- 施設システムの調達活動に参加
- ベンダーからのシステムソフトウェアスタックのリリースを監視 (AMD, Intel, NVIDIA, HPE)
- ベンダーと契約し、リリース前のソフトウェア開発キットを用いたKokkosのテスト実行を可能化
- 必要に応じて新しいテストシステムを導入
- 新しいソフトウェアスタックに対応するため、テストプロセスを更新

### プログラミングモデルの機能

Kokkos　プロジェクトの要件は、お客様からのご要望と、Kokkosチームメンバーによる調査活動の両方から収集されます。

顧客の要望は、Kokkos Slackチャンネル、GitHubの課題、ハッカソン、およびユーザーグループミーティングを通じて収集されます。
機能リクエストを担当する Kokkos チームメンバーは、ユースケースの詳細を収集し、
その機能の一般的な適用可能性について初期評価を行います.

調査結果は、Kokkos　開発者会議で報告および議論され、当該機能がロードマップに組み込まれるか否かの判断が可能となります。
機能に関する議論は、公開のGitHubの問題にて、記録および追跡が行われます。

Kokkos チームのメンバーによる新たな機能要件は、個別の研究活動において開発されます。これらの活動では、機能性、
ユースケース、および一般的な適用可能性について調査が行われます。
その後、それらは、Kokkosチーム全体に提示され、本プロジェクトへの組み込みについて、議論されます。
これらの議論を経て、機能の配置先について、その機能が主要なコアパッケージに組み込むに値する重要性を持つかどうか、あるいはKokkos GitHub　組織内の独自リポジトリに独立したライブラリとして配置すべきかどうかについて、決定されます。

#### 本要件をサポートする活動:

- 新機能のリクエストについて、Slackチャンネルと　GitHub　の課題管理システムを監視
- HPCコミュニティが主催するHackathonsに参加
- 年2回の　Usergroup　会議を開催
- 開発者会議において提案された機能について議論し、ロードマップへの組み込みを検討

### ISO C++ 互換性

Kokkos　における第三の要件は、将来の　ISO C++　規格への移行経路を提供するとともに、規格の方向性に影響を与えることです。
本要件は、Kokkos　の機能を　ISO C++　に組み込むことを可能にし、
長期的に　C++　実装者コミュニティ全体と保守負担を分担することで、Kokkos　の長期的な持続可能性目標に貢献します。

オンランプを有効化するため、Kokkos　は適切な場合および必要に応じて、ISO C++　の機能を過去の　C++　標準へバックポートいたします。
Kokkos　は、GPU　上で動作する　ISO C++　機能の拡張も提供しますが、これはデフォルトでは利用できない機能です。

Kokkos　の機能のうち、実績が証明され、幅広いユーザー層の関心を集めているものは、ISO C++　標準への採用可能性について評価されます。
Kokkos チームは、適切な時期に　ISO C++　委員会に向けて提案書を作成いたします。

ISO C++　標準に機能が組み込まれた場合、Kokkos　チームは、将来の　C++　標準で提供される　API　のバリエーションを
可能な限り、現在　Kokkos　がサポートしているソフトウェアスタック上で、利用可能とする予定です。

#### 本要件をサポートする活動:

- ISO C++ 委員会会議に参加
- Kokkos　が提供する　ISO C++　機能に関する要望を監視
- 成熟したKokkos　の機能で幅広い適用性を持つものについて、ISO C++　向けの提案書を作成
- Kokkos　がサポートする規格に、関連する将来の　ISO C++　機能をバックポート

## 公開計画

Kokkos　のリリースは、"catch the train" モデルに基づいています。つまり、主な目標は定期的なリリースを実現することであり、
各リリースごとに特定の機能リストを用意することではありません。

メジャーリリースは3年ごとに実施され、マイナーリリースは3～4か月ごとに実施されることを目指しており、必要に応じて追加のパッチリリースも行います。

メジャーリリースとマイナーリリースの主な違いは、非推奨機能がメジャーリリース時のみ削除される点と、
メジャーリリースではコンパイラの最小バージョン要件が引き上げられ、ISO C++標準の最小バージョンが更新される点です。
それ以外には、メジャーリリースとマイナーリリースの計画と実行に違いはありません。

メジャーリリースやマイナーリリースとは異なり、パッチリリースには通常、バグ修正のみが含まれ、新機能の追加は含まれません。

リリースサイクルの開始時に、Kokkos Core　のリーダーシップチームは、当該リリースサイクルにおける優先度の高い重点分野を決定いたします。
さらに、各チームメンバーは、リリースサイクルにおける自身の優先事項リストを作成いたします。.
優先順位については、Kokkos　開発者会議で議論され、精査された後、内部文書にまとめられます。

各項目に関する課題は、チームメンバーの割り当てを含め、[Kokkosプロジェクト計画](https://github.com/orgs/kokkos/projects/1) に割り当てられております。

[Kokkos project plan](https://github.com/orgs/kokkos/projects/1) は、7カテゴリーのうちの1つに、問題を割り当てます:

- *未割り当て:*  チームメンバーにまだ割り当てられていない課題。
- *未割り当て - 優先事項:* チームメンバーにまだ割り当てられていませんが、優先度が高い課題。これらの問題は、次回の週例開発者会議にて、割り当てられる必要があります。
- *実装予定:* 課題はチームメンバーに割り当てられているが、現時点では積極的には取り組まれていません。
- *実装予定 - 優先事項:* 課題はチームメンバーに割り当てられているが、現時点では積極的には取り組まれていません。 担当者の次の作業項目として予定されてます。次回の開発者会議までに、この項目が　*対応中*　の状態に移行しない場合、担当者の再割り当てが検討されます。
- *対応中:* 課題については、現在対応中です。
- *対応中 - 優先事項:* 課題については、現在対応中です。 本件に関するコードレビューは、早急に解決を図るため、優先的に実施されるものとします。
- *完了:* 課題はマージされたプルリクエストにより処理されました。あるいは、新たな情報により不要となったためクローズされました。 マージされたプルリクエストについては、プロジェクト計画から項目を削除する前に、必要に応じて、変更履歴エントリが生成されていることを確認します。


## 課題の優先順位付け

課題の優先順位付けは、二つの方法によって行われます:
- Kokkos リーダーシップ会議
- Kokkos 開発者全体会議。

リーダーシップ会議は毎週月曜日に開催されます。
それは、複数の目的を果たします:
- 週の緊急対応事項を決定
- 新規課題リストを確認し、重要度の優先順位付けを実施
- Kokkos の計画項目について検討
- 新規活動項目についての予備的なチーム割り当てを実施
- 開発者会議議題の草案を作成

[Kokkos project plan](https://github.com/orgs/kokkos/projects/1)　内に項目の優先順位付けを記録します。

会議の議事録は、非公開のリポジトリに保管されています: [internal repository](https://github.com/kokkos/internal-documents)

さらに優先順位付けが行われるのは、下記の開発者会議においてです。

## 開発者の調整

チームは主に、Slack　の　#nucleus　チャネルで連絡を取っています。
メンバーは、[Slack](https://kokkosteam.slack.com)　にご参加後、クリスチャンまたはダミアンにより追加されます。
開発者同士は、公開の会話と非公開の会話の両方を行うことができます。
彼らは、あまり詳しくないコードの部分について質問する、あるいは
進行中の問題についてフィードバックを求めることができます。
Slack上での会話は一時的なものとみなされます。 90日以上経過したメッセージは削除されます（無料プランの場合）。 
何かを長期的に参照可能にする必要がある場合は、適切な場合であれば、GitHub上で議論する必要があります。
個人情報は、[内部リポジトリ](https://github.com/kokkos/internal-documents)　に保管される場合がありますが、NDAデータは、そこには投稿しないでください。

2pm ET / 12 pm MT / 18:00 UTC  Zoom上で、週一回毎水曜日に開催される、Kokkos 開発者会議 
議題は、事前に社内リポジトリに掲載されています（[`meeting-notes/`](https://github.com/kokkos/internal-documents/tree/master/meeting-notes/2023) ディレクトリ内で確認できます）。
開発者の方々は、議題を編集し、会議で議論したいトピックや課題を追加することが可能です。

## リリースプロセス

リリースプロセスは、6つのステップで構成されています:

- リリース候補ブランチを作成
- perform integration tests with release candidate
- resolve issues and cherry-pick fixes to release candidate
- check Changelog
- tag a release
- conduct release briefing for user community

When nearing a desired release date, the release candidate branch will be created from the Kokkos develop branch.
Before creating the release candidate, possible delay reasons will be discussed at the developer meeting.
This could include important bug fixes, or an important feature being in the last phase of code review,
but is generally done under exceptional circumstances.
Furthermore, merging major new features into the development branch may be delayed until after the creation
of the release candidate.
This ensures that major new features have a period of testing in the develop branch before they are shipped.

After creating the release candidate branch integration testing is started.
This includes internal testing by the Kokkos team with selected customer codes, as well as partnering
with some primary customers who will try the release candidate in their testing processes.

The release candidate creation is also announced on the Slack channel, inviting the general Kokkos
user community to test it, and provide feedback.

Defect reports (both functionality and performance) are collected as GitHub issues and marked with
"Blocks Promotion".
These items are then assigned to Kokkos team members at highest priority.

Defect resolutions are merged into the develop branch first, and then cherry picked onto the
release candidate branch, ensuring that no regression remains unaddressed on the primary development
branch.

Upon resolution of all defect reports the release candidate branch is used to create a GitHub release tag,
after checking and merging the Changelog.

After the release is created a Release Briefing date is set approximately two to three weeks after the release,
providing an overview of new capabilities to users.
The release briefing also serves as an additional point for feedback collection.

