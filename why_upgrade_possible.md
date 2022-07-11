# はじめに

Upgradeableなコントラクト、スマートコントラクトを書き始めた人なら一度は聞いたことのある用語だと思います。Upgradeableなコントラクトについては、OpenZeppelinのLibraryも充実しています。そのため、すでに多くの人がUpgradeableなコントラクトを書いたことがあるかもしれません。

今回のブログでは、なぜコントラクトがUpgradeable可能になるかについて書いていきます。 DELEGATECALL、Proxy patternを理解している人には退屈な内容だと思います。それでは見ていきます。

# なぜUpgradeableなコントラクトが必要とされるのか？

スマートコントラクトはアプリケーション開発において、バックエンドを担うとされます。しかし、本来のバックエンド開発と大きな違いがあります。それは、一度デプロイされたスマートコントラクトは変更ができない点です。このことはアプリケーション開発を難しくします。デプロイ後、変更のできないスマートコントラクトでは、バグの修正、新規機能追加などができません。その解決策とされるのが、Upgradeableなコントラクトです。

#  なぜUpgradeableなコントラクトが可能になるのか？

スマートコントラクトは、デプロイ後変更できない。それではなぜUpgradeableなコントラクトは可能なのでしょうか？Upgradeableなコントラクトが可能になる背景には、DelegatecallとProxy patternがあります。

## Delegatecall



## Proxy pattern

