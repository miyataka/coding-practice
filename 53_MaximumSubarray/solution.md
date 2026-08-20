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

# 1回目 v3 (LLMからの回答）
```go
// exclusiveな累積和 `P` を考える．`P[0] = 0, P[i] = nums[0]+...+nums[i-1]`,
// 求めたいのは，max(P[r] - P[l]) (l<r). 右端rが固定値の時，P[l]はそれまでで最小になればよい
// なので，累積和から，それまでの最小累積和の値を引いてやれば，そのときのsubarrayの最大値をとれる
// 最大値を求めながらnumsを一周すればOK
func maxSubArray(nums []int) int {
    maxSum := nums[0]
    prefixSum := 0
    minPrefixSum := 0
    for _, n := range nums {
        prefixSum += n
        maxSum = max(maxSum, prefixSum-minPrefixSum)
        minPrefixSum = min(minPrefixSum, prefixSum)
    }
    return maxSum
}
```
- わからんかったのでLLMに確認した
- v1を書いたあとに最小値を求める計算を何度もやる必要がないことに気づければよかった

# 2回目
```go
func maxSubArray(nums []int) int {
    maxVal := nums[0]
    prefixSum := 0
    minPrefixSum := 0
    for _, n := range nums {
        prefixSum += n
        maxVal = max(maxVal, prefixSum-minPrefixSum)
        minPrefixSum = min(minPrefixSum, prefixSum)
    }
    return maxVal
}
```
- maxSumをmaxValにrenameしたくらい
- 他の人のコードを読む
- https://github.com/MA-yo-TA/leetcode/pull/31
    - https://github.com/MA-yo-TA/leetcode/pull/31/changes#r3823308446
    - > 二つのloopは一つにすることができそうです
        prefix_sum[i] は足し合わせていけばタイムリーにとれる
- https://github.com/chryschron/codings/pull/30
    - この人のコードどうこうではなく，DPに関して全然自分の頭が馴染んでいないのだなという感覚がする
- https://github.com/skypenguins/coding-practice/pull/34
    - kadane法, まだ頭に馴染んでない
- https://github.com/rimokem/arai60/pull/32
- https://github.com/tom4649/Coding/pull/130
    - 自分と解き方が違いすぎて，さっと読めない．読める幅を広げないとなぁ
- https://github.com/h-masder/Arai60/pull/35

# 3回目
```go
func maxSubArray(nums []int) int {
    maxVal := nums[0]
    prefixSum := 0
    minPrefixSum := 0
    for _, n := range nums {
        prefixSum += n
        maxVal = max(maxVal, prefixSum - minPrefixSum)
        minPrefixSum = min(minPrefixSum, prefixSum)
    }
    return maxVal
}
```
- これを3回繰り返した
- 正直DPは全然理解が追いついていないがある程度回数をこなすしかないのだろうと思って先に進む
