<!-- # はじめに

Upgradeableなコントラクト、スマートコントラクトを書き始めた人なら一度は聞いたことのある用語だと思います。Upgradeableなコントラクトについては、OpenZeppelinのLibraryも充実しています。そのため、すでに多くの人がUpgradeableなコントラクトを書いたことがあるかもしれません。

今回のブログでは、なぜコントラクトがUpgradeable可能になるかについて書いていきます。 delegatecall, proxy patternを理解している人には退屈な内容になると思います。


# Upgradeableなコントラクトの必要性

スマートコントラクトはアプリケーション開発において、バックエンドを担うとされます。しかし、本来のバックエンド開発と大きな違いがあります。それは、一度デプロイされたスマートコントラクトは変更ができない点です。このことはアプリケーション開発を難しくします。デプロイ後、変更のできないスマートコントラクトでは、バグの修正、新規機能追加などができません。その解決策とされるのが、Upgradeableなコントラクトです。


#  なぜUpgradeableなコントラクトが可能になるのか？

スマートコントラクトは、デプロイ後変更できない。それではなぜUpgradeableなコントラクトが可能になるのでしょうか？Upgradeableなコントラクトが可能になる背景には、DelegatecallとProxy patternがあります。

## Delegatecall

delegatecallはLow-level functionで、あるコントラクトから別のコントラクトを呼ぶ際に使われます。delegatecallには、二つの特徴があります。コントラクトAからコントラクトBを呼び出すというケースを想定します。一つ目の特徴は、コントラクトBのコードはロジックとして利用されます。そして、コントラクトAのstate variablesを更新します。二つ目の特徴は、msg.senderがコントラクトAを呼び出したアドレスになるということです。（delegatecallを使わずコントラクトAからコントラクトBを呼ぶ場合。この時、コントラクトBで実行されるコードは、コントラクトBのState variablesを参照し更新します。msg.senderはコントラクトAなります。）

文章だけでは理解するのが大変だと思います。具体例をみていきます。以下のコードは、[Solidity by Example](https://solidity-by-example.org/delegatecall)からの引用です。

```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

// NOTE: Deploy this contract first
contract B {
    // NOTE: storage layout must be the same as contract A
    uint public num;
    address public sender;
    uint public value;

    function setVars(uint _num) public payable {
        num = _num;
        sender = msg.sender;
        value = msg.value;
    }
}

contract A {
    uint public num;
    address public sender;
    uint public value;

    function setVars(address _contract, uint _num) public payable {
        // A's storage is set, B is not modified.
        (bool success, bytes memory data) = _contract.delegatecall(
            abi.encodeWithSignature("setVars(uint256)", _num)
        );
    }
}
```

コントラクトAのsetVarsという関数から、コントラクトBのsetVarsという関数を呼び出しています。コントラクトAのsetVarsがパラメーターとして、コントラクトBのアドレスを取ります。そして、_contract.delegatecall()という部分でdelegatecallが使われて、コントラクトBのsetVars関数が呼ばれています。コントラクトBのsetVars関数の中で、num、sender、valueというstate variablesが更新されます。実際に更新されるstate variablesは、コントラクトAのstate variablesになります。senderにはコントラクトAのsetVars関数を呼び出したアドレスが代入され、valueはコントラクトAが呼び出された際に、送金されたEtherになります。このことは、以下のテストコードでも確認することができます。興味がある方は、テストコードをhardhatで実行してみてください。

```
import { ethers } from 'hardhat'
import { expect } from 'chai'
import { Contract, constants } from 'ethers'
import { SignerWithAddress } from '@nomiclabs/hardhat-ethers/signers';

describe("Delegatecall", function () {
  let constractA: Contract
  let contractB: Contract
  let user1: SignerWithAddress

  before(async () => {
    [user1] = await ethers.getSigners()
    contractB = await (await ethers.getContractFactory("B")).deploy()
    constractA = await (await ethers.getContractFactory("A")).deploy()
  })

  it("prove of the delegatecall", async () => {
    expect(await constractA.num()).to.equal(0)
    expect(await constractA.sender()).to.equal(constants.AddressZero)
    expect(await constractA.value()).to.equal(0)
    
    const options = {value: ethers.utils.parseEther("1.0")}
    
    await constractA.connect(user1).setVars(contractB.address, 7, options)

    expect(await constractA.num()).to.equal(7)
    expect(await constractA.sender()).to.equal(user1.address)
    expect(await constractA.value()).to.equal(options.value)
  })
}); 
```

以上がdelegatecallの説明になります。まとめると、

①コントラクトBはロジックを実行するコードとなり、コントラクトA（呼び出し元）のstate variablesが更新されること
②コントラクトBのmsg.senderはコントラクトAではなく、コントラクトAを呼び出したアドレスになること

の二点が重要です。

## Proxy pattern

Proxy patternでは、ロジックとなるコントラクトのアドレスをstate variableとして保存します。さらに呼び出し元（コントラクトA）では、fallback functionを使用します。このfallback functionの中でdelegatecallが使われ、保存したアドレスをロジックコントラクトとして利用することを可能にします。

このロジックコントラクト自体は、他のコントラクトと同様、変更できないコントラクトです。コントラクトをUpgradeableにするのは、先ほど保存したロジックコントラクトのアドレスを更新します（例えば、コントラクトBアドレスから、コントラクトCのアドレスに変更する）これにより、delegatecallの対象はコントラクトCになります。delegatecallの特徴から、今までコントラクトBのロジックを利用して更新されてきたコントラクトAのstate variablesは維持されることになります。

# 最後に

今回のブログでは、なぜUpgradeableなコントラクトが可能になるのかについてまとめました。デプロイ後、変更できないはずのスマートコントラクトが、なぜUpgradeableになるのか理解できたと思います。次回のブログでは、Upgradeableなコントラクトを作成する際の注意点について書きたいと思います。最後まで読んでいただき、ありがとうございました。

# 参考資料

Solidity<br />
https://docs.soliditylang.org/en/v0.8.15/index.html

The State of Smart Contract Upgrades <br />
https://blog.openzeppelin.com/the-state-of-smart-contract-upgrades/#upgrade-patterns

Proxy Upgrade Pattern <br />
https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies -->




