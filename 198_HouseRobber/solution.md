# 問題
https://leetcode.com/problems/house-robber/

通りに並んだ家を襲う泥棒。各家には金が置いてあり、`nums[i]` がその額。ただし**隣り合う家を同じ夜に2軒とも襲うと警報が鳴る**ので、隣接する2軒を同時には選べない。警報を鳴らさずに奪える**金額の最大値**を int で返す。

- 例1: nums = [1,2,3,1] → 4（1軒目の 1 と 3軒目の 3）
- 例2: nums = [2,7,9,3,1] → 12（1軒目の 2、3軒目の 9、5軒目の 1）
- 制約: 1 <= len(nums) <= 100、0 <= nums[i] <= 400

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。

# 1回目 (以下はfailするコード)
```go
// 全ての数字に対して，とる・とらないの選択肢がある．なおかつ「隣り合わないようにする」という条件を満たす組み合わせは，
// 組み合わせは，O(N^2)から，隣り合う場合を引いたとき．厳密にはわからんが，半分くらいになるのだろうか？
// いずれにしても，N^2オーダーになることは間違いなさそう．1<=N<=100なので，これは避けたい．

// 先頭から順に，みていく．
// i番目をとき，i番目をとるとしたときのベストと，i番目をとらないときのベストを両方保持しておく
// 最後の要素までみたときに，大きい方を返せばよさそう
func rob(nums []int) int {
    var withLast, withoutLast int
    for _, n := range nums {
        withLast += n
        withLast, withoutLast = withoutLast, withLast
    }
    return max(withLast, withoutLast)
}
```
- 2個飛ばしでバグってたけど，どう書き直せばいいのかわからなかった


# 1回目 (LLMからのアドバイス）
```go
// robbedHere:  この家を襲ったときの、ここまでの最大額
// skippedHere: この家を見送ったときの、ここまでの最大額
func rob(nums []int) int {
	robbedHere, skippedHere := 0, 0
	for _, money := range nums {
		robbedHere, skippedHere = skippedHere+money, max(robbedHere, skippedHere)
	}
	return max(robbedHere, skippedHere)
}
```
- LLMに回答をもらった
    - 回答ついでに，コメントで書いたO(N^2)が2^Nの誤記であると指摘された
- xxxHereは個人的には冗長な命名な気がした
- ループの中の同時代入は，一時変数を削減するためという理由があるが，ループの外の最初の初期化で，わざわざ同時代入を行う理由はないなと感じるので，単に宣言するだけにする



# 2回目
```go
func rob(nums []int) int {
    var robbed, skipped int
    for _, n := range nums {
        robbed, skipped = skipped+n, max(robbed, skipped)
    }
    return max(robbed, skipped)
}
```
- これはこれで満足
- 他の人のコードを読む
    - https://github.com/h-masder/Arai60/pull/38/changes
    - https://github.com/rimokem/arai60/pull/35/changes
    - https://github.com/skypenguins/coding-practice/pull/45/changes
    - https://github.com/chryschron/codings/pull/33/changes
    - https://github.com/MA-yo-TA/leetcode/pull/34/changes
- みた感想
    - 自分の発想の幅がまだ狭い
    - 再帰という選択肢は思い浮かんでいなかった

# 3回目
```go
func rob(nums []int) int {
    var robbed, skipped int
    for _, n := range nums {
        robbed, skipped = skipped+n, max(robbed, skipped)
    }
    return max(robbed, skipped)
}
```
- これを3回繰り返した
