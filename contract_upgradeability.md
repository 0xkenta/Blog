# はじめに

スマートコントラクトの特徴の一つとして、Censorship resistanceがあげられます。ネットワークにデプロイされたコントラクトは、変更することができません。誰にも変更できないスマートコントラクトは、利点として考えられます。その一方、開発者にとっては難しい特徴ともいえます。例えば、

①デプロイしたコントラクトに生じたバグ
②新しい機能の追加

など、デプロイ後に変更できないスマートコントラクトは、開発を難しくすることもあります。この問題を解決するのが、Upgradeable Contractsです。

Proxy patternを利用することで、コントラクトを更新することを可能にしています。Proxy Patternについて詳しく知りたい方は、[この記事](https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies#the-constructor-caveat)を参考にしてください。

Upgradeableなコントラクトを作成する。少し難しいテーマのように聞こえます。Upgradeableなコントラクトを作る際、筆者は毎回OpenZeppelinのサイトを参考にしています。今回のブログでは、これらのサイトをまとめます。そして、Upgradeableなコントラクト作成における、注意点チェックリストを作ってみたいと思います。さらに、このチェックリストをもとに、メインネットにデプロイされているUpgradeableなコントラクトを見ていきます。

メインネットデプロイの際には、ご自身で参考資料を読んで確認してください。また、テストネットで確認するなど、ご自身の責任でお願いします。


# Upgradeabilityチェックポイント

## initializeを使う
constructorではなく、initializeを使う。

### construcotrについて
solidityではコントラクトを初期化する際、constructorを使います。constructorに書かれたコードは、デプロイ前に自動で実行されます。

### initializeとは
Proxy patternが可能にするUpgradeableなコントラクトでは、constructorによる初期化が実行されません。そのため、constructorではなく、普通の関数を利用して、コントラクトを初期化する必要があります。Upgradeableなコントラクトを読むと、initializeという関数をよく目にします。この関数がconstructorの役割を果たしています。ただし、名前自体はinitializeである必要はありません。初期化を実行する関数は、constructorと同じように一度のみ実行されるべきです。一度のみ実行されるよう、関数を実装する必要があります。普通の関数として書かれた初期化関数（initialize）は、自動で実行されません。デプロイ後、この関数を手動で実行する必要があります。


## Libraryを使う場合

Libraryも初期化のための関数を必要とする。

スマートコントラクトの開発では、Libraryを使う機会が多くあります。Upgradeableなコントラクトを作るときは、Libraryの使用にも注意が必要です。先ほど書いたように、Upgradeableなコントラクトでは、constructorを使用することはできません。このことはLibraryにも当てはまります。Upgradeableなコントラクトを書く場合は、使用するLibraryにconstructorがないことを確認しましょう。

例えば、OpenZeppelinは、UpgradeableなコントラクトのためのLibraryを用意しています。Upgradeableなコントラクトのinitializeの中で

__Pausable_init();

などの記述がみられます。これは、Libraryにあるコントラクト初期化の関数を呼び出しています。


## state variablesの初期化は、initialize内で行う

Solidityでは、state variablesを宣言する際に、初期値を代入することができます。この初期化は、constructor内での初期化と同様と考えられます(constant variablesに関しては、該当しません。Upgradeableなコントラクトでも、constant variablesは宣言と同時に初期化が可能です)

```
/// 宣言と初期化
contract Hokusai {
    string public name = "Hokusai";
} 
```

これまで書いた二つの注意点と同様の理由から、state variablesの初期化もinitializeの中で実行する必要があります。

```
/// Upgradeableなコントラクトにおける初期化
contract Hokusai {
    string public name;

    function initialize() external initializer {
        name = "Hokusai";
    }
}
```


## チェックリスト

☐ Upgradeableなコントラクトでは、constructorがないこと、別の初期化の関数があることを確認する。<br /> 

☐ Libraryを使用する際には、初期化のための関数があるものを使用する。<br /> 

☐ constant variables以外のstate variablesは、initializeの中で初期化する。<br /> 

以上が、代表的な注意点です。他にも、Upgradeableなコントラクト内で、新しいコントラクトを作る、delegatecall、selfdestruct使う際などには、注意が必要です。気になる方は、[もとの記事](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#initializers)を確認して下さい。


# 具体例

ここからは、上記のチェックリストを使って、実際にデプロイされているUpgradeableなコントラクトのコードを見ていきます。今回見ていくのは、Nouns DAOの[NounsAuctionHouse.sol](https://github.com/nounsDAO/nouns-monorepo/blob/master/packages/nouns-contracts/contracts/NounsAuctionHouse.sol)です。それでは見ていきます。

重要になるのは、NounsAuctionHouse.solにある、以下の関数です。

```
 function initialize(
    INounsToken _nouns,
    address _weth,
    uint256 _timeBuffer,
    uint256 _reservePrice,
    uint8 _minBidIncrementPercentage,
    uint256 _duration
) external initializer {
    __Pausable_init();
    __ReentrancyGuard_init();
    __Ownable_init();

    _pause();

    nouns = _nouns;
    weth = _weth;
    timeBuffer = _timeBuffer;
    reservePrice = _reservePrice;
    minBidIncrementPercentage = _minBidIncrementPercentage;
    duration = _duration;
}
```

### ☐ Upgradeableなコントラクトでは、constructorがないこと、別の初期化の関数があることを確認する。

まず、NounsAuctionHouse.solにconstructorは存在しません。代わりに、initializeが使われています。チェックリストの１つ目は満たされていることが分かります。また、initializeではinitializerというOpenZeppelinのLibraryが使われています。これにより、この関数は初期化の際、一度のみ使用されることになります。

### ☐ Libraryを使用する際には、初期化のための関数があるものを使用する。

NounsAuctionHouse.solは以下のように、いくつかのLibraryを継承しています。

```
contract NounsAuctionHouse is INounsAuctionHouse, PausableUpgradeable, ReentrancyGuardUpgradeable, OwnableUpgradeable {}
```

これらのLibraryは、OpenZeppelinが提供するUpgradeableなコントラクトのためのLibraryです。initializeの中では、

```
__Pausable_init();
__ReentrancyGuard_init(); 
__Ownable_init();
```

を使い、これらのLibraryを初期化しています。チェックリストの二つ目もクリアしています。

### ☐ constant variables以外のstate variablesは、initializeの中で初期化する。

コントラクトの中で、以下のstate variablesが宣言されています。

```
// The Nouns ERC721 token contract
INounsToken public nouns;

// The address of the WETH contract
address public weth;

// The minimum amount of time left in an auction after a new bid is created
uint256 public timeBuffer;

// The minimum price accepted in an auction
uint256 public reservePrice;

// The minimum percentage difference between the last bid amount and the current bid
uint8 public minBidIncrementPercentage;

// The duration of a single auction
uint256 public duration;
```

これらのstate variablesは宣言されていますが、初期値が代入されていません。代入はinitialize内部で、次のように行われています。

```
nouns = _nouns;
weth = _weth;
timeBuffer = _timeBuffer;
reservePrice = _reservePrice;
minBidIncrementPercentage = _minBidIncrementPercentage;
duration = _duration;
```

三つ目の注意点もクリアしていることが確認できました。

NounsAuctionHouse.solは、Zoraの[AuctionHouse.sol](https://github.com/ourzora/auction-house/blob/54a12ec1a6cf562e49f0a4917990474b11350a2d/contracts/AuctionHouse.sol)参考に作られています。Zoraのコントラクトは、Upgradeableなコントラクトとして作成されていません。興味がある方は、NounsとZoraのコントラクトを比べると、さらに理解が深まると思います。


# 最後に

今回のブログでは、Upgradeableなコントラクトを作成する際の注意点について紹介しました。Upgradeableなコントラクトは、開発において大きなメリットをもたらします。ぜひ試してみてください。メインネットにデプロイを予定される方は、ご自身で以下に記載する参考資料などを読んでコントラクトを作成してください。


# 参考資料

Writing Upgradeable Contracts <br />
https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable

The State of Smart Contract Upgrades <br />
https://blog.openzeppelin.com/the-state-of-smart-contract-upgrades/#upgrade-patterns

Proxy Upgrade Pattern <br />
https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies#the-constructor-caveat

Using with Upgrades <br />
https://docs.openzeppelin.com/contracts/4.x/upgradeable


