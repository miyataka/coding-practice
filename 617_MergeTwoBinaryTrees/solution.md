# 問題
https://leetcode.com/problems/merge-two-binary-trees/

2つの二分木を受け取り、重ねて1つにマージした木を返す。
同じ位置に両方のノードが存在する場合は値を足し合わせ、片方のみ存在する場合はそのノードをそのまま使う。

- 例1: root1 = [1,3,2,5], root2 = [2,1,3,null,4,null,7] → [3,4,5,5,4,null,7]
- 制約: 各木のノード数は [0, 2000]、-10^4 <= Node.val <= 10^4

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

 // 方針: treeを走査する処理なので，DFSか，BFSのどちらかを使うとよさそうだ．
 // よくわからないポイントとして，片方にしかないノードのとき，どのような処理にすればいいのだろうか？
 // 2つのルートを受け取る再帰的な処理にすれば，まぁいけそうかな？
 // スタックオーバーフローになるかの簡単な検算をしておく
 // N=2000 の配列が2つ受け取る．int32以上なら各Valは収まる．
 // まぁでも，TreeNodeのstruct見る感じ，(24+8)くらいで32byteとして，これを二つ受け取る感じの関数が再帰することになりそう
 // 最大2000回のスタックなので，2*10^3 * 32 = 64 * 10^3 = 64KB程度
 // Goのgoroutineのスタック上限である1GBに収まりそうなので，では気にせず書いていく
func mergeTrees(root1 *TreeNode, root2 *TreeNode) *TreeNode {
    switch {
        case root1 != nil && root2 != nil:
            nextLeft := mergeTrees(root1.Left, root2.Left)
            nextRight := mergeTrees(root1.Right, root2.Right)
            return &TreeNode{
                Val: root1.Val + root2.Val,
                Left: nextLeft,
                Right: nextRight,
            }
        case root1 != nil && root2 == nil:
            return &TreeNode{
                Val: root1.Val,
                Left: root1.Left,
                Right: root1.Right,
            }
        case root1 == nil && root2 != nil:
            return &TreeNode{
                Val: root2.Val,
                Left: root2.Left,
                Right: root2.Right,
            }
        default:
            return nil
    }
}

