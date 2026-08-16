# 問題
https://leetcode.com/problems/maximum-depth-of-binary-tree

二分木 `root` が与えられる。その**最大の深さ**を返す。最大の深さとは、根ノードから最も遠い葉ノードまでの経路上にあるノード数。

- 例1: `root = [3,9,20,null,null,15,7]` → `3`
- 例2: `root = [1,null,2]` → `2`
- 例3: `root = []` → `0`（空の木）
- 制約: ノード数は `[0, 10^4]` の範囲、`-100 <= Node.val <= 100`

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。
---

# 見積もり
- 入力サイズ: 0 <= ノード数 <= 10^4
- 時間計算量: O(N)（全ノードをちょうど1回ずつ処理）
  - 実行時間の見積もり: 10^4 / 10^8 = 10^-4 s = 0.1ms
- 空間計算量: O(H)（H = 木の高さ）
  - 平衡木なら H = log N、最悪（連結リスト状）なら H = N で O(N)
  - 必要メモリの見積もり: 最悪 10^4 段の再帰スタック。1フレームの引数がint,pointer,pointerなので，24byte，そしてreturn-addressがあるはずなので，32byte.
      32byte * 10^4 = 320kb


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


func maxDepth(root *TreeNode) int {
    maxDepth := 0
    maxDepth = countDepth(root)
    return maxDepth
}

func countDepth(node *TreeNode) int {
    if node == nil {
        return 0
    }
    if node.Left == nil && node.Right == nil {
        return 1
    }

    leftDepth := 0
    rightDepth := 0
    if node.Left != nil {
        leftDepth = countDepth(node.Left)
    }
    if node.Right != nil {
        rightDepth = countDepth(node.Right)
    }
    return max(leftDepth, rightDepth) + 1
}
```
- treeに関する知識がなく，なんとなく解いた．
    - なんとなく解いたから，コードが整理されていないと感じる．
- あと問題の図と，exampleのinputと，go言語で与えられるinputの`*TreeNode`が違うもの渡されている感じがして，すこし戸惑った

# 2回目
```go
func maxDepth(root *TreeNode) int {
    return countDepth(root)
}

func countDepth(node *TreeNode) int {
    if node == nil {
        return 0
    }
    if node.Left == nil && node.Right == nil {
        return 1
    }

    return max(countDepth(node.Left), countDepth(node.Right)) + 1
}
```
- 短くはなったが，まだ自然には感じられない
- 日本語にすると
    - nilが渡されたら0にする
    - nilではないが，次のリーフ（LeftとRight)がどちらもnilなら1として返す
    - どちらかのリーフがあるとき，両方を調べて，大きい方と現階層の分(1)を足し合わせて返す
- うーむ，多少わかるようなわからんような．まだ頭に馴染まない．
- 典型コメント https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0
    - pythonには `nonlocal` というのがあるのね. ついでに `global` もあるらしい．
        - https://docs.python.org/3.14/reference/simple_stmts.html#the-nonlocal-statement
    - リンク先に飛ぶとgo, backとかが出てくるコードがあるが，直接は関係なさそうな概念が定義されていて読みづらかった
    - DFSはstackを使ってループにすることができるのはその通り．自分でもやるかな．これは入力の制限が，緩和されたときにスタックオーバーフローにならないため，くらいでとらえるとよさそう
- 他の人のコード
    - https://github.com/komdoroid/arai60/pull/17
    - https://github.com/hiroki-horiguchi-dev/leetcode/pull/21
    - https://github.com/nicah4o/arai60/pull/20
        - ボトムアップ，トップダウンの話がわからないなと思ったので，LLMに聞いてみた．引数でdepthを渡して，一番下にきたときにすでにdepthが求まっているか，あるいは一番下から上に戻っていく過程でカウントされるかの違いと理解した．自分のstep1/step2は，ボトムアップ．
    - https://github.com/Kazuuuuuuu-u/arai60/pull/22
    - https://github.com/jjysogfy/arai60-202603/pull/10
    - https://github.com/h-masder/Arai60/pull/22
        - 幅優先探索の再帰は書いたことないな．と思った．ただ，queueでやっていることをlayer単位で次の再帰に渡すだけぽい．
    - https://github.com/rimokem/arai60/pull/21
        - `幅優先探索はmax_depthを保持しなくても` というのはDFSでもできるはずだな
        - こっちはqueueを利用した，BFSのパターン
- 自分の2回目のコード，LeftとRightのnilチェックなくしても動くということがわかった

# 3回目
```go
func maxDepth(root *TreeNode) int {
    if node == nil {
        return 0
    }

    return max(maxDepth(node.Left), maxDepth(node.Right)) + 1
}
```
- 元の関数とシグネチャが一緒なので，実はヘルパは不要だった．
- めちゃめちゃ短くなった
- これを3回繰り返した
