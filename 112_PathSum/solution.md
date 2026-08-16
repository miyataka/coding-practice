# 問題
https://leetcode.com/problems/path-sum/

二分木の root と整数 targetSum を受け取る。root から葉（leaf）までのパスに沿ったノードの値の合計が targetSum に一致するパスが存在するかを bool で返す。

- 例1: root = [5,4,8,11,null,13,4,7,2,null,null,null,1], targetSum = 22 → true（5→4→11→2 の合計が22）
- 例2: root = [1,2,3], targetSum = 5 → false
- 例3: root = [], targetSum = 0 → false
- 制約: ノード数は [0, 5000]、-1000 <= Node.val <= 1000、-1000 <= targetSum <= 1000

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
// 方針: 一個見つければ終了してよさそうなので，DFSでもBFSでもよさそうだ．ただし，leafに行った時点での判定なので，DFSのほうが直感的に思える
// 注意すべきはexample3で，treeがemptyだと一致ではないという判定をする必要がある
// DFSで，その時点までのpathSumを引数として渡していく．leafが自分自身と足し合わせた時にtargetSumと一致していたらtrueを返す．
// 入力の長さは 0 <= N <= 5 * 10^3
// 最悪ケースですべてのノードを辿ったとする，かつGoの秒間ステップ数を10^8とすると
// 5 * 10^-5 なので，0.05 ms程度
// メモリは，最悪ケースで5 * 10^3回の再帰．引数は，root(ポインタ)，targetSum(int), pathSum(int)とすると，8*3 = 24byte, リターンアドレスで+8B, ローカル変数はあとで考慮するとして，ここまでで(32+x) * 5 * 10^3 = 160KB+5xKB程度. goroutineの1GB上限には達しなさそうだ．

func hasPathSum(root *TreeNode, targetSum int) bool {
    return hasPathSumFunc(root, targetSum, 0)
}

