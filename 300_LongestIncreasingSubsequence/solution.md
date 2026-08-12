# 問題
https://leetcode.com/problems/longest-increasing-subsequence/

整数配列 nums を受け取り、**狭義単調増加**（strictly increasing）な部分列のうち最長のものの**長さ**を int で返す。部分列（subsequence）は、元の配列から要素を削除して（残った要素の順序は変えずに）作れる列のこと。連続している必要はない。

- 例1: nums = [10,9,2,5,3,7,101,18] → 4（[2,3,7,101]）
- 例2: nums = [0,1,0,3,2,3] → 4
- 例3: nums = [7,7,7,7,7,7,7] → 1（狭義増加なので同じ値は繋げない）
- 制約: 1 <= nums.length <= 2500、-10^4 <= nums[i] <= 10^4
- Follow up: O(n log n) のアルゴリズムを思いつけるか？

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。

# 1回目

```go
// O(N^2)の解法
func lengthOfLIS(nums []int) int {
    dp := make([]int, len(nums))
    best := 0
    for i := range nums {
        dp[i] = 1
        for j := 0; j < i; j++ {
            if nums[j] < nums[i] {
                dp[i] = max(dp[i], dp[j]+1)
            }
        }
        best = max(best, dp[i])
    }
    return best
}
```

```go
// O(N log N) の解法
import "sort"

func lengthOfLIS(nums []int) int {
    tails := []int{}
    for _, x := range nums {
        i := sort.Search(len(tails), func(i int) bool { return tails[i] >= x })
        if i == len(tails) {
            tails = append(tails, x)
        } else {
            tails[i] = x
        }
    }
    return len(tails)
}
```

- なにもわからないまま20分過ぎたので，回答をみた
- O(N^2)のやつの解釈をしていく
- 解釈メモ
    - 部分列を全列挙すると，各数字をとる/とらないの組み合わせ全体なので，2^Nとなる. この方針は結構厳しいことがわかる
    - DPは，やりかたを全探索から複数回参照される部分問題をmemoizationすることで効率化する手法
    - DPでは，なにが決まれば状態が確定するのか，をはっきりさせることが大事らしい
    - 今回の部分列は，「末尾の数字」がなにかによって，後ろに伸ばせるかどうかが決まる
    - 数列を前から走査しながら，
