# 問題
https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/

二分木の preorder（行きがけ順: 自分→左→右）と inorder（通りがけ順: 左→自分→右）の走査結果の配列を受け取り、その二分木を復元して root（*TreeNode）を返す。

- 例1: preorder = [3,9,20,15,7], inorder = [9,3,15,20,7] → [3,9,20,null,null,15,7]
- 例2: preorder = [-1], inorder = [-1] → [-1]
- 制約: 1 <= preorder.length <= 3000、inorder.length == preorder.length、-3000 <= Node.val <= 3000
- preorder と inorder は**値がすべて相異なり**、同一の木の正しい走査結果であることが保証されている

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
// pre-orderとin-orderのarrayが渡されて，それをbinary-treeに構築しなおしてreturnする
// あまり思いつかないので，条件の確認から．
// pre-orderは通った順．なのでrootが先頭にくることは保証される．
// in-orderはleftからrightに渡るときにrootを消費するパターン
// それぞれのarrayに含まれる数値がuniqueな値であることから
// これはpre-orderのarrayをtreeに組み立て直すときに，どのnode/leafをnullにすべきかというヒントをin-order arrayから判断せよ，というふうに解釈できそうかな？

// 方針:
// pre-orderの先頭がrootになることは保証されるのと，arrayに含まれる数字がuniqueであることを利用する
// 手順としては，pre-orderから，先頭をとり，その値のin-orderにおける位置を取得する．
// in-orderにおける位置を取得したらその値を境に，値を含まない形で左部分木と右部分木に分ける.
// pre-orderのarrayを，in-order左部分木の長さ分，n=1から始める形で切り取る，ここで初めに戻って再帰する．
// ちなみにpre-orderの切り取った残りは右部分木と一致するはずなので，こちらも再帰する

