---
title: 実際のユーザー環境で Web Vitals を測定するためのベスト プラクティス
subhead: 現在ご利用中のアナリティクス ツールを使用して Web Vitals を測定する方法。
authors:
  - philipwalton
description: 現在ご利用中のアナリティクス ツールを使用して Web Vitals を測定する方法
date: 2020-05-27
updated: 2020-07-21
hero: image/admin/WNrgCVjmp8Gyc8EbZ9Jv.png
alt: 現在ご利用中のアナリティクス ツールを使用して Web Vitals を測定する方法
tags:
  - blog
  - performance
  - web-vitals
---

実際の環境でのページのパフォーマンスを測定してレポートする機能は、パフォーマンスを診断し、長期的に改善していくにあたっては不可欠な要素です。[フィールド データ](/user-centric-performance-metrics/#in-the-field)がなければ、サイトに加えた変更が実際に望ましい結果をもたらしているかどうかを確認することができません。

人気のある[リアル ユーザー モニタリング (RUM)](https://en.wikipedia.org/wiki/Real_user_monitoring) アナリティクス プロバイダーの多くが、提供するツールですでに (数多くの[その他の Web Vitals](/vitals/#core-web-vitals) と同様に) [Core Web Vitals](/vitals/#other-web-vitals) 指標をサポートしています。こういった RUM アナリティクス ツールを現在ご利用中であれば、運営するサイト内のページが[推奨されている Core Web Vitals のしきい値](/vitals/#core-web-vitals)をどの程度満たしているかを評価し、将来的なパフォーマンスの低下を防止するという観点では申し分のない環境であると言えるでしょう。

Core Web Vitals 指標をサポートしているアナリティクス ツールの使用をお勧めはしているものの、現在ご利用中のアナリティクス ツールがそれらをサポートしていないような場合でも、必ずしも使用するツールを切り替える必要はありません。ほとんどのアナリティクス ツールでは、[カスタム指標](https://support.google.com/analytics/answer/2709828)や[イベント](https://support.google.com/analytics/answer/1033068)を定義して測定する方法が提供されています。つまり、現在ご利用中のアナリティクス プロバイダーを使用しながらでも Core Web Vitals 指標を測定し、既存のアナリティクス レポートやダッシュボードへとそれらを追加できる可能性があるのです。

このガイドでは、Core Web Vitals 指標 (またはカスタム指標) をサードパーティ製ツールや社内のアナリティクス ツールを用いて測定するためのベスト プラクティスについて説明を行います。また、自社のサービスに Core Web Vitals のサポートを追加したいと考えているアナリティクス ベンダー向けのガイドとしてもご利用いただけます。

## カスタム指標またはイベントを使用する

前述したように、ほとんどのアナリティクス ツールでカスタム データの測定を行うことができます。ご利用のアナリティクス ツールがこの機能をサポートしていれば、このメカニズムを使用して Core Web Vitals の各指標を測定することができるはずです。

アナリティクス ツールを使用してカスタム指標やイベントを測定するためには、一般的に以下の 3 つの手順に従います。

1. ツールの管理画面でカスタム指標の[定義または登録](https://support.google.com/analytics/answer/2709829?hl=en&ref_topic=2709827)を行います (必要な場合)。*(注: すべてのアナリティクス プロバイダーでカスタム指標を事前に定義する必要があるわけではありません。)*
2. フロントエンドの JavaScript コードで指標の値を計算します。
3. 指標の値をアナリティクスのバックエンドへと送信し、名前または ID が手順 1 で定義したものに一致することを確認します *(必要な場合)*。

手順 1 と 3 については、ご利用のアナリティクス ツールのドキュメンテーションを参照してください。手順 2 については、[web-vitals](https://github.com/GoogleChrome/web-vitals) JavaScript ライブラリを使用して Core Web Vitals の各指標の値を計算することも可能です。

次のコード サンプルでは、コードでこれらの指標を追跡し、アナリティクス サービスへと送信することがいかに簡単かを示しています。

```js
import {getCLS, getFID, getLCP} from 'web-vitals';

function sendToAnalytics({name, value, id}) {
  const body = JSON.stringify({name, value, id});
  // 利用可能であれば `navigator.sendBeacon()` を使用し、`fetch()` にフォールバックします。
  (navigator.sendBeacon && navigator.sendBeacon('/analytics', body)) ||
      fetch('/analytics', {body, method: 'POST', keepalive: true});
}

getCLS(sendToAnalytics);
getFID(sendToAnalytics);
getLCP(sendToAnalytics);
```

## 分布のレポート方法

カスタム指標やイベントを使用して Core Web Vitals の各指標の値を計算してアナリティクス サービスへと送信したら、次の手順として、収集した値を表示するレポートやダッシュボードの作成を行います。

[推奨される Core Web Vitals のしきい値](/vitals/#core-web-vitals)を満たされていることを確認するには、レポートに各指標の 75 パーセンタイルの値を表示する必要があります。

ご利用のアナリティクス ツールに分位数レポートが組み込まれていない場合には、すべての指標の値を昇順に並べ替えたレポートを作成することによって、このデータを手動で取得することができます。このレポートが生成されたら、レポートに含まれているすべての並べ替えられた値の完全なリストの中で 75% に相当する値を探します。その値こそが、75 パーセンタイルの値です。この方法は、データをどのようにセグメント化したとしても (デバイスの種類、接続の種類、国など) 同じです。

ご利用のアナリティクス ツールが指標レベルのレポート粒度をデフォルトで提供していない場合でも、アナリティクス ツールが[カスタム ディメンション](https://support.google.com/analytics/answer/2709828)をサポートしていれば、同じ結果を得ることができるはずです。レポート構成にカスタム ディメンションを含めれば、追跡する個々の指標インスタンスごとに固有のカスタム ディメンション値を設定することで個々の指標インスタンスごとに分類されたレポートを生成することができるはずです。各インスタンスには固有のディメンション値があるため、グループ化は行われません。

この手法の例としては、Google Analytics を利用した [Web Vitals Report](https://github.com/GoogleChromeLabs/web-vitals-report) が挙げられます。このレポートのコードは[オープン ソース](https://github.com/GoogleChromeLabs/web-vitals-report)化されているため、開発者の方がこのセクションで説明した技術のサンプルとしてこれらを参照することが可能です。

<img src="https://user-images.githubusercontent.com/326742/101584324-3f9a0900-3992-11eb-8f2d-182f302fb67b.png" no="" alt="Web Vitals Report のスクリーンショット">

{% Aside %}ヒント: [`web-vitals`](https://github.com/GoogleChrome/web-vitals) JavaScript ライブラリでは、レポートされた各指標インスタンスに ID を提供しており、これによってほとんどのアナリティクス ツールで分布を構築できるようにしています。詳細については、[`Metric`](https://github.com/GoogleChrome/web-vitals#metric) (指標) インターフェースのドキュメンテーションを参照してください。{% endAside %}

## 適切なタイミングでデータを送信する

パフォーマンス指標の中には、ページの読み込みが完了した時点ですぐに計算できるものもあれば、CLS のようにページの表示期間全体を考慮し、ページのアンロードが開始された時点で初めて確定するものもあります。

ただし、`beforeunload` と `unload` イベントの両方が (特にモバイル環境では) 信頼性が低く、その使用が[推奨されていない](https://developer.chrome.com/blog/page-lifecycle-api/#legacy-lifecycle-apis-to-avoid)ため (ページが [Back-Forward Cache](https://developer.chrome.com/blog/page-lifecycle-api/#what-is-the-back-forward-cache) の対象となることを妨げてしまう可能性があるため)、これが原因となって問題を引き起こしてしまう可能性があります。

ページの表示期間全体を追跡する指標の場合には、ページの可視性の状態が `hidden` に変化したときには、指標の現在の値にかかわらず `visibilitychange` イベントの発生中に送信を行うことが最善の処理となります。これは、ページの可視性の状態が `hidden` に変更された時点で、そのページのスクリプトが再度実行可能となる保証がないからです。これは、ページのコールバックが発生していない状態でブラウザー アプリそのものを閉じることができるモバイル OS において特に顕著に見られます。

なお、一般的にモバイル OS では、タブを切り替えたり、アプリを切り替えたり、ブラウザー アプリそのものを閉じたりした場合に `visibilitychange` イベントが発生します。また、タブを閉じた場合や新しいページに移動した場合にも `visibilitychange` イベントが発生します。そのため、`visibilitychange` イベントは `unload` や `beforeunload` イベントよりもはるかに信頼性が高くなります。

{% Aside 'gotchas' %}[いくつかのブラウザのバグ](https://github.com/w3c/page-visibility/issues/59#issue-554880545)により、`visibilitychange` イベントが発生しない場合もあります。独自のアナリティクス ライブラリを構築している場合には、これらのバグにご注意ください。なお、[web-vitals](https://github.com/GoogleChrome/web-vitals) JavaScript ライブラリでは、それらのバグをすべて明らかにすることが可能です。{% endAside %}

## パフォーマンスを経時的に監視する

アナリティクスの実装を更新し、Core Web Vitals 指標の追跡およびレポートが可能になったら、次の手順として、サイトの変更がパフォーマンスに及ぼす影響を経時的に追跡していきましょう。

### 変更内容のバージョン管理を実施する

変更内容を追跡するにあたっての素朴な (そして最終的には信頼ができない) アプローチとして、変更内容を本番環境にデプロイし、デプロイ日以降に受信した指標データがすべて新しいサイトに対応し、デプロイした日以前に受信した指標データがすべて旧サイトに対応していると仮定するアプローチが挙げられます。しかしながら、様々な要因 (HTTP、Service Worker、CDN 層でのキャッシングを含む) により、このアプローチが機能しなくなる可能性があります。

より良いアプローチとしては、デプロイされた変更ごとに固有のバージョンを作成し、そのバージョンをアナリティクス ツールで追跡する方法があります。ほとんどのアナリティクス ツールではバージョンの設定がサポートされています。ご利用のツールが対応していない場合には、カスタム ディメンションを作成し、そのディメンションをデプロイされたバージョンに設定することができます。

### 実験を行う

バージョン管理をさらにもう一歩前進させ、複数のバージョン (または実験) を同時に追跡することができます。

ご利用のアナリティクス ツールで実験グループを定義できる場合には、その機能をご利用ください。そうではない場合には、カスタム ディメンションを使用することで、レポートの中で各指標の値が特定の実験グループに関連付けられるように設定することができます。

アナリティクスで実験を行うことにより、一部のユーザーに対して実験的な変更を適用し、その変更適用時のパフォーマンスをコントロール グループのユーザーによるパフォーマンスと比較することができるようになります。その変更によって確実にパフォーマンスが改善されたという確証が得られた場合には、その変更内容をすべてのユーザーに対して展開することができます。

{% Aside %}実験グループについては、必ずサーバー上で設定するようにしてください。クライアント側で動作する実験ツールや A/B テスト ツールの使用は避けてください。一般的にこういったツールはユーザーの実験グループが決定されるまでの間レンダリングをブロックするため、LCP の値に悪影響を及ぼす可能性があります。{% endAside %}

## 測定がパフォーマンスに影響を与えないようにする

実際のユーザーのパフォーマンスを測定する際には、実行するパフォーマンス測定コードがページのパフォーマンスに対して絶対に悪影響を及ぼさないようにすることが重要です。もしも影響を与えてしまっているとすれば、パフォーマンスがビジネスに与える影響についてあなたが導き出そうとしている結論は信頼性を失ってしまいます。これは、アナリティクス コードそのものが最大の悪影響を及ぼしている可能性を否定することができないためです。

RUM アナリティクスのコードを本番環境にデプロイする場合には、必ず以下の原則に従ってください。

### アナリティクスを先送りする

アナリティクス コードは常に非同期かつブロックが発生しない方法で読み込む必要があり、一般的には一番最後に読み込む必要があります。ブロックが発生する方法でアナリティクス コードを読み込んでしまうと、LCP に悪影響を及ぼす可能性があります。

Core Web Vitals の測定に使用されるすべての API は ([`buffered`](https://www.chromestatus.com/feature/5118272741572608) フラグを介して) 非同期または遅延型のスクリプト読み込みをサポートするよう特別に設計されているため、スクリプトの読み込みタイミングを無理に早める必要はありません。

ページの読み込みタイムラインの後半では計算ができないような指標を測定する場合には、早いタイミングで実行する必要のあるコード*のみ*を ([レンダリングを妨げるリクエスト](https://developer.chrome.com/docs/lighthouse/performance/render-blocking-resources/)にならないように) ドキュメントの `<head>` 内にインライン化し 、残りの部分を先送りします。必要な指標が 1 つあるという理由だけで、すべてのアナリティクスを早いタイミングで読み込まないようにしてください。

### 長く時間がかかっているタスクを作らない

アナリティクス コードはしばしばユーザーの入力に応じて実行されますが、アナリティクス コードが多数の DOM 測定を行っていたり、その他のプロセッサに負荷がかかる API を使用していたりすると、アナリティクス コード自体が入力に対する応答性を低下させる原因となってしまいます。また、アナリティクス コードを含む JavaScript ファイルのサイズが大きい場合、そのファイルの実行によりメイン スレッドがブロックされ、FID に悪影響を及ぼしてしまいます。

### ノンブロッキング API を使用する

 <code>[sendBeacon()](https://developer.mozilla.org/docs/Web/API/Navigator/sendBeacon)</code> や <code>[requestIdleCallback()](https://developer.mozilla.org/docs/Web/API/Window/requestIdleCallback)</code> などの API は、ユーザーにとって重要なタスクをブロックすることなく重要ではないタスクを実行できるように、特別に設計されています。

これらの API は、RUM アナリティクス ライブラリでの使用に最適なツールです。

一般的に、すべてのアナリティクス ビーコンは `sendBeacon()` API を使用して送信される必要があります。また、すべての受動的なアナリティクス測定コードは、アイドル期間中に実行される必要があります。

{% Aside %}アイドル時間を最大限に活用しつつ、必要なとき (ユーザーがページをアンロードするときなど) にコードを緊急実行できるようにする方法のガイダンスについては、idle-until-urgent パターンに関する[こちらの記事](https://philipwalton.com/articles/idle-until-urgent/)を参照してください。{% endAside %}

### 必要以上に追跡しない

ブラウザーは多くのパフォーマンス データを提供してくれますが、データが利用可能だからといって必ずしもそれらを記録し、アナリティクス サーバーへと送信する必要はありません。

たとえば [Resource Timing API](https://w3c.github.io/resource-timing/) では、ページに読み込まれるすべてのリソースについての詳細なタイミング データが提供されています。しかしながら、それらのデータすべてがリソースの読み込みパフォーマンスの改善に必要であったり、役に立ったりするとは考えられません。

要するに、データがそこにあるからという理由でただ追跡するのではなく、データを追跡するためのリソースを消費する前に、そのデータが使用されることを確認する必要があるのです。