// 再帰でやること
// 2つのnodeと，resultのparentを受け取る．
// 片方がnilのとき，non-nilのLeft,Rightを繋げて終わり
// 両方がnilのとき，なにもしない（停止条件）
// 両方がnon-nilのとき，newNodeを作り，valを足して．．．？
// parent受け取るだけじゃ，leftなのかrightなのかわからないからそれも判別できるようにわたしたほうがいいのかな？
// えっと，これトップダウンで組み立てるか，ボトムアップなのかで違うかな？ボトルアップなら，left/rightは戻すときに判別すればよさそう
// func mergeTreeFunc(root1, root2 *TreeNode) *TreeNode {
//     switch {
//         case root1 != nil && root2 != nil:
//             nextLeft := mergeTreeFunc(root1.Left, root2.Left)
//             nextRight := mergeTreeFunc(root1.Right, root2.Right)
//             return &TreeNode{
//                 Val: root1.Val + root2.Val,
//                 Left: nextLeft,
//                 Right: nextRight,
//             }
//         case root1 != nil && root2 == nil:
//             return &TreeNode{
//                 Val: root1.Val,
//                 Left: root1.Left,
//                 Right: root1.Right,
//             }
//         case root1 == nil && root2 != nil:
//             return &TreeNode{
//                 Val: root2.Val,
//                 Left: root2.Left,
//                 Right: root2.Right,
//             }
//         default:
//             return nil
//     }
// }
```
- 2つめの関数は，一度書いたけど，全く同じシグネチャになったので不要だと気づいてコメントアウトした．
- あとこれは，もとのroot1,root2とアドレスを共有しているので破壊的になりうる
- 書き直すところは，あまり浮かばないが，わざわざnextLeftとかをnextとつける意味は薄いから削ってもいいかも．くらい
- 計算量をあらためて確認する
    - mergeTrees自体はO(1)
    - root1, root2が両方あるときだけ再帰がかかるので，2000回の再帰が上限
    - 時間計算量はO(N) 2000 * 10^-8 = 2*10^-5 = 0.02ms
    - 空間計算量もO(N) 32 * 2000 = 64 * 10^3 = 64KB

# 2回目
```go
func mergeTrees(root1 *TreeNode, root2 *TreeNode) *TreeNode {
    switch {
        case root1 != nil && root2 != nil:
            left := mergeTrees(root1.Left, root2.Left)
            right := mergeTrees(root1.Right, root2.Right)
            return &TreeNode{
                Val: root1.Val + root2.Val,
                Left: nextLeft,
                Right: nextRight,
            }
        case root1 != nil && root2 == nil:
            return root1
        case root1 == nil && root2 != nil:
            return root2
        default:
            return nil
    }
}
```
- 先に他の人のコードを読む
    - https://github.com/hiroki-horiguchi-dev/leetcode/pull/23
        - 非破壊的なほうがいいという点には同意なんだよな．Goではどうしよう．
    - https://github.com/jjysogfy/arai60-202603/pull/12
        - step1の20,21行目が改行入っている理由とかよくわからんかった．もう一度読んだらこれ関数呼び出しか．わかりづらくて好みではない．
    - https://github.com/nicah4o/arai60/pull/22
        - step1の35,36行目は`if (root1 && root2)` 内に入っているべきだなと感じた
    - https://github.com/attractal/leetcode/pull/32
        - 番兵はたしかにな．思いつかなかった.
    - https://github.com/hiro111208/leetcode/pull/17
        - pythonの三項演算子，rubyの `condition ? true-value : false-value` みたいな形式に慣れている身からすると読みにくかった
    - https://github.com/h-masder/Arai60/pull/26
        - Goで完全に非破壊的にする（inputとアドレスを共有しないようにする）のはどうしようかなと思った．
    - https://github.com/rimokem/arai60/pull/23
        - 自分もループverのDFSを書いてみないとな
    - https://github.com/Hiroto-Iizuka/coding_practice/pull/23
        - DFS（再帰）のコードを読んで，無駄なallocationをしていることに気づいた．
        - BFSの実装，理解するのにすこし時間がかかったが，これはTreeが配列で実装されていることに依存しすぎてやしないかと少し気になった
            - pythonのシグネチャがそうだからこれは考えすぎのようだ

# 2回目 非破壊
```go
func mergeTrees(root1 *TreeNode, root2 *TreeNode) *TreeNode {
    switch {
        case root1 != nil && root2 != nil:
            return &TreeNode{
                Val:   root1.Val + root2.Val,
                Left:  mergeTrees(root1.Left, root2.Left),
                Right: mergeTrees(root1.Right, root2.Right),
            }
        case root1 != nil:
            return &TreeNode{
                Val:   root1.Val,
                Left:  mergeTrees(root1.Left, nil),
                Right: mergeTrees(root1.Right, nil),
            }
        case root2 != nil:
            return &TreeNode{
                Val:   root2.Val,
                Left:  mergeTrees(nil, root2.Left),
                Right: mergeTrees(nil, root2.Right),
            }
        default:
            return nil
    }
}
```
- どうすんだ？と悩んでしまったのでLLMに聞いてみたら上記のコードを出力された
- こうすることで，全ノードが新規アロケーションされるようになり，非破壊になる
    - 非破壊にすることで，片方のnodeがnilでも関数呼び出しが起きるし，アロケーションされるようになったので，時間計算量，空間計算量どちらもO(N)のままだが，定数倍の悪化が起きる
    - なのでこれは結局は本番の使われ方による気がした．

# 2回目 非破壊/ループ
```go
func mergeTrees(root1 *TreeNode, root2 *TreeNode) *TreeNode {
    dummy := &TreeNode{}

    type task struct {
        n1, n2 *TreeNode
        parent *TreeNode
        left   bool
    }

    stack := []task{{root1, root2, dummy, true}}

    for len(stack) > 0 {
        t := stack[len(stack)-1]
        stack = stack[:len(stack)-1]

        if t.n1 == nil && t.n2 == nil {
            continue
        }

        val := 0
        var left1, right1, left2, right2 *TreeNode
        if t.n1 != nil {
            val += t.n1.Val
            left1, right1 = t.n1.Left, t.n1.Right
        }
        if t.n2 != nil {
            val += t.n2.Val
            left2, right2 = t.n2.Left, t.n2.Right
        }

        newNode := &TreeNode{Val: val}
        if t.left {
            t.parent.Left = newNode
        } else {
            t.parent.Right = newNode
        }

        stack = append(stack, task{left1, left2, newNode, true})
        stack = append(stack, task{right1, right2, newNode, false})
    }

    return dummy.Left
}
```
- これも一度自力で書いてみようとして，時間かかったので，LLMに聞いたもの．
    - 自力でやろうとしたとき，あれ，parentとかどうすんだっけ？という点は番兵とparent, left or notという情報を加えることで解決されている．やっぱその情報必要だよな，と思ったのと，parent一番最初に自力で解こうとしたときの方針を完走できればこれっぽくなったんだろうなという気持ち
- 自分の理解でかきくださう

# 3回目
```go
# 非破壊・再帰
func mergeTrees(root1, root2 *TreeNode) *TreeNode {
    switch {
        case root1 != nil && root2 != nil:
            return &TreeNode{
                Val: root1.Val + root2.Val,
                Left: mergeTrees(root1.Left, root2.Left),
                Right: mergeTrees(root1.Right, root2.Right),
            }
        case root1 != nil && root2 == nil:
            return &TreeNode{
                Val: root1.Val,
                Left: mergeTrees(root1.Left, nil),
                Right: mergeTrees(root1.Right, nil),
            }
        case root1 == nil && root2 != nil:
            return &TreeNode{
                Val: root2.Val,
                Left: mergeTrees(nil, root2.Left),
                Right: mergeTrees(nil, root2.Right),
            }
        // case root1 == nil && root2 == nil:
        default:
            return nil
    }
}

