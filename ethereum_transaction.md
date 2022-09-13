# このブログの対象読者

①イーサリアムについて知りたい人<br />
②暗号資産やNFTを触れたことがあり、背景で何が起こっているのか理解したい人<br />
③スマートコントラクトを書いていて、EVM（Ethereum virtual machine）の知識を増やしたい人

# はじめに

[前回のブログ](https://zenn.dev/hokusai/articles/204a443fadcc21)では、イーサリアムのアカウントについて紹介しました。その中で、externally owned accounts(EOAs)のみがトランザクション作り、署名できることに触れました。今回のブログでは、トランザクションについて紹介します。このブログの最終目標は、MetaMaskなどのソフトウェアウォレットを使わずにトランザクションを実行することです。具体的にはEthers.jsを使用して、トランザクションを行います。前半部分でトランザクションについてまとめます。後半の部分では、プログラムを書いてトランザクションを実行していきます。エンジニアでない方も前半部分を読んで、トランザクションの知識を広げることができます。前回のブログでは、

# トランザクション

## トランザクションとは？

トランザクションは、イーサリアムのyellow paperの中で以下のように定義されています。

> A piece of data, signed by an External Actor.

日本語にすると、外部の存在によって署名されたデータくらいの意味でしょう。冒頭で書いたように、EOAsのみが署名をすることができます。an External Actorについては、EOAsに読み替えてもいいと思います。EOAsによって署名されたデータと考えられます。

## どのようなデータに署名がされるのか？

具体的にどのようなデータに署名がされるのでしょうか？次の項目が署名されるデータに含まれる内容です。

- to(recipient)
- nonce
- data
- value
- gasLimitなどのガスに関する情報

### to(recipient)

イーサリアムのアドレス。このデータが送信される宛先になります。to(recipient)がないトランザクションは、ブロックチェーンにコントラクトを作成するトランザクションになります。

### nonce

アカウントから送られたトランザクションの数を示します。

### data

トランザクションで送られるデータ。to(recipient)がコントラクトのアドレスである場合には、呼び出す関数名と引数がエンコードされたもの。

### value

このトランザクションでto(recipient)に送られるETH（単位はwei）。

### gasLimitなどのガスに関する情報

イーサリアムでトランザクションが実行されるにはガスが必要です。ガスに関する情報もトランザクションに含めることができます。

以上が署名されるデータの内容になります。

署名されるデータのなかで、特に重要な情報がdataとvalueです。この二つを組み合わせることで様々なトランザクションを実行することができます。代表的なのがコントラクトの関数を呼び出すことです。例えば、気に入ったＮＦＴをＥＴＨを支払って購入する場面を考えてみましょう。ＥＴＨで販売されているＮＦＴを購入するにはデータの内でvalueを設定してＥＴＨを送金します。同時に気に入ったＮＦＴのIDを指定するため、dataには引数としてIDが必要なことが予想されます。このトランザクションのto(recipient)は呼び出すコントラクトのアドレスになります。他にもdataは設定せず、valueのみ指定することで友人にETHを送るトランザクションなども考えられます。この際のto(recipient)は、友人のアドレスになります。ここからはコードを書いてトランザクションを実行していきます。


# トランザクションを実行してみよう

ここからはソフトウェアウォレットを使わずにトランザクションを実行していきます。テストネット上にデプロイされているコントラクトのstateをアップデートします。以下がPolygonのテストネットであるMumbaiにデプロイされているコントラクトのコードです。

```
pragma solidity 0.8.9;

contract MonobundleBlogCode {

    struct Account {
        string name;
        uint256 balance;
    }

    mapping(address => Account) public accounts;
    constructor() {}

    function setAccount(string calldata _newName) external payable {
        Account storage account = accounts[msg.sender];
        account.name = _newName;
        account.balance += msg.value;
    }
}
```

setAccountという関数は、コントラクトの内部にnameとbalanceを持つaccountsを作成します。このaccountsはsetAccountを呼び出したEOAsのアドレスに紐づけられています。balanceは関数を呼び出す際に送金されたETHの量、nameはパラメーター_newNameにより更新されます。Node.jsとethers jsの使い方については、[こちらの](https://docs.etherscan.io/tutorials/signing-raw-transactions)の記事１、２の項目を参考にしてください。

Ether.jsを使ってsetAccountを呼び出すコードを書いていきます。次の４つを順に実行していきます。

1. private keyとproviderからウォレットを作成
2. setAccountを実行するためのdataを作成する
3. 署名する対象をつくる
4. トランザクションを送る

## 1. private keyとproviderからウォレットを作成

Ethers.jsのnew Wallet ( privateKey [ , provider ] )を使って、ウォレットを作ります。コントラクトがデプロイされているネットワークは、mumbaiです。このネットワークにアクセスするために、mumbaiに接続されたウォレットを用意します。privateKeyと書かれた部分には、ご自身のprivateKeyを書き込んでください。

```
const provider = ethers.getDefaultProvider("https://rpc-mumbai.maticvigil.com/") 

const privateKey = "0xprivatekey"

const walletWithProvider = new ethers.Wallet(privateKey, provider);
```

## 2.　setAccountを実行するためのdataを作成する

呼び出す関数と引数を指定してきます。最初にnew ethers.utils.Interface( abi )でインターフェースを作成します。今回のAbi（Application Binary Interface ）はsolidityのコンパイラーが作成したものではなく、簡素化のため手書きのものを使用しています。次に、interface.encodeFunctionData( fragment [ , values ] )を使います。encodeFunctionDataを使用することによって、呼び出したい関数とその関数が必要とする引数をエンコードすることができます。encodeFunctionDataの最初の引数には、呼び出したい関数の名前を記述します。二つ目の引数である配列の中に、呼び出す関数が要求する引数を入れます。setAccountではstring calldata _newNameというパラメーターが必要なので、ここでは"Andi"というstringを入れました。ご自身の名前など好きな名前を入れてください。dataはこれで完成です。

```
const iface = new ethers.utils.Interface([
    "function setAccount(string calldata _newName) payable"    
])

const data = iface.encodeFunctionData("setAccount", [
    "Andi"
])
```

## 3. 署名する対象をつくる

署名がされるデータは以下のようになります。toには呼び出し先のコントラクトのアドレスが入ります。またsetAccountはpayableな関数なので、ETHをおくるためにvalueという項目も使用しています。最後のdataは、この直前に作成したsetAccountを実行するためのdataになります。

```
const tx = {
    to: "0xC00C7AdfB5f1927ba43CdC7e0A8d0f4c360319E5",
    value: ethers.utils.parseEther("1.0"),
    data
}
```

## 4. トランザクションを送る

署名をして、ネットワークにおくります。署名の対象はすべての項目を含んでいませんでした。例えば、nonce, gasLimit, gasPriceの情報がありません。sendTransactionはこれらの項目を自動で追加し、さらに署名もしてくれます。

```
await walletWithProvider.sendTransaction(tx)
```

トランザクションが実行されたか確認していきましょう。[mumbaiのpolygonscan](https://mumbai.polygonscan.com/)上で自分のアドレスを検索します。Transactionsの一番上にMETHODがsetAccountになっていて、Toが0xC00C7AdfB5f1927ba43CdC7e0A8d0f4c360319E5であるトランザクションを発見できたら、今回の目標は達成です。今回使用したコードをもう一度まとめておきます。

```
const { ethers } = require("ethers");

async function main() {
    const provider = ethers.getDefaultProvider("https://rpc-mumbai.maticvigil.com/") 

    const privateKey = "0x your privatekey"

    const walletWithProvider = new ethers.Wallet(privateKey, provider);

    const iface = new ethers.utils.Interface([
        "function setAccount(string calldata _newName) payable"    
    ])

    const data = iface.encodeFunctionData("setAccount", [
        "Andi",
    ])

    const tx = {
        to: "0xC00C7AdfB5f1927ba43CdC7e0A8d0f4c360319E5",
        value: ethers.utils.parseEther("1.0"),
        data
    }

    const result = await walletWithProvider.sendTransaction(tx)
    console.log(result)
}

main().catch((error) => {
    console.error(error);
    process.exitCode = 1;
});
```

## おまけ: なぜprivate keyが重要なのか？

private keyを失うと勝手にトランザクションを実行されてしまいます。上記のコードで、ウォレット作成に必要としたのはprivate keyのみです。private keyがあればウォレットを作成し、トランザクションに署名することが可能です。自分の資産(ETH,ERC20,NFT)を守るためにも、private keyはしっかり保管しましょう（絶対に他人に教えてはいけません）！

# 最後に

今回のブログではイーサリアムのトランザクションについて紹介しました。setAccountという関数はパラメーター_newNameを必要とし、またpayableであるため、トランザクションの中にvalueとdataが入っています。しかし、その組み合わせは自由です。例えば、EOAsにETHのみを送金したい場合、toに送金したい相手のアドレスを入れて、valueに送りたいETHの量を書くことで送金が可能です。payableでない関数を呼び出す際には、dataを設定して、valueを０に設定してトランザクションを実行することもできます。興味のある方は、テストネットで色々なトランザクションを実行してみてください。今回はトランザクションの中身を知ることで、Dapps利用の裏側に触れることができたと思います。最後まで読んでいただき、ありがとうございました。


# 参考資料

Ethereum Yellow Paper<br /> 
https://ethereum.github.io/yellowpaper/paper.pdf

Ethereum Whitepaper<br /> 
https://ethereum.org/en/whitepaper/

Andreas M. Antonopoulos. Mastering Ethereum

SENDING TOKENS USING ETHERS.JS
https://ethereum.org/da/developers/tutorials/send-token-etherjs/

How to Send Transactions on Ethereum
https://docs.alchemy.com/docs/how-to-send-transactions-on-ethereum

Signing Raw Transactions
https://docs.etherscan.io/tutorials/signing-raw-transactions

TRANSACTIONS
https://ethereum.org/en/developers/docs/transactions/

Transactions
https://docs.ethers.io/v5/api/utils/transactions/