// 計算量が大丈夫かどうかをみておく
// 1 <= N <= 3000
// valはint64に十分収まる．
// nodeを構築するのが定数ステップだとすると，最悪時間計算量がO(N)
// Goの秒間ステップが10^8として，3*10^3 / 10^8 = 3 * 10^-5 = 30 us程度
// 空間計算量は，最終的に返すものが，Nに比例するので，O(N)なのと，再帰で，引数でpre-order, in-order, ローカル変数としてresultをそれぞれポインタで保持して，24B，
// それが最悪3*10^3回ネストするとする．
// nodeひとつあたりval, left, right. これは24B程度とする
// 24B * 3 * 10^3 * 2 = 144B * 10^3 = 144KB．Goのgoroutineスタックの上限は1GBなので，これは十分問題ない．
func buildTree(preorder []int, inorder []int) *TreeNode {
    if len(preorder) == 0 && len(inorder) == 0 {
        return nil
    }
    // 長さが一致しないときの考慮はいれたい

    root := preorder[0]
    rootIndex := 0
    for i, v := range inorder {
        if v == root {
            rootIndex = i
            break
        }
    }

    leftInorder := inorder[:rootIndex]
    rightInorder := inorder[rootIndex+1:]

    leftPreorder := preorder[1:len(leftInorder)+1]
    rightPreorder := preorder[len(leftInorder)+1:]

    return &TreeNode{
        Val: root,
        Left: buildTree(leftPreorder, leftInorder),
        Right: buildTree(rightPreorder, rightInorder),
    }
}
```
- 一発でpassはした
- 方針考えるのと，コードを書くのに45分くらいつかった
- セルフレビュー
    - コメントに書いたように，問題の前提に頼っている部分が多く，ちょっと怖いコード
        - array長さが一致することに頼っている
            - 長さ一致しないときは．．．このシグネチャでは無理がありそう．返り値でerrorも返すようにすべきかな
        - rootIndexが必ず見つかることを前提としている
            - rootIndexの初期値は`-1`にすべきだった．そしてindex探したあとにチェックすべき
    - 再帰でかけるということはiterativeにかけるはずだな．
        - iterativeにするには，書き込むべきアドレス（ポインタのポインタ）と，preorderのleft/right, inorderのleft/rightのindexをそれぞれ保持してスタックとして管理すればよさそうかな
        - まぁなんとなく再帰の方がコードはシンプルになりそうだが，スタックオーバーフローの懸念を排除できるのでやるか
    - これは深さ優先探索になっている気がする．そんでこれはボトムアップかな．
    - 幅優先探索っぽい考え方もできそうだが，全走査するのであまりうまみはなさそう．
    - index探すとき，slices.Find?とかが使えそう.
        - slices.Indexだった． https://pkg.go.dev/slices#Index

# 2回目
```go
func buildTree(preorder []int, inorder []int) *TreeNode {
    type nodeWithOrders struct {
        nodeAddress **TreeNode
        preorder []int
        inorder []int
    }

    result := &TreeNode{}
    pendingNodes := []nodeWithOrders{
        {nodeAddress: &result, preorder: preorder, inorder: inorder},
    }

    for len(pendingNodes) > 0 {
        n := pendingNodes[len(pendingNodes)-1]
        pendingNodes = pendingNodes[:len(pendingNodes)-1]
        nodeAddress := n.nodeAddress
        subPreorder := n.preorder
        subInorder := n.inorder

        if len(subPreorder) == 0 && len(subInorder) == 0 {
            continue
        }

        root := subPreorder[0]
        rootIndex := slices.Index(subInorder, root)
        node := &TreeNode{Val: root}
        *nodeAddress = node

        leftInorder := subInorder[:rootIndex]
        rightInorder := subInorder[rootIndex+1:]
        leftPreorder := subPreorder[1:len(leftInorder)+1]
        rightPreorder := subPreorder[len(leftInorder)+1:]

        pendingNodes = append(pendingNodes, nodeWithOrders{nodeAddress: &node.Left, preorder: leftPreorder, inorder: leftInorder})
        pendingNodes = append(pendingNodes, nodeWithOrders{nodeAddress: &node.Right, preorder: rightPreorder, inorder: rightInorder})
    }

    return result
}
```
- iterativeにした．スタックオーバーフローの懸念が薄いなら最初のバージョンが好みかも．
- ダブルポインタの操作は何回かミスった. まぁでも，left/rightを判別するフラグを渡すより好みなのでヨシとする
- 他の人のコードをみる
    - https://github.com/jjysogfy/arai60-202603/pull/18
        - 自分と同じ解法ぽい
        - rootValのindexが `numLeftNodes` というのはなるほど，たしかに！
        - javaにはsublistというメソッドがあるんだ．便利だな
            - https://docs.oracle.com/javase/jp/8/docs/api/java/util/List.html#subList-int-int-
        - あんま意識できてなかったけど，inorderのsliceからrootを探す処理の計算量はもうすこし考えるべきだったのか
        - valToInorderIndexなるほどなぁ，たしかにNがデカくなるとこれやりたくなりそう
        - `step 2 その2` のStackFrameという命名はもうちょい中身に言及した命名が嬉しいなぁ
        - indexを持ち回るより，listを持ち回ったほうが個人的に好みだな
    - https://github.com/chryschron/codings/pull/28
        - 時間計算量の検討がO(N^2)になっている．線形探索をN回やるから，たしかにそうか. わかってなかったな
        - この人のコードというわけでなく，今回も結構いろんな解法があって，理解が追いつかない部分が多い
    - https://github.com/skypenguins/coding-practice/pull/42
        - > この人のコードというわけでなく，今回も結構いろんな解法があって，理解が追いつかない部分が多い
    - https://github.com/nicah4o/arai60/pull/28
        - みんなわりと難しがっているな．
    - https://github.com/rimokem/arai60/pull/29
        - 両方のleftとsizeだけでいいのか，たしかにな
    - https://github.com/h-masder/Arai60/pull/32
        - みんなわりと難しがっているな．その2
        - 自分が45分かけたら解けたのは，たまたまで，持つべき疑念・懸念を持てていないだけで，たまたま問題の制約上Passしたロジックを作れただけという気がしてきた
    - https://github.com/Manato110/LeetCode-arai60/pull/29
        - みんなわりと難しがっているな．その3

# 3回目

## recursive DFS
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

func buildTree(preorder []int, inorder []int) *TreeNode {
    if len(preorder) == 0 && len(inorder) == 0 {
        return nil
    }

    rootVal := preorder[0]
    rootIndex := slices.Index(inorder, rootVal)

    leftInorder := inorder[:rootIndex]
    rightInorder := inorder[rootIndex+1:]

    leftPreorder := preorder[1:len(leftInorder)+1]
    rightPreorder := preorder[len(leftInorder)+1:]

    return &TreeNode{
        Val: rootVal,
        Left: buildTree(leftPreorder, leftInorder),
        Right: buildTree(rightPreorder, rightInorder),
    }
}
```

## iterative DFS
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

func buildTree(preorder []int, inorder []int) *TreeNode {
    type task struct {
        nodeAddress **TreeNode
        preorder []int
        inorder []int
    }

    result := &TreeNode{}
    tasks := []task{{nodeAddress: &result, preorder: preorder, inorder: inorder}}

    for len(tasks) > 0 {
        t := tasks[len(tasks)-1]
        tasks = tasks[:len(tasks)-1]
        nodeAddress, subPreorder, subInorder := t.nodeAddress, t.preorder, t.inorder

        if len(subPreorder) == 0 && len(subInorder) == 0 {
            continue
        }

        root := subPreorder[0]
        rootIndex := slices.Index(subInorder, root)
        node := &TreeNode{Val: root}
        *nodeAddress = node

        leftInorder := subInorder[:rootIndex]
        rightInorder := subInorder[rootIndex+1:]

        leftPreorder := subPreorder[1:1+len(leftInorder)]
        rightPreorder := subPreorder[1+len(leftInorder):]

        tasks = append(tasks, task{nodeAddress: &node.Left, preorder: leftPreorder, inorder: leftInorder})
        tasks = append(tasks, task{nodeAddress: &node.Right, preorder: rightPreorder, inorder: rightInorder})
    }
    return result
}
```
- これをそれぞれ3回繰り返した
