# はじめに

スマートコントラクトの特徴の一つとして、Censorship resistanceがあげられます。ネットワークにデプロイされたコントラクトは、変更することができません。誰にも変更できないスマートコントラクトは、利点として考えられます。その一方、開発者にとっては、難しい特徴ともいえます。例えば、

①細心の注意を払って書いたコントラクトに生じたバグ
②新しい機能の追加

など、デプロイ後に変更できないスマートコントラクトは、開発を難しくすることもあります。この問題を解決するのが、Upgradeable Contractsです。

Proxy patternを利用することで、コントラクトを更新することを可能にしています。Proxy Patternについては、[この記事](https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies#the-constructor-caveat)を参考にしてください。

Upgradeableなコントラクトを作成する。少し難しいテーマのように聞こえます。Upgradeableなコントラクトを作る際、筆者は毎回Openzeppelinの[サイト](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#initializers)を参考にしています。今回のブログでは、このサイトをまとめます。そして、Upgradeableなコントラクト作成における、注意点チェックリストを作ってみたいと思います。さらに、このチェックリストをもとに、メインネットにデプロイされているUpgradeableなコントラクトを見ていきたいと思います。

メインネットデプロイの際には、ご自身で参考資料を読んで確認してください。また、テストネットで確認するなど、ご自身の責任でお願いします。

# Upgradeabilityチェックポイント

特に、constructors are replaced by initializer functions, state variables are initialized in initializer functions, and we additionally check for storage incompatibilities across minor versions.

## initializeを使う
constructorではなく、initializeを使う。

### construcotrについて
solidityではコントラクトを初期化する際、constructorを使います。constructorに書かれたコードは、デプロイ前に自動で実行されます。

### initializeとは
Proxy patternが可能にするUpgradeableなコントラクトでは、constructorによる初期化が実行されません。そのため、constructorではなく、普通の関数を利用して、コントラクトを初期化する必要があります。Upgradeableなコントラクトを読むと、initializeという関数をよく目にします。この関数がconstructorの役割を果たしています。ただし、名前自体はinitializeである必要はありません。初期化を実行する関数は、constructorと同じように一度のみ実行されるべきです。一度のみ実行されるよう、関数を実装する必要があります。さらに、普通の関数として書かれた初期化関数（initialize）は、自動で実行されません。デプロイ後、この関数を手動で実行する必要があります。


## Libraryを使う場合

Libraryも初期化のための関数を必要とする。

スマートコントラクトの開発では、Libraryを使う機会が多くあります。Upgradeableなコントラクトを作るときは、Libraryの使用にも注意が必要です。上記に書いたように、Upgradeableなコントラクトでは、constructorを使用することはできません。このことはLibraryにも当てはまります。Upgradeableなコントラクトを書く場合には、使用するLibraryにconstructorがないことを確認しましょう。

例えば、OpenZeppelinは、UpgradeableなコントラクトのためのLibraryを用意しています。Upgradeableなコントラクトのinitializeの中で

__Pausable_init();

などの記述がみられます。これは、Libraryにあるコントラクト初期化の関数を呼び出していることを示します。


## state variablesの初期化は、initialize内で行う

Solidityでは、state variablesを宣言する際に、初期値を代入することができます。この初期化は、constructor内での初期化と考えられます。(constant variablesに関しては、該当しません。Upgradeableなコントラクトでも、宣言と同時に初期化が可能です。)

```
/// 宣言と初期化
contract Hokusai {
    string public name = "Hokusai";
} 
```

これまで書いた二つの注意点と同様の理由から、state variablesの初期化もinitializeの中で実行する必要があります。


## チェックリスト

☐ Upgradeableなコントラクトでは、constructorがないこと、別の初期化の関数があることを確認する。<br /> 

☐ Libraryを使用する際には、初期化のための関数があるものを使用する。<br /> 

☐ constant variables以外のstate variablesは、initializeの中で初期化する。<br /> 

以上が、代表的な注意点です。他にも、Upgradeableなコントラクト内で新しいコントラクトを作る、delegatecall、selfdestruct使う際には、注意する必要があります。気になる方は、[もとの記事](https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable#initializers)を確認して下さい。

## 具体例

ここからは、上記のチェックリストを使って、実際にデプロイされているコントラクトのコードを見ていきます。今回見ていくのは、Nouns DAOの[NounsAuctionHouse.sol](https://github.com/nounsDAO/nouns-monorepo/blob/master/packages/nouns-contracts/contracts/NounsAuctionHouse.sol)です。それでは見ていきます。

### ☐ Upgradeableなコントラクトでは、constructorがないこと、別の初期化の関数があることを確認する。





Writing Upgradeable Contracts <br />
https://docs.openzeppelin.com/upgrades-plugins/1.x/writing-upgradeable

The State of Smart Contract Upgrades <br />
https://blog.openzeppelin.com/the-state-of-smart-contract-upgrades/#upgrade-patterns

Proxy Upgrade Pattern <br />
https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies#the-constructor-caveat

Using with Upgrades <br />
https://docs.openzeppelin.com/contracts/4.x/upgradeable

<!-- ## upgradeability check lists

- [ ]  use initialize instead of constrcutor
- [ ]  use Initializable to prevent a contract from being *initialized*
 multiple times
- [ ]  use upgradeable smart cntract libraries
    - import statement
    - inherit
    - in initialize
- [ ]  avoid initializing values in field declarations(except constants variables)
- [ ]  initialize the implementation contract with _disableInitializers()
- [ ]  in case of creating new instances from the contract code -->

