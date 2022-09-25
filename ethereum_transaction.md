# このブログの対象読者

1. イーサリアム、Ethereum virtual machine(EVM)について知りたい方<br />
2. 暗号資産やNFTを触れたことがあり、背景で何が起こっているのか理解したい方<br />
3. スマートコントラクトを書いていて、EVMの知識を増やしたい方

# はじめに

[前回のブログ](https://zenn.dev/hokusai/articles/204a443fadcc21)では、イーサリアムのアカウントについて紹介しました。その中で、externally owned accounts(EOAs)のみがトランザクション作り、署名できることに触れました。

今回のブログでは、トランザクションについて紹介します。このブログの最終目標は、MetaMaskなどのソフトウェアウォレットを使わずにトランザクションを実行することです。具体的にはEthers.jsを使用して、トランザクションを行います。前半部分でトランザクションについてまとめます。後半の部分では、プログラムを書いてトランザクションを実行していきます。エンジニアでない方も前半部分を読んで、トランザクションの知識を広げることができます。

# トランザクション

## トランザクションとは？

トランザクションは、イーサリアムのyellow paperの中で以下のように定義されています。

> A piece of data, signed by an External Actor.

日本語にすると、『外部の存在によって署名されたデータ』くらいの意味でしょう。冒頭で書いたように、EOAsのみが署名をすることができます。an External Actorについては、EOAsに読み替えてもいいと思います。EOAsによって署名されたデータと考えられます。

## どのようなデータに署名がされるのか？

具体的にどのようなデータに署名がされるのでしょうか？次の項目が署名されるデータに含まれる内容です。

- to(recipient)
- nonce
- data
- value
- gasLimitなどのガスに関する情報

### to(recipient)

イーサリアムのアドレス。このデータが送信される宛先になります。to(recipient)がないトランザクションは、ブロックチェーン上にコントラクトを作成するトランザクションになります。

### nonce

アカウントから送られたトランザクションの数を示します。

### data

トランザクションで送られるデータ。to(recipient)がコントラクトのアドレスである場合には、呼び出す関数名と引数がエンコードされたもの。

### value

このトランザクションでto(recipient)に送られるETH（単位はwei）。

### gasLimitなどのガスに関する情報

イーサリアムでトランザクションが実行されるにはガスが必要です。ガスに関する情報もトランザクションに含めることができます。


以上が署名されるデータの内容になります。署名されるデータのなかで、特に重要な情報がdataとvalueです。この二つを組み合わせることで様々なトランザクションを実行することができます。

代表的なのがコントラクトの関数を呼び出すことです。例えば、マーケットプレイスでのNFTの購入を考えてみます。マーケットプレイスのコントラクトの中に、
気に入ったＮＦＴをＥＴＨを支払って購入できる関数があるとします。ＥＴＨで販売されているＮＦＴを購入するには、データの内部でvalueを設定してＥＴＨを送金します。同時に気に入ったＮＦＴのIDを指定するため、dataには引数としてIDを入れます。このトランザクションのto(recipient)は呼び出すコントラクトのアドレスになります。

他にもdataは設定せず、valueのみ指定することで友人にETHを送るトランザクションなども考えられます。この際のto(recipient)は、友人のアドレスになります。

ここからはコードを書いてトランザクションを実行していきます。


# トランザクションを実行してみよう

ここからはソフトウェアウォレットを使わずにトランザクションを実行していきます。テストネット上にデプロイされているコントラクトのstateをアップデートします。以下がイーサリアムのテストネットであるGoerliにデプロイされているコントラクトのコードです。MonobundleDepositは、簡易的な銀行口座を模したコントラクトです。

```
pragma solidity 0.8.9;

contract MonobundleDeposit {

    struct Account {
        string name;
        uint256 balance;
    }

    mapping(address => Account) public accounts;

    event AccoundCreated(address userAddress, string name, uint256 balance);
    
    constructor() {}

    function openAccount(string calldata _newName) external payable {
        Account storage account = accounts[msg.sender];
        require(bytes(account.name).length == 0, "ACCOUNT EXISTS");

        account.name = _newName;
        account.balance = msg.value;

        emit AccoundCreated(msg.sender, _newName, msg.value);
    }
}
```

openAccountという関数は、コントラクトの内部にnameとbalanceを持つaccountsを作成します。このaccountsはopenAccountを呼び出したEOAsのアドレスに紐づけられています。balanceは関数を呼び出す際に送金されたetherの量、nameは引数_newNameにより更新されます。Node.jsとethers jsの使い方については、[こちら](https://docs.etherscan.io/tutorials/signing-raw-transactions)の記事１、２の項目を参考にしてください。

Ether.jsを使ってopenAccountを呼び出すコードを書いていきます。次の４つを順に実行していきます。(トランザクションを実行する際に、ETHを送金します。ETHは送金され、アカウントのETHは減少します。大量のETHは送らないようにしましょう。)

1. private keyとproviderからウォレットを作成
2. openAccountを実行するためのdataを作成する
3. 署名するデータをつくる
4. トランザクションを送る

## 1. private keyとproviderからウォレットを定義する

Ethers.jsのnew Wallet ( privateKey [ , provider ] )を使って、ウォレットを定義します。コントラクトがデプロイされているネットワークは、Goerliです。このネットワークにアクセスするために、Goerliに接続されたウォレットを用意します。privateKeyと書かれた部分には、ご自身のprivateKeyを書き込んでください。

```
const provider = ethers.getDefaultProvider("goerli");

const privateKey = "0x your private key"

const walletWithProvider = new ethers.Wallet(privateKey, provider);
```

## おまけ: なぜprivate keyが重要なのか？

private keyを失うと勝手にトランザクションを実行されてしまいます。上記のコードで、ウォレットの定義に必要としたのはprivate keyのみです。private keyがあればウォレットを定義し、トランザクションに署名することが可能です。自分の資産(ETH,ERC20,NFT)を守るためにも、private keyはしっかり保管しましょう（絶対に他人に教えてはいけません）！

開発の際に注意したい点は、間違ってprivate keyをいれたファイルをGitHub上で公開してしまうことです。private keyを入れたファイルをcommitしないように注意しましょう。また、現実の資産に紐付いていない開発専用のアカウントを使用することも、自身の暗号資産を守ることにつながります。

## 2.openAccountを実行するためのdataを作成する

呼び出す関数と引数を指定してきます。

最初にnew ethers.utils.Interface( abi )でインターフェースを作成します。今回のAbi（Application Binary Interface ）はsolidityのコンパイラーが作成したものではなく、簡素化のため手書きのものを使用しています。

次に、interface.encodeFunctionData( fragment [ , values ] )を使います。encodeFunctionDataを使用することによって、呼び出したい関数とその関数が必要とする引数をエンコードすることができます。encodeFunctionDataの最初の引数には、呼び出したい関数の名前を記述します。二つ目の引数である配列の中に、呼び出す関数が要求する引数を入れます。openAccountではstring calldata _newNameというパラメーターが必要なので、ここでは"Andi"というstringを入れました。ご自身の名前など好きな名前を入れてください。dataはこれで完成です。

```
const iface = new ethers.utils.Interface([
    "function openAccount(string calldata _newName) payable"    
])

const data = iface.encodeFunctionData("openAccount", [
    "Andi",
])
```

## 3. 署名するデータをつくる

署名がされるデータは以下のようになります。toには呼び出し先のコントラクトのアドレスが入ります。またopenAccountは、etherを受け取ることができるpayableな関数なので、etherをおくるためにvalueという項目も使用しています。最後のdataは、この直前に作成したsetAccountを実行するためのdataになります。

```
const unsignedTx = {
    to: "0xE1cC7A816521cD5C67a5b555151e82721666c275",
    value: ethers.utils.parseEther("0.0005"),
    data
}
```

## 4. トランザクションを送る

sendTransaction()という関数を使って、署名、トランザクションの送信を行います。ここで勘の良い方はお気づきかもしれませんが、「3. 署名するデータをつくる」ではnonce, gasLimitなどの情報を含んでいませんでした。これは、 sendTransaction()関数が自動でそれらの項目を設定してくれるためです。（手動で指定することもできます。）

```
const tx = await walletWithProvider.sendTransaction(unsignedTx)
```

コードを実行して、トランザクションが成功したか確認していきましょう。[GoerliのEtherscan](https://goerli.etherscan.io/)上で自分のアドレスを検索します。Transactionsの一番上にToが0xE1cC7A816521cD5C67a5b555151e82721666c275であるトランザクションを発見できたら、今回の目標は達成です。[0xE1cC7A816521cD5C67a5b555151e82721666c275](https://goerli.etherscan.io/address/0xE1cC7A816521cD5C67a5b555151e82721666c275#readContract)のRead contractからも、accountsが送金したetherの量と名前で更新されていることが確認できると思います。今回使用したコードをもう一度まとめておきます。

```
const { ethers } = require("ethers");

async function main() {
    const provider = ethers.getDefaultProvider("goerli");

    const privateKey = "0x your private key"

    const walletWithProvider = new ethers.Wallet(privateKey, provider);

    const iface = new ethers.utils.Interface([
        "function openAccount(string calldata _newName) payable"    
    ])

    const data = iface.encodeFunctionData("openAccount", [
        "Andi",
    ])

    const unsignedTx = {
        to: "0xE1cC7A816521cD5C67a5b555151e82721666c275",
        value: ethers.utils.parseEther("0.0005"),
        data
    }

    const tx = await walletWithProvider.sendTransaction(unsignedTx)
    console.log(tx)
}

main().catch((error) => {
    console.error(error);
    process.exitCode = 1;
});
```

# 最後に

今回のブログではイーサリアムのトランザクションについて紹介しました。ソフトウェアウォレットを使わずトランザクションを実行することで、Dapps利用の裏側に触れることができたと思います。興味のある方は、テストネットで色々なトランザクションを実行してみてください。最後までご覧いただきありがとうございました。

# 参考資料

Ethereum Yellow Paper<br /> 
https://ethereum.github.io/yellowpaper/paper.pdf

Ethereum Whitepaper<br /> 
https://ethereum.org/en/whitepaper/

Andreas M. Antonopoulos. Mastering Ethereum

SENDING TOKENS USING ETHERS.JS<br />
https://ethereum.org/da/developers/tutorials/send-token-etherjs/

How to Send Transactions on Ethereum<br />
https://docs.alchemy.com/docs/how-to-send-transactions-on-ethereum

Signing Raw Transactions<br />
https://docs.etherscan.io/tutorials/signing-raw-transactions

TRANSACTIONS<br />
https://ethereum.org/en/developers/docs/transactions/

Transactions<br />
https://docs.ethers.io/v5/api/utils/transactions/
