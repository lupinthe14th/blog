---
title: "木"
date: 2020-07-01T09:08:05+09:00
tags: ["memo", "trie", "trie node", "prefix tree", "graph", "algorithm"]
math: true
draft: false
---

木はノードで構成されるデータ構造です。
また、木はグラフの一種です。
簡単にいうと、木は閉路なしの連結グラフです。

## 木の種類

- 木は根（root）ノードを持っています。
- 根ノードは0個以上の子ノードを持っています。
- 各子ノードは0個以上の子ノードを持っていて、それらの子ノード以下も同様に子ノードを持っています。

シンプルなノード構造体は次のようになります。

```golang
type Node struct {
	name     string
	childlen []Node
}
```

### 木と二分木（Tree and Binary tree）

二分木は各ノードが最大2個の子をもつ木です。
すべての木が二分木というわけではありません。

二分木ではない木を使う場合はあります。
たとえばたくさんの電話番号を木で表現する場合です。
この場合は各ノードが最大10個の子ノードのような10分木を使うことになります。



## 二分木の走査

In-Order（間順）、Pre-Order（前順）、Post-Order（後順）の走査は次のとおりです。
なお、これらの中でもっとも一般的なものはIn-Order走査です。

### In-Order Traverse

左の枝、現在のノード、右の枝の順序で訪れることを意味します。

次に再帰的アプローチを例示します。

```golang
// TreeNode is binary tree node definition
type TreeNode struct {
        Val   int
        Left  *TreeNode
        Right *TreeNode
}

func inorderTraversal(root *TreeNode) []int {
        out := make([]int, 0)

        var helper func(node *TreeNode)
        helper = func(node *TreeNode) {
                if root == nil {
                        return
                }
                if node.Left != nil {
                        helper(node.Left)
                }
                out = append(out, node.Val)
                if node.Right != nil {
                        helper(node.Right)
                }
        }

        helper(root)
        return out
}
```

https://play.golang.org/p/caIwRkdLJxS

### Pre-Order Traverse

まず初めに、現在のノードを訪れることです。
次に左の枝、最後に右の枝を訪れます。

次に再帰的アプローチを例示します。

```golang
// TreeNode is binary tree node definition
type TreeNode struct {
        Val   int
        Left  *TreeNode
        Right *TreeNode
}

func preorderTraversal(root *TreeNode) []int {
        out := make([]int, 0)

        var helper func(node *TreeNode)
        helper = func(node *TreeNode) {
                if node == nil {
                        return
                }
                out = append(out, node.Val)
                if node.Left != nil {
                        helper(node.Left)
                }
                if node.Right != nil {
                        helper(node.Right)
                }
        }

        helper(root)
        return out
}
```

https://play.golang.org/p/g8PRyXhAsWg


### Post-Order

TBD

## トライ木（trie tree）

トライ木（プレフィックス木と呼ばれることもあります）は各ノードに文字が保持されるn分木の一種で、木を下る経路が単語を表しています。

*ノード（nullノードという場合もある）は、完全な単語であることを示すために使われることがよくあります。[^1]

トライ木のノードは、どの場所でも０からアルファベットサイズ+1の子ノードを持ちます。

トライ木の利用でよく見られるのは、接頭辞検索用に言語全体を（英語）を保持する場合です。
ハッシュテーブルでは文字列が正しい単語かどうかを素早く検索できますが、文字列が正しい単語の接頭部であるかどうかの判別はできません。
しかしトライ木を使うと非常に素早く行うことができます。

Kを文字列の長さとすれば、 \(O(K)\) の実行時間で行うことができます。ハッシュテーブルでも同様です。[^2]

[^1]: 参考にしたソースでは、#ノードを使っていた。
[^2]: ハッシュテーブルを用いた検索の実行時間はO(1)ですが、入力のすべての文字を読み込まなければならず、そのため単語の検索にはO(K)の実行時間を要することになります。

トライ木ノード構造体は次のようになります。

```golang
type TrieNode struct {
	word     string
	children [26]*TrieNode
}
```

## 参考文献
- [世界で闘うプログラミング力を鍛える本 ~コーディング面接189問とその解法~](https://www.amazon.co.jp/dp/4839960100/ref=cm_sw_em_r_mt_dp_U_u83ZEbF120Y65)
- [212. Word Search II](https://leetcode.com/problems/word-search-ii/)
- [[Go] DFS + Trie solution](https://leetcode.com/problems/word-search-ii/discuss/542405/Go-DFS-+-Trie-solution)
