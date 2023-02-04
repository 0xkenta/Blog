# Duration
start: 15:30<br />
pause: 17:30<br />
restart: 20:30<br />
end: 22:30<br />

# NFTマーケットプレイスにおけるvampire attackへの対処法について

最初にvampire attackがどのようなものであるかをみていく。次に、実際に起こったNFTマーケットプレイスでのvampire attackを紹介したい。そして、マーケットプレイスにおけるvampire attackへの対処法を提示していく。

# vampire attackとは？

vampire attackとは、Defi、NFTなどのサービス提供者が競合するサービスにしかけるマーケティングの一種。vampire attackを仕掛けるサービス提供者は、ユーザーに魅力的な報酬,条件を提供することでユーザー、流動性そして取引量の獲得を狙う。有名なものとしてSushiswapがuniswapに対しておこなったvampire attackがある。

# OpenSeaに対するvampire attackでの事例

次に、NFTマーケットプレイスのOpenSeaに対して行われた２つのvampire attackについてみていく。一つ目がX2Y2、二つ目がLooksRareの例になる。両者とも過去にOpenSeaを利用したユーザーに対してトークンをAirdropするというvampire attackをおこなった。

## X2Y2

X2Y2では、発行量の１２パーセントにあたるX2Y2を配布するvampire attackを行った。具体的には、イーサリアム上で最初に生成されたブロックから、２０２２年の一つ目までのブロックまでのOpenSeaの利用者を対象とした。一般のユーザーは、この期間にOpenSeaでNFTの売買をしたアドレスを所持していること。さらにX2Y2条にNFTをリストすることが条件とされた。[トークンの配布量は、リストするNFTの数、NFTのコレクションの数のよって決定された。](https://docs.x2y2.io/tokens/rewards/opensea-airdrop)

## LooksRare

LooksRareにおいても、X2Y2と同様に総供給量の１２％のLOOKSがAirdropとしてvampire attackに使われた。LooksRareのAirdrop対象者の条件は以下の二つである。

1. 一定期間内（イーサリアムのBlock 12642194から13812868まで）にOpenSea上で合計3Ether以上取引したユーザー。
2. LooksRare上でERC721またはERC1155のトークンをリストしたもの。

以上がOpenSeaに対して行われたvampire attackの内容である。

# X2Y2とLooksRareのvampire attack後のNFTマーケットプレイスの推移

ここからは、LooksRareの仕掛けたvampire attackの結果を確認していきたい。LooksRareのAirdropの最終日は、２０２２年の３月２２日。まず最初に２０２２年のNFTの取引高からみるOpenSeaとLooksRareのマーケットシェア推移をみていく。

![image share per volume](https://github.com/0xkenta/Blog/blob/main//image/share_per_volume1.png)

取引高によるマーケットシェアの推移をみると、LooksRareは一時期シェアを落とす時期がもあるが、一定のシェアを維持していることが分かる。

次に取引量におけるマーケットシェアに関するチャートを見ていく。

![image share per transaction amount](https://github.com/0xkenta/Blog/blob/main//image/market_share_per_transaction_count.png)

取引量におけるマーケットシェアからは、vampire attackによるシェアの獲得をみることはできない。

以上のことから、LooksRareではNFT一つあたりの取引価格が高いことが予想される。実際に以下のチャートが示す通り、Average Sale Size (ETH)はLooksRareがOpenSeaを上回っていることが分かる。

![image share per transaction amount](https://github.com/0xkenta/Blog/blob/main//image/nft_average_sale_size.png)


# 僕が考えるマーケットプレイスへの対処法

LooksRareのNFT一つあたりの取引価格が高いことから、OpenSeaにおいて価格の高いNFTに対してインセンティブ設計を行うことを提案したい。一つ目は、高価格帯のNFTの手数料を低くすることである。単価の高いNFTの取引手数料をLooksRare以下にすることで、高価格帯の取引を活性化する。二つ目として、複数の高価格帯のNFTを取引するユーザーにキャッシュバックを行うことである。この二つの政策により、取引高におけるマーケットシェアが改善され、取引高、取引量におけるマーケットシェアでトップに立つことを目指す。

# 結論

vampire attackにおける対策は、前もって行えるものではないのではないだろうか。適切な防衛策は、vampire attack後に、どの領域において競合にユーザーを取り込まれたかを考えることだ。LooksRareでは価格の高いNFTが取引されていることが確認された。このことから、価格の高いNFTを取引するユーザーへのインセンティブが重要になるのではないだろうか。今回はLooksRareとOpenSeaの例から、vampire attackへの対処法を考えた。しかし、OpenSeaとX2Y2を比較した場合には、他の結果が出ていたことも考えられる。これからも個々の事例を考えていきたいと思う。

# 参考資料
- Vampire Attack Explained: An Incentive-Based Marketing Tactic That Does the Trick
https://www.blockchainmarketing.space/vampire-attack-explained/

- What is a Vampire Attack? How do Vampire Attacks work in Crypto?
https://coin98.net/what-is-a-vampire-attack  

- Airdrop to OpenSea Users
https://docs.x2y2.io/tokens/rewards/opensea-airdrop

- What Is The LOOKS Airdrop?
https://docs.looksrare.org/guides/faqs/what-is-the-looks-airdrop

https://twitter.com/looksrare/status/1502612671749058561?lang=en

https://dune.com/999777/hildobby-nft-market-overview

https://dune.com/queries/1622353/2688987
