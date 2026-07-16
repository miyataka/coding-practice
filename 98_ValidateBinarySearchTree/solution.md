# 問題
https://leetcode.com/problems/validate-binary-search-tree/

二分木の root を受け取り、それが有効な二分探索木（BST）かどうかを bool で返す。

有効なBSTの定義:
- 各ノードの左部分木のすべてのノードの値は、そのノードの値より小さい（strict）
- 各ノードの右部分木のすべてのノードの値は、そのノードの値より大きい（strict）
- 左右の部分木もまた有効なBSTであること

- 例1: root = [2,1,3] → true
- 例2: root = [5,1,4,null,null,3,6] → false（root の右部分木に root より小さい 3 が含まれる）
- 制約: ノード数は [1, 10^4]、-2^31 <= Node.val <= 2^31 - 1

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。

# 1回目 （これは動かない）

```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
// 方針: DFSで，validationを走らせながら，全走査すればよい．
// 平衡二分探索木ではなくニ分探索木であればよいので，高さの情報などは持ち回る必要はなさそう
// BFSでもいけそうだけど，全走査なのでどっちでもよさそう．invalidなものがでてきたら早目に打ち切ることもできる．最短経路とかではないので，DFS/BFSどちらも変わらない
// 計算量・実行時間について見積もる
// 1 <= N <= 10^4
// -2^31 <= Node.Val <= 2^31 - 1
// valはint32に収まるということ．足し合わせるとかは多少注意が必要そう．
// validationを満たすかは定数ステップとして，全走査するしかないので，時間計算量はO(N)
// Goの秒間ステップ数を10^8とすると，
// 10^4/10^8 = 10^-4 = 0.1ms
// 空間計算量は，再帰ならある程度かかりそうだけど，iterativeなら，parentのvalと自分自身のvalの比較，左右どちらなのかというフラグがあればよさそう. O(1)かな？

func isValidBST(root *TreeNode) bool {
    if root == nil {
        return false
    }

    toVisit := []*TreeNode{root}
    for len(toVisit) > 0 {
        node := toVisit[len(toVisit)-1]
        toVisit = toVisit[:len(toVisit)-1]

        if node.Left != nil {
            if node.Left.Val < node.Val {
                toVisit = append(toVisit, node.Left)
            } else {
                return false
            }
        }

        if node.Right != nil {
            if node.Val < node.Right.Val {
                toVisit = append(toVisit, node.Right)
            } else {
                return false
            }
        }
    }

    return true
}
```
- 問題を読み違えていたのに気づいた
- しかしリトライしても解けなくて時間を使いすぎたのでLLMに回答をもらったのが下記

## LLMに答えを教えてもらった

```go
func isValidBST(root *TreeNode) bool {
    return isInRange(root, math.MinInt64, math.MaxInt64)
}

// (lower, upper) の開区間に部分木の全ノードが収まっているか
func isInRange(node *TreeNode, lower, upper int) bool {
    if node == nil {
        return true
    }
    if node.Val <= lower || upper <= node.Val {
        return false
    }
    return isInRange(node.Left, lower, node.Val) &&
        isInRange(node.Right, node.Val, upper)
}
```
- なぜこれで解けるのか最初わからなかったが，数直線を分割していくイメージと言われてやっと理解できた気がする

# 2回目

## iterative DFS
```go
import "math"

func isValidBST(root *TreeNode) bool {
    if root == nil {
        return true
    }

    type nodeWithRange struct {
        node *TreeNode
        lower int
        upper int
    }

    toVisit := []nodeWithRange{
        {node: root, lower: math.MinInt64, upper: math.MaxInt64},
    }
    for len(toVisit) > 0 {
        v := toVisit[len(toVisit)-1]
        toVisit = toVisit[:len(toVisit)-1]
        node := v.node

        if node.Val <= v.lower || v.upper <= node.Val {
            return false
        }

        if node.Left != nil {
            toVisit = append(toVisit, nodeWithRange{node: node.Left, lower: v.lower, upper: node.Val})
        }
        if node.Right != nil {
            toVisit = append(toVisit, nodeWithRange{node: node.Right, lower: node.Val, upper: v.upper})
        }
    }
    return true
}
```
- iterativeでも書いた．一度イメージを掴めると書ける．
- 他の人のコードを読む
    - https://github.com/chryschron/codings/pull/27
        - in-order, pre-orderって自分とは全然違う解法だった．あとでもう一度読む
        - 雑にググると，DFSで辿るときの数え上げ順だった．DFSに種類があることを認識してなかったので覚えておく
    - https://github.com/skypenguins/coding-practice/pull/39
        - in-orderは日本語で言うと中順走査らしい．
        - さくらんぼみたいな単純なtreeを想定して，真ん中のrootをいつ数えるかでpre-order, in-order, post-orderと呼ぶみたい．
    - https://github.com/dorxyxki/arai60/pull/28/
        - pre-order, in-order, post-orderの解法を書いているが，それぞれのイメージがつかめていないので，全然頭に入らない．書けるのすごいな
    - https://github.com/rimokem/arai60/pull/28
        - step2のL46-L50あたりは行が詰まってて多少読みづらい．趣味の範囲とも思う
    - https://github.com/nicah4o/arai60/pull/27
        - cppはむずい感じしちゃう. だが読むときの許容度を広げることが大事というのもわかる．
            - 難しく感じてしまうのは，まだ理解が弱いんだろうなと思う．一度理解してしまえば，めちゃくちゃ読みやすくなるので．
        - 快楽が大きいわかる．俺も1時間くらいかけてしまった
    - https://github.com/jjysogfy/arai60-202603/pull/13
        - in-order traversalがあまりわかった気がしていない...
    - https://github.com/h-masder/Arai60/pull/31/
        - `筋が悪いコード例も書いておくとよいと思うので書く。` この考えは自分にはなかったかも．取り入れたい
            - odaさんがなんかどっかでボール球を投げてみる必要がある，と書かれていたような

# 3回目
## recursive
```go
import "math"

func isValidBST(root *TreeNode) bool {
    return isInRange(root, math.MinInt64, math.MaxInt64)
}

func isInRange(node *TreeNode, lower, upper int) bool {
    if node == nil {
        return true
    }
    if node.Val <= lower || upper <= node.Val {
        return false
    }
    return isInRange(node.Left, lower, node.Val) &&
        isInRange(node.Right, node.Val, upper)
}
```

## iterative
```go
import "math"

func isValidBST(root *TreeNode) bool {
    if root == nil {
        return true
    }

    type nodeWithRange struct {
        node *TreeNode
        lower int
        upper int
    }

    toVisit := []nodeWithRange{{node: root, lower: math.MinInt64, upper: math.MaxInt64}}

    for len(toVisit) > 0 {
        v := toVisit[len(toVisit)-1]
        toVisit = toVisit[:len(toVisit)-1]
        node := v.node

        if node.Val <= v.lower || v.upper <= node.Val {
            return false
        }
        if node.Left != nil {
            toVisit = append(toVisit, nodeWithRange{node: node.Left, lower: v.lower, upper: node.Val})
        }
        if node.Right != nil {
            toVisit = append(toVisit, nodeWithRange{node: node.Right, lower: node.Val, upper: v.upper})
        }
    }
    return true
}
```
- それぞれ3回繰り返した
