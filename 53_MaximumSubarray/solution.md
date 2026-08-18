# 問題
https://leetcode.com/problems/maximum-subarray/

整数配列 nums を受け取り、和が最大になる**部分配列**（subarray）を見つけて、その**和**を int で返す。部分配列は元の配列から**連続した**要素を取り出した列のこと（300 の部分列 subsequence と違い、飛ばして取ることはできない）。空の部分配列は選べず、必ず1要素以上を含む。

- 例1: nums = [-2,1,-3,4,-1,2,1,-5,4] → 6（[4,-1,2,1] の和）
- 例2: nums = [1] → 1
- 例3: nums = [5,4,-1,7,8] → 23（全体の和）
- 制約: 1 <= nums.length <= 10^5、-10^4 <= nums[i] <= 10^4
- Follow up: O(n) の解法が分かったら、分割統治（divide and conquer）でも書いてみよう。そちらのほうが巧妙（more subtle）とのこと。

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。

# 1回目

```go
// **以下はTLEするコード**

// subarrayは空ではない, 連続した部分

// 方針
// 累積和の配列(sums)を作る．
// 累積和の配列に対して，開始位置beginと終了位置endを探索するために二重ループをする．
// for end := begin; end < len(); end++ {}
// のようなループを回して，sums[end] - sums[begin] の最大値を探せばよい

// 時間の見積もり
// 1 <= N <= 10^5
// 二重ループだから，O(N^2), 10^10ステップかかる．Go言語の実行ステップは秒間10^8ステップとすると，10^2秒かかる．100秒かぁ．
// ダメでもいったん書いてみよう．そのあとで最適化しよう．

// 以下はTLE
func maxSubArray(nums []int) int {
    fromHeadSums := make([]int, len(nums))

    previous := 0
    for i, n := range nums {
        fromHeadSums[i] += previous + n
        previous = fromHeadSums[i]
    }

    maxVal := fromHeadSums[0] // TODO boundary-check
    for begin := range nums {
        // ここ最適化できそうな感じある
        for end := begin; end < len(nums); end++ {
            val := maxVal
            if begin == end {
                val = fromHeadSums[begin]
            } else {
                val = fromHeadSums[end] - fromHeadSums[begin]
            }
            maxVal = max(maxVal, val)
        }
    }
    return maxVal
}
```

# 1回目 v2

```go
// **以下はTLEするコード**

func maxSubArray(nums []int) int {
    fromHeadSums := make([]int, len(nums))

    previous := 0
    for i, n := range nums {
        fromHeadSums[i] += previous + n
        previous = fromHeadSums[i]
    }

    if len(nums) == 0 {
        return 0
    }
    maxVal := fromHeadSums[0]

    // begin-endの組みのsumをcacheすることを考える.
    cachedSums := make(map[string]int, len(nums)) // key format like "<end>-<begin>"
    for begin := range nums {
        for end := begin; end < len(nums); end++ {
            key := fmt.Sprintf("%d-%d", end, begin)
            if v, ok := cachedSums[key]; ok {
                maxVal = max(maxVal, v)
                continue
            }

            val := 0
            if begin == end {
                val = fromHeadSums[begin]
            } else {
                val = fromHeadSums[end] - fromHeadSums[begin]
            }
            maxVal = max(maxVal, val)
            cachedSums[key] = val
        }
    }
    return maxVal
}
```
