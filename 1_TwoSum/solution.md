# 問題
https://leetcode.com/problems/two-sum

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。
---

# 1回目

```go
// 最初の方針．愚直解
// numsを二重にループを回して，足してtargetになったら終了．
// 外側のループと内側のループのインデックスが一緒ならskip
// 外側のループと内側のループのどちらかのvalueがtarget以上ならskip→これは無理だ．マイナスの数値もありうるから．
// また，最悪計算量が10^4^2 10^8

// なので，別の解法を考える．
// 最初にループで，hashに登録する．
// 別のループで，target - v = wantを計算し，これがhashに存在するかをチェックすればOK
// hashはO(1)なので，2*O(n)なので，O(n)

func twoSum(nums []int, target int) []int {
    val2index := map[int]int{}
    for i, v := range nums {
        val2index[v] = i
    }

    for i, v := range nums {
        want := target - v
        if wantIndex, ok := val2index[want]; ok {
            if i == wantIndex {
                continue
            }
            return []int{i, wantIndex}
        }
    }

    return []int{}
}
```
- 最初の愚直解のときはメモに記載しているのに， `if i == wantIndex` のskip条件を書き忘れ，一度だけfailした．
- これをミスらずに繰り返すようにする

# 2回目

```go
func twoSum(nums []int, target int) []int {
    val2index := map[int]int{}
    for i, v := range nums {
        val2index[v] = i
    }

    for i, v := range nums {
        want := target - v
        if wantIndex, ok := val2index[want]; ok && i != wantIndex {
            return []int{i, wantIndex}
        }
    }

    return []int{}
}
```
- 十分にシンプルだと思ったのでそのまま