func hasPathSumFunc(root *TreeNode, targetSum, pathSum int) bool {
    if root == nil {
        return false
    }
    // 停止条件
    if root.Left == nil && root.Right == nil {
        return root.Val + pathSum == targetSum
    }
    pathSum += root.Val
    left := hasPathSumFunc(root.Left, targetSum, pathSum)
    right := hasPathSumFunc(root.Right, targetSum, pathSum)
    return left || right
}
```
- これでpassed.
- セルフレビュー
    - hasPathSumFuncの命名はどうにかしたい
    - そのなかでpathSumを上書きするのもちょっと嫌
    - 再帰で書いてあるから，やっぱスタックオーバーフローは気になる．ループで書き直したい
    - これはトップダウンの感じ．これまでの処理結果を上から渡すので．
    - targetSumとpathSumを二つ渡していく必要はあるんだろうか？
        - これはあれか，差をとってそれを渡していって，leaf時点で，0になればいいのか．すこしだけシンプルにできそうだ．
- 他の選択肢に思いを馳せる
    - BFSもまああり．イメージとしては，複数チームに同時に指令をだして，最初に条件に一致したものが見つかった時点で打ち止めする，ような感じ
    - ブルートフォースは，特に思い浮かばない．DFSも全部たどるので愚直解みたいなモノだしな．

# 2回目
```go
func hasPathSum(root *TreeNode, targetSum int) bool {
    if root == nil {
        return false
    }
    // 停止条件
    if root.Left == nil && root.Right == nil {
        return targetSum - root.Val == 0
    }
    nextTargetSum := targetSum - root.Val
    left := hasPathSum(root.Left, nextTargetSum)
    right := hasPathSum(root.Right, nextTargetSum)
    return left || right
}
```

# 2回目 ループ
```go
func hasPathSum(root *TreeNode, targetSum int) bool {
    type nodeAndPathSum struct {
        node *TreeNode
        pathSum int
    }
    toVisit := []nodeAndPathSum{
        {node: root, pathSum: 0},
    }

    found := false
    for len(toVisit) > 0 {
        v := toVisit[len(toVisit)-1]
        toVisit = toVisit[:len(toVisit)-1]
        node, pathSum := v.node, v.pathSum

        if node == nil {
            continue
        }
        nextPathSum := pathSum + node.Val
        if node.Left == nil && node.Right == nil {
            if nextPathSum == targetSum {
                found = true
                break
            }
        }
        toVisit = append(toVisit, nodeAndPathSum{node: node.Left, pathSum: nextPathSum})
        toVisit = append(toVisit, nodeAndPathSum{node: node.Right, pathSum: nextPathSum})
    }
    return found
}
```
- 問題なく通った．
- 他の人のコードをみにいく
    - https://github.com/chryschron/codings/pull/24
        - step1に対して，if (codition) { return true } としている箇所がすこし気になる．それが2回繰り返されているのも．以下のように書きたい気持ち. syntaxには自信はない．
        ```cpp
        bool left = hasPathSum(root->left, target_sum - root->val)
        bool right = hasPathSum(root->right, target_sum - root->val)
        return left || right
        ```
        - 引数に渡すときにはもう足しておくこともたしかにできるのか．ちょっとずれる感じがして好みではないが了解，という感じ
        - step3は綺麗で読みやすい
    - https://github.com/jjysogfy/arai60-202603/pull/15
        - step1から綺麗．これは引きながら渡していくスタイル．
        - 事前に引いていくスタイルも二度目だと，まぁたしかに自然に思えてきた
        - step2のレコードは，nodeとnum, numという命名はもうすこし改善の余地ありそうな気もする．`rest` とか？うーむ．
    - https://github.com/Hiroto-Iizuka/coding_practice/pull/25
        - pythonのtupleはイミュータブルなんだという学び
        - remainingという命名はいいですね
        - と思ったらstep2でprefix_sumになってた．これはこれでよさそう
        - step3はないのかな
    - https://github.com/hiroki-horiguchi-dev/leetcode/pull/25
        - recordの定義，先にしてくれたほうが個人的には嬉しいな．慣習的にはどっちなんでしょうね
        - 終了判定をする前に，一度queueに詰めるのが先なのが，なんとなく自分と違うなと思った
        - `NodeAndPrefixSum` のlistが，nodesという変数名なのはすこし気になる．名前とずれている気がして．
    - https://github.com/nicah4o/arai60/pull/24
        - `return left_value || right_value` のほうが自然というのは同意
        - 47,48行目のifには`{}`がほしい
        - この人に限らないんだが，計算量O(N)というのはどう考えた結果O(N)としたのかは，気になる
    - https://github.com/dorxyxki/arai60/pull/25
        - 読みやすい
        - あまりpythonの作法を知らないんだが，以下のようなのはよくあることなのだろうか．ちょっとだけ面食らう
        ```python
        def xxx
            def yyy

        ```
    - https://github.com/attractal/leetcode/pull/37
        - `if root is None`のほうがよいかもしれない
        - if xxx and not yyy はちょっと負荷高まった気がする．自分は避ける．
    - https://github.com/hiro111208/leetcode/pull/19
        - どんどんネストが深くなるなぁ，ちょっと下げたいなぁの気持ち
    - https://github.com/h-masder/Arai60/pull/28
        - [2]の解法は，layer分けをしているんだなと思ったけど，今回の問題はする必要がなさそうと思った
        - 再帰にするとき，関数名困るけど，helperというのはよいかわしかただと思った
        - フォローアップにも取り組んでいて素晴らしいなと思った
    - https://github.com/rimokem/arai60/pull/25
        - 感じることは全部コメントされていますね
    - https://github.com/Manato110/LeetCode-arai60/pull/25
        - いい感じだ．2分は速い
        - 空間計算量はO(N)
            - 同上で済ませられているけど，本当だろうか？ループの度にpopされるのでそこまでいかない気もするが勘違いかな．あるいはpopしたとて定数倍だから無視できるぽいかな

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
func hasPathSum(root *TreeNode, targetSum int) bool {
    type nodeAndRemaining struct {
        node *TreeNode
        remaining int
    }

    toVisit := []nodeAndRemaining{{node: root, remaining: targetSum}}

    found := false
    for len(toVisit) > 0 {
        v := toVisit[len(toVisit)-1]
        toVisit = toVisit[:len(toVisit)-1]
        node, remaining := v.node, v.remaining

        if node == nil {
            continue
        }
        if node.Left == nil && node.Right == nil {
            if remaining == node.Val {
                found = true
                break
            }
        }
        toVisit = append(toVisit, nodeAndRemaining{node: node.Left, remaining: remaining - node.Val})
        toVisit = append(toVisit, nodeAndRemaining{node: node.Right, remaining: remaining - node.Val})
    }
    return found
}
```
- これを3回繰り返した
