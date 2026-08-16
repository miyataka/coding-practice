# 問題
https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/

昇順にソートされた整数配列 `nums` を受け取り、それを平衡二分探索木（height-balanced BST）に変換して、そのrootを返す。
height-balancedとは、各ノードについて左右の部分木の高さの差が1以内であること。

- 例1: nums = [-10,-3,0,5,9] → [0,-3,9,-10,null,5] や [0,-10,5,null,-3,null,9] など（複数の正解がありうる）
- 例2: nums = [1,3] → [3,1] や [1,null,3] など
- 制約: 1 <= nums.length <= 10^4、-10^4 <= nums[i] <= 10^4、nums は厳密な昇順（重複なし）

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。

# 見積もり
- 入力サイズ: N = nums.length ≤ 10^4
- 時間計算量: O(N)
  - 実行時間の見積もり: 10^4 / 10^8 = 10^-4 s = 0.1ms
- 空間計算量: O(N)
  - 必要メモリの見積もり: 8byte * 10^4 = 80KB

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

 // 方針: これはなんの性質が使えるだろうか？
 // pre-sortedだからそれを利用するとよさそう．
 // 長さを半分にしながら，中央値をルートにしながら，再帰的に分割して，ボトムアップでツリーを構築する，とすればどうか？
 // nums.lengthが10^4なので，
 // 時間計算量は, 全部触るだろうからO(N)
 // 空間計算量は，再帰が最大logNだろうから O(logN)．これはunbalancedなときは考えなくていい.
 // Goが一秒あたり10^8ステップ実行できるとすると，
 // 実行時間が 10^-4 s = 0.1ms
 // 消費メモリが，最大int64を10^4だから，8 * 10^4 = 80KB

