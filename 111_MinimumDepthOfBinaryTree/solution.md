# 問題
https://leetcode.com/problems/minimum-depth-of-binary-tree

二分木 `root` が与えられる。その**最小の深さ**を返す。最小の深さとは、根ノードから最も近い**葉ノード**までの経路上にあるノード数。葉ノードとは子を持たないノード。

- 例1: `root = [3,9,20,null,null,15,7]` → `2`
- 例2: `root = [2,null,3,null,4,null,5,null,6]` → `5`
- 例3: `root = []` → `0`（空の木）
- 制約: ノード数は `[0, 10^5]` の範囲、`-1000 <= Node.val <= 1000`

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。
---



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

 // 方針: 幅優先探索を利用して，一番最初にLeftとRightが両方nullになるところで探索をストップする．
 // 再帰のパターンと，キューを使ったループでの実装がありうる．
 // スタックオーバーフローになるかを検討するために，ざっくり計算量を見積もる．
 // nodeの数は, 0 <= N <= 10^5
 // Go言語だと，スタックメモリは1GB程度だったはず．あとで再確認する．
 // 10^9 Byteなので，一つのスタックが10^3程度までならギリ収まりそう．スタックオーバーフローを気にしたくはないので，ループで書く
 // とりうる値は，-1000 <= M <= 1000 で，これはint16程度で収まるので気にしない．
 // 時間計算量は，あとで考える
func minDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    depth := 1
    frontier := []*TreeNode{root}

    for 0 < len(frontier) {
        nextFrontier := []*TreeNode{}
        for _, node := range frontier {
            if node == nil {
                continue
            }
            if node.Left == nil && node.Right == nil {
                return depth
            }
            nextFrontier = append(nextFrontier, node.Left)
            nextFrontier = append(nextFrontier, node.Right)
        }
        frontier = nextFrontier
        depth++
    }
    return 0 // ここにはこないはず
}
```
- 時間計算量は，frontierが最悪10^5回ループするので，O(N)
- 空間計算量は，nextFrontierの最大サイズとなりそうだが，これはバイナリツリーの高さをHとすると，2^H
    - 高さHは，logNなので，O(2^logN) かな？
    - 違った．balanced（平衡木）ではないので，高さも最大はNとなる. よって空間計算量もO(N)
- Goのgoroutine一つあたりのstack size上限は1GBであっている https://github.com/golang/go/blob/master/src/runtime/proc.go#L160-L167
- rootのnilチェックを統合できそうな気がしたが，特別扱いのifがループの中に入るだけのようなので，外にあったほうが素直だなと思った


# 2回目（再帰ver）
```go
func minDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    return minDepthFromLevel([]*TreeNode{root}, 1)
}

func minDepthFromLevel(level []*TreeNode, depth int) int {
    if len(level) == 0 {
        return depth - 1
    }
    next := []*TreeNode{}
    for _, node := range level {
        if node.Left == nil && node.Right == nil {
            return depth
        }
        if node.Left != nil {
            next = append(next, node.Left)
        }
        if node.Right != nil {
            next = append(next, node.Right)
        }
    }
    return minDepthFromLevel(next, depth+1)
}
```

# 2回目（DFS再帰ver）
```go
func minDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }
    if root.Left == nil {
        return minDepth(root.Right) + 1
    }
    if root.Right == nil {
        return minDepth(root.Left) + 1
    }
    return min(minDepth(root.Left), minDepth(root.Right)) + 1
}
```
- 再帰を書いてみた．ちょっとつまづいたので，LLMからヒントをもらった．
- BFSのループから再帰への書き換えの理解を書いておく
    - queueを受け渡す必要がある．あとは渡したい情報があればそれも渡す（今回はdepth）
    - 停止条件を書く．
        - 停止したら，全てが巻き戻り，そのままreturn
    - 他はループのときと同じ処理
    - 次のlevel，frontierに行く時は，depthをカウントアップしたものを渡す
- 感想．BFSはループのほうがしっくりくる．
- 他の人のコードをみる
    - https://github.com/komdoroid/arai60/pull/18
    - https://github.com/hiroki-horiguchi-dev/leetcode/pull/22
    - https://github.com/jjysogfy/arai60-202603/pull/11
    - https://github.com/nicah4o/arai60/pull/21
        - layer分けをしないBFS．
        - cppには詳しくないが，queueはpopで値を返さないんだなというのを知った．dequeはpop_frontというのがあるらしい
    - https://github.com/attractal/leetcode/pull/31
        - DFSのときの関数名が `traverse` というのはなるほど．
    - https://github.com/hiro111208/leetcode/pull/16
        - float("inf") は番兵的なことかな. `10 ** 5` だとわかりにくかった. resという変数名も
    - https://github.com/h-masder/Arai60/pull/23
        - 自分と似ている．DFSとBFSを両方やっている
    - https://github.com/rimokem/arai60/pull/22
        - DFSでもminを使えばいけるのか．それはそうか．
- DFSの再帰は書いてみた


# 3回目
```go
func minDepth(root *TreeNode) int {
    if root == nil {
        return 0
    }

    depth := 1
    frontier := []*TreeNode{root}

    for 0 < len(frontier) {
        nextFrontier := []*TreeNode{}
        for _, node := range frontier {
            if node == nil {
                continue
            }
            if node.Left == nil && node.Right == nil {
                return depth
            }
            nextFrontier = append(nextFrontier, node.Left)
            nextFrontier = append(nextFrontier, node.Right)
        }
        frontier = nextFrontier
        depth++
    }
    return depth
}
```
- これを3回書いた
- 個人的に一番しっくり来ている．layer分けしたBFSのループを使いたいので．
