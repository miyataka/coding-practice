# 問題
https://leetcode.com/problems/binary-tree-level-order-traversal/

二分木の root を受け取り、ノードの値を「レベル順（上から下へ、各レベルは左から右へ）」に、レベルごとの配列としてまとめて返す。

- 例1: root = [3,9,20,null,null,15,7] → [[3],[9,20],[15,7]]
- 例2: root = [1] → [[1]]
- 例3: root = [] → []
- 制約: ノード数は [0, 2000]、-1000 <= Node.val <= 1000

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。


# 1回目
```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */

// 方針: なんとなく，BFSなのだろうという気がするが．．．
// layer単位でループを回し，BFSを行う．left->rightの順が求められているので，それは気をつける．
// inputが 0 <= N <= 2000
// 時間計算量: 全ノードを辿る必要があるので，O(N)
// 空間計算量: 結果を返すために全ノードの数値を保持する必要があるので，O(N)
func levelOrder(root *TreeNode) [][]int {
    result := [][]int{}
    frontier := []*TreeNode{root}

    for len(frontier) > 0 {
        nextLayer := []*TreeNode{}
        layerValues := []int{}

        for _, node := range frontier {
            if node == nil {
                continue
            }
            layerValues = append(layerValues, node.Val)
            nextLayer = append(nextLayer, node.Left)
            nextLayer = append(nextLayer, node.Right)
        }
        frontier = nextLayer
        if len(layerValues) != 0 {
            result = append(result, layerValues)
        }
    }
    return result
}
```
- 何度かfailさせつつ，passed
    - `if len(layerValues) != 0` の漏れなど
- 見直したが，あまり嫌なところはない．強いて言えば`if len(layerValues) != 0`の箇所だが，これは避けられないように思える
- 他の選択肢について思いを馳せる
    - 再帰でも一応できるはずだけど，嬉しさなさそう．
    - レイヤーごとに処理しないBFSも一応はあるけど，分けて処理していたのは正解
    - DFSでも一応解けるんだろうけど，全然自然じゃないと感じるのでこれは考えない

# 2回目
```go
func levelOrder(root *TreeNode) [][]int {
    result := [][]int{}
    if root == nil {
        return result
    }
    frontier := []*TreeNode{root}

    for len(frontier) > 0 {
        nextLayer := []*TreeNode{}
        layerValues := []int{}

        for _, node := range frontier {
            layerValues = append(layerValues, node.Val)
            if node.Left != nil {
                nextLayer = append(nextLayer, node.Left)
            }
            if node.Right != nil {
                nextLayer = append(nextLayer, node.Right)
            }
        }
        frontier = nextLayer
        result = append(result, layerValues)
    }
    return result
}
```
- 全部キューに積んでからnilをskipする．という方針から，そもそもnilは積まないという方針へ
- これは正直どっちでもいいな．あまり綺麗になったとも今のところ思わない．最初にrootのnilチェックするという処理は増えているしなぁ．
    - でも一方で，resultをappendするときのifも消せているのか
- 他の人のPRをみる
    - https://github.com/chryschron/codings/pull/25
        - DFSも解いている．えらい
        - iterativeのDFSで，queueにright, leftの順でpushするのが，ちょっとアレと思ったけど，コードは正しかった
            - よく読んだらstackという命名にちゃんとなっていた
    - https://github.com/skypenguins/coding-practice/pull/40
        - level_by_level, あまり意図わからんかった．でも修正されてた
    - https://github.com/Hiroto-Iizuka/coding_practice/pull/26
        - step1のコメントはよくわからんかった．
        - `node_with_level`という変数名は複数形にしたいと思った
        - step3は同じ方針になってそう
    - https://github.com/hiroki-horiguchi-dev/leetcode/pull/26
        - なぜ階層分けした状態にresultがなるのか，一瞬迷った．外側のループで，nodes.size()するときに，ループ回数が固定されるからだという理解をした．
    - https://github.com/nicah4o/arai60/pull/25
        - step3が，dfsという関数名なのがすこし気になった.
        - https://github.com/tom4649/Coding/pull/146#discussion_r3566048414 のようなこと．実装の詳細ではなく，なにが起きるのか，何を返すのかがわかる命名にしたい．
    - https://github.com/dorxyxki/arai60/pull/26
        - if/whileの話を読んだ（感想ではない）
    - https://github.com/h-masder/Arai60/pull/29
        - `if not xxx` と `if xxx is None` がどちらもあるのが気になった．今回は`Optional[TreeNode]`なので `if root is None` に統一でよいのではと思った．python詳しくないけど．
    - https://github.com/rimokem/arai60/pull/26
        - `node_groups` という命名はコメントにあるとおり
        - 別にこの方に限った話ではないのだが，pythonを使っている方で，関数内で別途関数を定義して，関数内部のローカル変数を関数内関数にとってのグローバル変数のようにして再帰などをしている例をよくみる．ある意味，副作用がある関数なので，なるべく避けたい気持ちになる．

# 3回目
```go
func levelOrder(root *TreeNode) [][]int {
    result := [][]int{}
    if root == nil {
        return result
    }

    frontier := []*TreeNode{root}
    for len(frontier) > 0 {
        nextLayer := []*TreeNode{}
        layerValues := []int{}

        for _, node := range frontier {
            layerValues = append(layerValues, node.Val)
            if node.Left != nil {
                nextLayer = append(nextLayer, node.Left)
            }
            if node.Right != nil {
                nextLayer = append(nextLayer, node.Right)
            }
        }
        frontier = nextLayer
        result = append(result, layerValues)
    }
    return result
}
```
- これを3回繰り返した