# 非破壊・ループ
func mergeTrees(root1, root2 *TreeNode) *TreeNode {
    dummy := &TreeNode{}

    type task struct {
        pair [2]*TreeNode
        parent *TreeNode
        isLeft bool
    }

    tasks := []task{
        {
            pair: [2]*TreeNode{root1, root2},
            parent: dummy,
            isLeft: true,
        },
    }

    for len(tasks) > 0 {
        t := tasks[len(tasks)-1]
        tasks = tasks[:len(tasks)-1]

        if t.pair[0] == nil && t.pair[1] == nil {
            continue
        }

        val := 0
        var left1, left2, right1, right2 *TreeNode

        if t.pair[0] != nil {
            val += t.pair[0].Val
            left1, right1 = t.pair[0].Left, t.pair[0].Right
        }
        if t.pair[1] != nil {
            val += t.pair[1].Val
            left2, right2 = t.pair[1].Left, t.pair[1].Right
        }

        newNode := &TreeNode{Val: val}
        if t.isLeft {
            t.parent.Left = newNode
        } else {
            t.parent.Right = newNode
        }

        tasks = append(tasks, task{pair: [2]*TreeNode{left1, left2}, parent: newNode, isLeft: true})
        tasks = append(tasks, task{pair: [2]*TreeNode{right1, right2}, parent: newNode, isLeft: false})
    }

    return dummy.Left
}
```
- それぞれ3回ずつ．再帰のほうがしっくりくる
