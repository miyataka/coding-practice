# 問題
https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/

二分木の root を受け取り、ノードの値を「ジグザグレベル順（最初のレベルは左から右へ、次のレベルは右から左へ、以降交互）」に、レベルごとの配列としてまとめて返す。

- 例1: root = [3,9,20,null,null,15,7] → [[3],[20,9],[15,7]]
- 例2: root = [1] → [[1]]
- 例3: root = [] → []
- 制約: ノード数は [0, 2000]、-100 <= Node.val <= 100

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

// 方針: BFSでたどりながら，現在のレイヤーを数えていき，その偶奇でreverseにするかを判定すればよい
// 気になるのはreverseにする処理の計算量だが，いったん作ってみる．
// 簡単に時間計算量と空間計算量を見積もる
// 入力サイズは，0 <= N <= 2000 で，これは全部のnodeをみるしかないので，O(N)，
// reverseにする処理がその時々のlayerのnodeを全部ループする．全部のlayer合計で，結局O(N)．足し合わせてO(2N)くらい
// 4000ステップだとする
// Goの秒間ステップを10^8とすると，4*10^-5=0.02ms=20μs
// 空間計算量は，結果を保持するのがO(N)．Valは -100 <= Node.Val <= 100でint64に十分収まるので，int64(8byte)が2000個なので，16KB程度．
func zigzagLevelOrder(root *TreeNode) [][]int {
    result := [][]int{}
    if root == nil {
        return result
    }

    layerNum := 0
    frontier := []*TreeNode{root}
    for len(frontier) > 0 {
        nextLayer := []*TreeNode{}
        sameLayerVals := []int{}

        for _, node := range frontier {
            sameLayerVals = append(sameLayerVals, node.Val)
            if node.Left != nil {
                nextLayer = append(nextLayer, node.Left)
            }
            if node.Right != nil {
                nextLayer = append(nextLayer, node.Right)
            }
        }

        if layerNum % 2 == 1 {
            l := len(sameLayerVals)
            reversed := make([]int, l)
            for i, v := range sameLayerVals {
                reversed[l-1-i] = v
            }
            sameLayerVals = reversed
        }

        frontier = nextLayer
        result = append(result, sameLayerVals)
        layerNum++
    }
    return result
}
```
- ミスなくpassed
- セルフレビュー
    - 逆順にする処理は，集めた値をreversedにするやりかたと，frontierのループを回す前に逆順にしてしまうやり方があるなと思った. 関数に切り出したら綺麗になるかも.
- 他の選択肢について想いを馳せる
    - treeだから，BFSだけじゃなくてDFSもあるよな．
        - DFSは自分にとっては自然なアイデアじゃないからあまり好まないが，まぁ次で一度書いてみよう．
    - iterativeとrecursiveがある．まぁrecursiveにするとスタックオーバーフローが気になるのでやりたくないけど．まぁ2000回なら問題はなさそうだけれども．
- かかった時間は18分ほど

# 2回目 (BFS)
```go
import "slices"

func zigzagLevelOrder(root *TreeNode) [][]int {
    result := [][]int{}
    if root == nil {
        return result
    }

    depth := 0
    frontier := []*TreeNode{root}
    for len(frontier) > 0 {
        nextLayer := []*TreeNode{}
        sameLayerVals := []int{}

        for _, node := range frontier {
            sameLayerVals = append(sameLayerVals, node.Val)
            if node.Left != nil {
                nextLayer = append(nextLayer, node.Left)
            }
            if node.Right != nil {
                nextLayer = append(nextLayer, node.Right)
            }
        }

        if depth % 2 == 1 {
            slices.Reverse(sameLayerVals)
        }

        frontier = nextLayer
        result = append(result, sameLayerVals)
        depth++
    }
    return result
}
```
- これはBFSをリファクタしたver
- slices.Reverseは便利

# 2回目 (DFS)
```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
// 方針: DFSで書いてみる
// DFSだと，layerをまとめるのどうやるんだろう．なんか[][]intの型の変数にappendしていけばまぁいいのか．
// resultに追加するときに，外側のindexの偶奇で，先頭に追加するか末尾に追加するかを分けてやればよさそう
func zigzagLevelOrder(root *TreeNode) [][]int {
    result := [][]int{}
    if root == nil {
        return result
    }

    type nodeWithDepth struct {
        node *TreeNode
        depth int
    }

    toVisit := []nodeWithDepth{
        {node: root, depth: 0},
    }
    for len(toVisit) > 0 {
        v := toVisit[len(toVisit)-1]
        toVisit = toVisit[:len(toVisit)-1]
        node, depth := v.node, v.depth

        // 未踏のresult[depth]に到達したら拡張しておく. ツリーを辿るので，一つずつしかdepth増えないので一つの拡張でよい
        if len(result) < depth + 1 {
            result = append(result, []int{})
        }

        if depth % 2 == 0 {
            result[depth] = append(result[depth], node.Val)
        } else {
            result[depth] = append([]int{node.Val}, result[depth]...)
        }

        // leftから探索してほしいので，toVisitにはRight->Leftの順で詰める
        if node.Right != nil {
            toVisit = append(toVisit, nodeWithDepth{node: node.Right, depth: depth + 1})
        }
        if node.Left != nil {
            toVisit = append(toVisit, nodeWithDepth{node: node.Left, depth: depth + 1})
        }
    }

    return result
}
```
- DFSで書いたらめちゃバグらせて時間を使ってしまったのでLLMに助けてもらった．`// 未踏の〜の`部分．自分の考え方に合っていない