// 細かい手順を考えていく
// 受け取ったnumsの長さをとり，その半分のindexの数字をValにセットしてLeftとRightのアドレスを渡してあげればよいような．
// なので，受け取るのはnumsと書き込み先のアドレスかな.
// 中央値はもう消費しているので，中央値より小さい部分，中央値より大きい部分をそれぞれ渡して呼び出す必要がある
func sortedArrayToBST(nums []int) *TreeNode {
    dummy := &TreeNode{}

    type task struct {
        nums []int
        node **TreeNode
    }

    tasks := []task{
        {nums: nums, node: &dummy}, // ここにカンマ忘れ
    }

    for len(tasks) > 0 {
        t := tasks[len(tasks)-1]
        tasks = tasks[:len(tasks)-1]

        middleIndex := len(t.nums) / 2 // NOTE: ここで切り捨てられるのは問題ないよね？の検討をする

        // 停止条件
        if len(t.nums) <= 0 {
            continue
        }
        node := &TreeNode{Val: t.nums[middleIndex]}
        *(t.node) = node

        tasks = append(tasks, task{nums: t.nums[:middleIndex], node: &node.Left}) // ここを`t.nums`, ではなく`nums`にしてしまい，無限ループになった
        tasks = append(tasks, task{nums: t.nums[middleIndex+1:], node: &node.Right}) // ここを`t.nums`, ではなく`nums`にしてしまい，無限ループになった
    }
    return dummy
}
```
- なんどかfailさせた
    - カンマ忘れ
    - t.numsにすべきところをnumsにしており，無限ループ
- 他の選択肢に思いを馳せておく
    - DFSでできるということはBFSでもできるのだろう．
    - 今回はスタックオーバーフローを避けるために，ループで書いたが再帰でも書けるだろう．
    - 愚直解もあんまりわからんな．今回は．
    - うーん，現状だとこれくらいしかパッとはでてこないな．

- 制約がなくなったらと考えてみる
    - スタックオーバーフローは心配ないので，N自体はまぁ問題ないでしょう．
    - intじゃなくってstringだったら？structだったら？
        - stringかつasciiだったら数値的に比較すればいいか．UTF-8だったらコードポイントとかで比較するのかな？
        - structだったら，そのときの順序づけるルールに乗っ取ってpre-sortすればいいか
    - なんかあまり有用な想像じゃない気がしてきたので終わり
- 改善すべきポイントはどこだろう
    - middleIndexはなんか冗長な気がするから，medianとかもいいかも．でもvalueぽすぎる．indexであることは明示したいから，medianIndexとかか？結局あまり変わらない感じになったな
    - dummyじゃなくて，resultでもよかったかも
    - tasksじゃなかったらなにかいいのあるかな？
- 他の人のをみてみる
    - https://github.com/h-masder/Arai60/pull/27
        - 幅優先探索のほうが考えやすい人もいるんだなぁと思った．自分は深さ優先の方ばかり考えてしまいがち．
        - height-balancedになるのはなぜか？という点への自分の理解としては，「ソート済み配列を二分割していくから，片方に偏ってしまうことが起こらないから」というもの
    - https://github.com/hiro111208/leetcode/pull/18
        - あ，自分はNがゼロになるときのケアができていないな，と思った
            - その場合はnilを返すべきとしたら，自分のコードはできていない
        - step1はおおよそ自分と同じ考え方なので，すぐ理解できる
        - step2は再帰を使いつつ，numsは共有して利用している．メモリの効率化をしている．
        - BFSのイメージがやっとついた．ツリーのルートから，Left/Rightを埋めていくことを優先してやってくイメージ
    - https://github.com/dorxyxki/arai60/pull/24
        - step1は一つ前の問題で自分がやった解き方に似ている．parentとleft|rightを渡すやりかた．
        - discussionみてると，単調増加を利用して，left|rightを渡さなくてもよいというのはなるほどなぁと
            - とはいえ，同じ数値が含まれる，など少し緩和したときはどうなる？という気持ちがすこし
        - `再帰における帰りがけの接続をループで表現`のコード， `stack` に説明なしに5つの値を持つtupleが追加されて，読むのがつらい
        - pythonの人たち，Listを生成するのがもったいない感じをわりとみんな？心配してる．自分はあんまり心配できてなかったかも
    - https://github.com/attractal/leetcode/pull/36
    - https://github.com/nicah4o/arai60/pull/23
        - 自分は結果としてAVL木だな，という連想ができてなかったな，と思った
        - CPPで，sliceのコピーを避けるため，最初の配列の参照を引き回していた．
        - mid_indexという命名いいな．
    - https://github.com/hiroki-horiguchi-dev/leetcode/pull/24
        - 時間計算量はO(logN)にならないのではと思った．特定の値を探し当てたら終わりではなく，全部みるしかないので．

# 2回目
```go
func sortedArrayToBST(nums []int) *TreeNode {
    result := &TreeNode{}

    type task struct {
        nums []int
        node **TreeNode
    }
    tasks := []task{{nums: nums, node: &result}}

    for len(tasks) > 0 {
        t := tasks[len(tasks)-1]
        tasks = tasks[:len(tasks)-1]
        midIndex := len(t.nums) / 2

        if len(t.nums) <= 0 {
            continue
        }

        node := &TreeNode{Val: t.nums[midIndex]}
        *(t.node) = node
        tasks = append(tasks, task{nums: t.nums[:midIndex], node: &node.Left})
        tasks = append(tasks, task{nums: t.nums[midIndex+1:], node: &node.Right})
    }
    return result
}
```
- 他の人のも読んだ上で，ループでDFSをやっているこれが自分の頭にはしっくりくるなと思った．
- もっと省メモリにしたほうがよいということなら，taskにnumsを持たせないようにする選択肢がある

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
func sortedArrayToBST(nums []int) *TreeNode {
    result := &TreeNode{}

    type task struct {
        nums []int
        node **TreeNode
    }
    tasks := []task{{nums: nums, node: &result}}

    for len(tasks) > 0 {
        t := tasks[len(tasks)-1]
        tasks = tasks[:len(tasks)-1]

        if len(t.nums) <= 0 {
            continue
        }
        midIndex := len(t.nums) / 2
        node := &TreeNode{Val: t.nums[midIndex]}
        *t.node = node
        tasks = append(tasks, task{nums: t.nums[:midIndex], node: &node.Left})
        tasks = append(tasks, task{nums: t.nums[midIndex+1:], node: &node.Right})
    }
    return result
}
```
- これを3回繰り返した
