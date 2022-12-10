# このブログの対象読者

1. イーサリアム、Ethereum virtual machine(EVM)について知りたい方<br />
2. 暗号資産に触れたことがあり、背景で何が起こっているのか理解したい方<br />

# はじめに

[前々回のブログ](https://zenn.dev/hokusai/articles/ac26ef75b89d7b)では、イーサリアムのトランザクションについて、[前回のブログ](https://zenn.dev/hokusai/articles/21d9647eb05e9d)ではThe Merge（コンセンサスアルゴリズムの移行）について取り上げました。今回のブログでは、その間に位置するノード、Mempoolについて取り上げます。トランザクションが送られ、ブロックに取り込まれるまでの過程についてみていきます。

今までイーサリアムを使っていて、なぜトランザクションが成功するまでに時間を要するのか？と考えた方もいると思います。Mempoolを理解することで、その疑問も解決すると思います。

最初に、復習としてトランザクションがブロックに取り込まれるまでの全体の流れを理解します。次に、今回のテーマであるノード、Mempoolについてみていきます。

##　トランザクションがブロックに取り込まれるまで

トランザクションがブロックに取り込まれるまでには、以下のステップがあります。

1. externally owned accounts(EOAs)がデータに署名し、トランザクションを送る
2. トランザクションは、ノードによってネットワークに拡散され、他の多くのトランザクションとともにMempool(Tx Pool)で待機する
3. トランザクションは、バリデーターによってブロックに取り込まれる

１については、[前々回](https://zenn.dev/hokusai/articles/ac26ef75b89d7b)の記事で詳しく紹介しました。３についても[前回](https://zenn.dev/hokusai/articles/21d9647eb05e9d)の記事で、ブロックの生成がエネルギー消費によるマイナーの競争から、Etherをステーキングしているバリデーターの役割に変更になったことを書きました。詳しくは過去のブログを参考にしてください。今回のブログの対象は２になります。ここからは２について詳しく見ていきます。

##　ノードにおけるMempool(Tx Pool)

署名されたトランザクションがブロックに取り込まれるまでには、以下の過程を通過する必要があります。

### ノードへ

EOAsが署名したトランザクションはノードに送られます。ノードに送られたトランザクションはすぐにブロックに取り込まれません。ノードにあるMempoolに送られます。イーサリアムでは、このMempoolをtransaction poolまたはtxpoolと呼びます。以下、txpoolと書いていきます。

[^anchor]:　ノードとは、イーサリアムのブロックやトランザクションを検証するプログラムを使用しているコンピューター。イーサリアムは、このノードの集合により成り立っています。

### Mempoolへ

txpoolとは、ノードにあるトランザクションが待機する場所です。このtxpoolにあるトランザクションはまだブロックに含まれていません。他のトランザクションと共に、txpoolでバリデーターによってブロックに取り込まれるのを待ちます。

### Mempoolの役割

Mempoolはトランザクションにとって単なる待合室ではありません。Mempoolはヴァリデーションという大きな役割があります。

まずtxpoolに入る前に、トランザクションのハッシュが既に存在しているものか、またトランザクションの署名が正しいものであるかが検証されます。次にtxpoolにおいて、さらなる検証が実行されます。txpoolにおける検証には、

* トランザクションのサイズは適切なものであるか
* トランザクションのガスが現在のブロックのガスリミットを超えていないか
* トランザクションが正しいnonceを使用しているか
* トランザクションを送ったEOAsが十分なetherを保持しているか

などがあります。（他にもいくつかの検証項目があります。詳しくはgo-ethereumのtxpoolのコード[https://github.com/ethereum/go-ethereum/blob/master/core/txpool/txpool.go]にあるとvalidateTxという関数を確認して下さい。）

これらの検証を通過したトランザクションは、'Pending'なトランザクションとされます。

### ノードからノードへ

プールというと、たったひとつの場所にトランザクションが入っている印象を受けます。しかし、txpoolは各ノードに存在します。'Pending'とされるトランザクションは、一つのノードから、そのノードと接点のあるノードに拡散されます。ノードからノードへの伝播を繰り返すことによって、トランザクションはネットワークに広がり、各ノードのtxpoolにいれられます。

注意すべき点は、各ノードのtxpoolのサイズが異なることです。そのため、各ノードのtxpoolが保持するトランザクションの数は異なる可能性があります。

### ブロックへ

トランザクションは、バリデーターによってtxpoolからブロックへ取り込まれます。トランザクションがブロックに取り込まれると、トランザクションは'Successful'な状態であると位置づけられます。ブロックに含まれたトランザクションは、txpoolから消去されることになります。

以上がトランザクションがノードからノードへ拡散し、バリデーターによってブロックに取り込まれる過程になります。トランザクションはブロックに取り込まれるまでtxpoolにあることが確認できたと思います。

# 最後に

今回のブログでは、txpoolについて説明しました。ノードに送られたトランザクションは、txpoolによってブロックに含まれることを待ちます。その際に、ガス、nonce、アカウントのether保有量など、そのトランザクションを実行する上で必要とされる項目について検証が行われます。

dapps利用の際、トランザクションが完了するまで待ち時間がある際には、txpoolの存在を思い出してみてください。あなたのトランザクションは、txpoolでバリデーターに拾われるのを待っています！最後までご覧いただき、ありがとうございました。

# 参考資料

What is a Mempool?
https://www.alchemy.com/overviews/what-is-a-mempool

What happens when you send 1 DAI
https://www.notonlyowner.com/learn/what-happens-when-you-send-one-dai#submitting-the-transaction

TRANSACTIONS
https://ethereum.org/en/developers/docs/transactions/

How to Debug Pending Ethereum Transactions
https://alchemy.com/blog/how-to-debug-pending-ethereum-transactions

Mempool
https://academy.binance.com/en/glossary/mempool

Is your EVM node good enough?
https://chainstack.com/is-your-evm-node-good-enough/

Metamask transactions tutorial – a developer’s guide to transactions in Ethereum mempool
https://chainstack.com/a-developers-guide-to-the-transactions-in-mempool-metamask-edition/