- 他の人のコードを読む

- https://github.com/chryschron/codings/pull/26
    - BFSで大体同じ方針
    - 議論されていることは雰囲気だけしかわかっていない．vectorとdequeで先頭への挿入がかわるようだ．
    - あらかじめサイズがわかるので，vectorをin-placeでreverseにするのがよいらしい．これはGoにも言えること
- https://github.com/jjysogfy/arai60-202603/pull/16
    - 同じ方針だ
- https://github.com/skypenguins/coding-practice/pull/41
    - > `deque` オブジェクトは空になっても `None` にはならないため、`is not None` は常に `True` であるため
    - 知らなかった. pythonの知識が増えた
    - if-elseのほうがよいというのは同意．
- https://github.com/Hiroto-Iizuka/coding_practice/pull/27
    - https://github.com/Hiroto-Iizuka/coding_practice/pull/27/changes#r3490906125 のコメントはpython力が低くて，わからなかった
    - https://docs.python.org/3/library/stdtypes.html#range
        - rangeの第三引数で「いくつごとに処理をするか」を指定できるのか
- https://github.com/hiroki-horiguchi-dev/leetcode/pull/27
    - いまさら思ったが，自分の計算量の見積もりがみんなと微妙にずれてるな．レビュワーのコメントがほしいところ．
    - isXXX は Rubyの慣習がそうだったので，自分もよくやる．C++の慣習とかだと違うんだな．まぁチームに合わせればよい部分であろう
    - tempはもうちょい良い名前がありそうな気がする．
- https://github.com/nicah4o/arai60/pull/26
    - step1の`int level = nodeq.size();`の，`level`という命名はすこし変な感じ．
    - dfsという命名は好みではない
- https://github.com/dorxyxki/arai60/pull/27
    - 計算量がなんでそうなるのかはコメントが欲しいなと思った
    - 全然関係ないが，ディスコードをディスコと書くこともあるんだな．別物じゃんという気持ちになった
    - 偶奇ではなく，フラグを反転させていくというのはなるほどなぁ
    - 最終系も `if root is None` とかでもいいかもなぁ
- https://github.com/rimokem/arai60/pull/27
    - 他の人も書いているが読みやすい．
    - 計算量についてふれてもいいかもしれない．N<=2000なので，心配するほどではないという捉え方もできる
- https://github.com/h-masder/Arai60/pull/30
    - pythonの`[::-1]`とか全然知らなかったな
        - これドキュメントの該当ページがわからんな．

# 3回目
```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */

import "slices"

func zigzagLevelOrder(root *TreeNode) [][]int {
    result := [][]int{}
    if root == nil {
        return result
    }

    depth := 0
    frontier := []*TreeNode{root}
    for len(frontier) > 0 {
        nextLayer := []*TreeNode{}
        sameLayerVals := []int{}

        for _, node := range frontier {
            sameLayerVals = append(sameLayerVals, node.Val)
            if node.Left != nil {
                nextLayer = append(nextLayer, node.Left)
            }
            if node.Right != nil {
                nextLayer = append(nextLayer, node.Right)
            }
        }

        if depth%2 == 1 {
            slices.Reverse(sameLayerVals)
        }

        frontier = nextLayer
        result = append(result, sameLayerVals)
        depth++
    }

    return result
}
```
- これを3回繰り返した
