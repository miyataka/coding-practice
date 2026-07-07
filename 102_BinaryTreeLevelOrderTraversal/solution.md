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

# 見積もり
- 入力サイズ: N = ノード数 ≤ 2000
- 時間計算量: O(N)
  - 実行時間の見積もり: <!-- 例: 2*10^3 / 10^8 = 2*10^-5 s ≈ 0.02ms (Go) -->
- 空間計算量: <!-- 例: O(N)（出力＋キュー分） -->
  - 必要メモリの見積もり: <!-- 例: N * (要素サイズ) 程度 -->

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
- 見直したが，あまり嫌なところはない．強いて言えば`if len(layerValues) != 0`の箇所だが，これは避けられないように思える
- 他の選択肢について思いを馳せる
    - 再帰でも一応できるはずだけど，嬉しさなさそう．
    - レイヤーごとに処理しないBFSも一応はあるけど，分けて処理していたのは正解
    - DFSでも一応

# 2回目
