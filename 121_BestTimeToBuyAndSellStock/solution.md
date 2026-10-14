# 問題
https://leetcode.com/problems/best-time-to-buy-and-sell-stock/

`prices[i]` が i 日目の株価を表す配列 `prices` が与えられる。1日を選んで株を買い、その後の別の1日を選んで売ることで得られる**利益の最大値**を返す（売りは買いより後の日でなければならない）。利益が出せない場合は `0` を返す。

- 例1: prices = [7,1,5,3,6,4] → 5（2日目(価格1)で買い、5日目(価格6)で売る。6-1=5）
- 例2: prices = [7,6,4,3,1] → 0（取引しないのが最善で、利益は0）
- 制約: 1 <= prices.length <= 10^5、0 <= prices[i] <= 10^4

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。

# 1回目
```go
// profit（diff）が最大となる二つの日の組み合わせを探し，最大のprofitをreturnすればよい

// 全探索すると，O(N^2)になる．

// ------------|-------------
// 上記のようなモデルを考える.
// arrayを分ける縦棒の左からはminを，右からはmaxをとって，そのdiffをとればよい
// これを，メモ化というか，効率的にやるために，
// arrayを左から走査して，縦棒がiまで左に含むときの最小値のarrayを作る
// arrayを右から走査して，縦棒がiまで左に含むとき（i+1以降を右に含むとき）の最大値のarrayを作る
// 二つのarrayのi番目のdiffを走査して，最大となったdiff（profit）をreturnする

func maxProfit(prices []int) int {
    mins := make([]int, len(prices))
    leftMin := prices[0]
    for i, n := range prices {
        leftMin = min(leftMin, n)
        mins[i] = leftMin
    }

    maxs := make([]int, len(prices))
    rightMax := 0
    for i := len(prices)-1; 0 <= i; i-- {
        rightMax = max(rightMax, prices[i])
        maxs[i] = rightMax
    }

    // diffのmaxを探す
    maxDiff := 0
    for i := 0; i < len(prices); i++ {
        if i+1 == len(prices) {
            break;
        }
        maxDiff = max(maxDiff, maxs[i+1] - mins[i])
    }

    return maxDiff
}
```

- 何度かfailed.
- `leftMin`の最初の初期化を0でやってしまい，それがminを記録するループでまったく上書きされないのが原因だった．
    - printデバッグをして解消
- 見積をできていないのでやっておく
    - inputは以下の制約となっている．
        - 1 <= prices.length <= 10^5
        - 0 <= prices[i] <= 10^4
    - 時間計算量はO(N)．pricesを操作するループを３回やっているので，3Nステップあるとする．Goの秒間実行ステップ数は10^8ステップとすると．
    - 3 * 10^5 / 10^8 = 3 * 10^-3 = 3ms程度
    - 空間計算量はN長のsliceが追加で2つだけ．なので，2N．intなので8byte程度として，最大2 * 8byte * 10^5 = 1.6mB程度
- セルフレビュー
    - len(prices) == 0 のチェックを最初にやったほうがよさそう
    - minを探すループとmaxを探すループのforの書き方を揃えた方が綺麗かも
    - 最後のdiffを探すループと，minを記録するループはまとめることができそう

# 2回目
```go
func maxProfit(prices []int) int {
    if len(prices) == 0 {
        return 0
    }
    maxs := make([]int, len(prices))
    rightMax := 0
    for i := len(prices)-1; 0 <= i; i-- {
        rightMax = max(rightMax, prices[i])
        maxs[i] = rightMax
    }

    // diffのmaxを探す
    maxDiff := 0
    leftMin := prices[0]
    for i := 0; i < len(prices); i++ {
        leftMin = min(leftMin, prices[i])
        if i+1 == len(prices) {
            break;
        }
        maxDiff = max(maxDiff, maxs[i+1] - leftMin)
    }

    return maxDiff
}
```
- コードが減ってすっきりした
- 空間計算量も半分になった
- 時間計算量も3Nだったのが，2Nに．
- 他の人のコードをみる
    - https://github.com/Hiroto-Iizuka/coding_practice/pull/37
        - `最安値とこれまでの最大利益を保存する方針` あ，これでいいな．たしかに．
    - https://github.com/MA-yo-TA/leetcode/pull/36
    - https://github.com/tsadamor/Leetcode/pull/8
    - https://github.com/chryschron/codings/pull/35
    - https://github.com/rimokem/arai60/pull/37
        - if と max/minの議論は良い議論だった．
        - 個人的にはこの問題を解く場合には可読性を優先したい. min/maxを選ぶ
    - https://github.com/h-masder/Arai60/pull/40
        - 自分も遠回りしてしまったので，すこし共感


# 3回目
```go
func maxProfit(prices []int) int {
    if len(prices) == 0 {
        return 0
    }

    minPrice := prices[0]
    maxProfit := 0
    for i := 0; i < len(prices); i++ {
        minPrice = min(minPrice, prices[i])
        maxProfit = max(maxProfit, prices[i] - minPrice)
    }
    return maxProfit
}
```

- これを3回繰り返した
