
# このブログの対象読者

①イーサリアムについて知りたい方<br />
②暗号資産やNFTを触れたことがあり、背景で何が起こっているのか理解したい人<br />
③スマートコントラクトを書いていて、EVM（Ethereum virtual machine）の知識を増やしたい方

# はじめに

突然ですが、問題です。Etherscanには、二つの異なるアカウントの種類があります。両者の違いはどこにあるでしょう？

ヒントです。イーサリアムのアカウントは、Etherscan上で見ることができます。ご自身のウォレットアドレスと[0x9C8fF314C9Bc7F6e59A9d9225Fb22946427eDC03](https://etherscan.io/address/0x9C8fF314C9Bc7F6e59A9d9225Fb22946427eDC03)を入力して、表示の違いを確認してみて下さい。

今回のブログでは、イーサリアムのアカウントについて紹介します。問題に答えることができた方も、まだ知らない発見があるかもしれません。最後まで読んでみてください。

# イーサリアムのアカウント

冒頭で触れたように、イーサリアムには二種類のアカウントがあります。一つ目がexternally owned accounts (EOAs)。二つ目がcontract accountsです。EOAsはprivate keyによって管理されるアカウントです。私たちユーザーが保持しているアカウントともいえます。contract accountsは、スマートコントラクトがデプロイされて作られるアカウントになります。contract accountsはprivate keyを保持していません。イーサリアムのworld stateはこの両方のアカウントの集合体になります。

ここからはアカウントの特徴をみていきます。

## アドレスとState

### アドレス

どちらのアカウントもアドレスを持っています。このアドレスは、十六進法で書かれた４２文字から成っています。先ほどあげた0x9C8fF314C9Bc7F6e59A9d9225Fb22946427eDC03もイーサリアムのアドレスです。４２文字あるか数えてみてください。

## State

それぞれのアドレスにはアカウントのStateが紐づけられています。アカウントのstateは以下の４項目から成ります。

### ①nonce

EOAsでは、アカウントから送られたトランザクションの数を示します。一番最初のトランザクションのnonceは０を示し、それ以降トランザクションを送るたび１増加していきます。contract accountsでは、アカウントから作られたコントラクトの数を示します。コントラクトが新しいコントラクトデプロイするとnonceが１増えます。(contract accountsのnonceは０ではなく１から始まります。)

### ②ether balance

アドレスが持つweiの量を表します。例えば、あるアカウントが1 ETH保持しているとします。その時のアカウントのバランスは、1,000,000,000,000,000,000 wei（０が１８個です）になります。どちらのアカウントもETHを受け取る、また送ることが可能です。その際に、このbalanceの値が更新されていきます。solidityでは、<address>.balanceで取得することが可能です。

### ③codeHash

EVM上でアカウントが持つプログラムコードのハッシュ。ユーザーがDappsを利用するときに実行されるスマートコントラクトのコードになります。スマートコントラクトは、デプロイした後に変更ができないと言われます。その理由は、このcodeHashを書き換えることができないためです。EOAsはコードを持たないため、empty stringとなります。

### ④storageRoot

[Merkle Patricia tree](https://ethereum.org/en/developers/docs/data-structures-and-encoding/patricia-merkle-trie/)のルーツノードのハッシュ。contract accountsのstorageは、Merkle Patricia treeで管理されています。そのルーツノードのハッシュになります。

EOAsとcontract accountsでは、それぞれの保持するstate(codeHashとstorageRoot)に違いがあることが確認できました。これが最初に出した問題の答えです。Etherscanで確認すると、contract accountsではContractCreatorという項目で、誰がこのコントラクトをデプロイしたかを確認することができます。またcontractという項目をみると、ソースコードを見ることができます（コードがverifyされていない場合は、見ることができません。）

しかし、両者の違いはstateだけではありません。最後にEOAsとcontract accountsの機能面での違いを一つ挙げます。

## 重要な相違点

イーサリアムのworld stateはトランザクションによって更新されます。このトランザクションはEOAsによって作られます。具体的には、
ユーザーはトランザクションを作成します。そして、作成したトランザクションにprivate keyで署名します。その後、トランザクションはブロックに取り込まれます。これによりworld stateが更新されます。private keyが使われることからも明らかですが、トランザクションを作成できるのはEOAsのみになります。このこともEOAsとcontract accountsの違いになります。

### おまけ: なぜprivate keyが重要なのか？

重要な相違点でみたように、private keyがあればトランザクションに署名することができます。private keyを誰にも教えてはいけない理由は、このためです。悪意のある他人が、あなたのprivate keyを手に入れたとします。この人物はトランザクションを作成し、あなたのprivate keyで署名します。彼はこのトランザクションで、あなたのETHやERC20トークンを自分の指定するアドレスに送ることが可能です。悪意のある人物は、あの手この手でprivate keyを聞き出そうとします。private keyの扱いには注意しましょう。

# 最後に

今回のブログではイーサリアムのアカウントについてみてきました。簡単なテーマのようですが、EVMやいくつかの重要なのEthereum Improvement Proposals (EIPs) を理解するときにも必要な知識です。また必要になったときに読み返してみてください。最後まで読んでいただきありがとうございました。


# 参考資料
Ethereum Yellow Paper<br /> 
https://ethereum.github.io/yellowpaper/paper.pdf

Ethereum Whitepaper<br /> 
https://ethereum.org/en/whitepaper/

Andreas M. Antonopoulos. Mastering Ethereum

Introduction to Smart Contracts<br /> 
https://docs.soliditylang.org/en/v0.8.15/introduction-to-smart-contracts.html?highlight=nonce#accounts

ETHEREUM ACCOUNTS<br /> 
https://ethereum.org/en/developers/docs/accounts/

State in Ethereum<br /> 
https://docs.polygon.technology/docs/edge/concepts/ethereum-state/#world-state

PATRICIA MERKLE TREES<br /> 
https://ethereum.org/en/developers/docs/data-structures-and-encoding/patricia-merkle-trie/

What is an Ethereum Address?<br /> 
https://info.etherscan.com/what-is-an-ethereum-address/#:~:text=An%20Ethereum%20address%20is%20a,receive%20funds%20from%20another%20party.

<!-- https://ethereum.stackexchange.com/questions/31809/why-do-contracts-start-with-nonce-1/32347#32347 -->



