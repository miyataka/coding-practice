# 問題
https://leetcode.com/problems/subarray-sum-equals-k

整数配列 `nums` と整数 `k` が与えられる。和が `k` に等しい連続部分配列（subarray）の**個数**を返す。

- 例1: `nums = [1,1,1]`, `k = 2` → `2`
- 例2: `nums = [1,2,3]`, `k = 3` → `2`（`[1,2]` と `[3]`）
- 制約: `1 <= nums.length <= 2*10^4`、`-1000 <= nums[i] <= 1000`、`-10^7 <= k <= 10^7`

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。
---

# 最初の見積もり
- 入力サイズ: 1 <= nums.length <= 2*10^4
- 時間計算量: O(N^2)
  - 実行時間の見積もり: 4*10^8 / 10^8 ≈ 4s
- 空間計算量: O(1)

# 1回目
```go
// 方針: 特定の数字になるsubarrayを見つける必要がある．そのパターンを探す必要がある．
// subarrayは，隣接するnon-emptyなarrayのことらしい，
// 先頭から，一つずつ，subarrayの始点を移動させながら，探索する．kに一致する必要がある，かつ数字は負の値をとりうるので，
// 一つ見つけてもそこで停止せず，末尾まで確認する必要がある．
// 全探索すると，n+(n-1)+(n-2)+...の計算量になりそうだ．逆にいうと，1+2+...+nということだから，n(n+1)/2ということか．
// (n^2+n)/2 →O(n^2)
// 制約としては，2*10^4なので，4*10^8のスケール．コンパイル言語が10^8ステップ/毎秒とすると4秒になるのでタイムアウト

// とはいえ，思いつかないので，書いてからそれを最適化する方針とする
func subarraySum(nums []int, k int) int {
    count := 0
    for i := 0; i < len(nums); i++ {
        sum := 0
        for j := i; j < len(nums); j++ {
            sum += nums[j]
            if sum == k {
                count++
            }
        }
    }
    return count
}
```
- ギリ通った．まあいいや．最適化する
- 考える時間もオーバーしてしまったので答えをみる．

# 2回目
```go
// 解法を自分で再度テキストにおとす．
// 先頭からarrayを走査して，prefix sum hashmapを作る．
// そうすると，subarrayの始点(i)と終点(j)を指定することで，subarrayのsumが引き算で求めることができる
// e.g.) prefixSum[j] - prefixSum[i]
// subarraySum = prefixSum[j] - prefixSum[i] = k
// これを変形して
// prefixSum[j] - k = prefixSum[i]
// これは，現在までのsubarraySum[j] の値から，kを引いた数になるSumをみつければよいということ
// arrayのなかには，マイナスの値やゼロがあるので，subarray[i] と一致するときのインデックスではなく，出現回数を記録しておく必要がある

func subarraySum(nums []int, k int) int {
    sum := 0
    count := 0

    seen := map[int]int{0: 1}

    for _, n := range nums {
        sum += n
        if c, ok := seen[sum-k]; ok {
            count += c
        }
        seen[sum]++
    }
    return count
}
```
- これは実質1回目

# 3回目
```go
func subarraySum(nums []int, k int) int {
    prefixSum := 0
    count := 0

    prefixSumCount := map[int]int{0: 1}

    for _, n := range nums {
        prefixSum += n
        if c, ok := prefixSumCount[prefixSum-k]; ok {
            count += c
        }
        prefixSumCount[prefixSum]++
    }
    return count
}
```
- 命名を見直した
- 典型コメントの確認
    - https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.bp0g0ai41eln
- 他の人のPR
    - https://github.com/rimokem/arai60/pull/16/changes
    - https://github.com/h-masder/Arai60/pull/17/changes
    - https://github.com/Shunii85/arai60/pull/16/changes
    - https://github.com/n6o/leetcode_arai60/pull/16/changes

# 4回目
```go
func subarraySum(nums []int, k int) int {
    prefixSum := 0
    prefixSumCount := map[int]int{0: 1}
    count := 0

    for _, n := range nums {
        prefixSum += n
        if c, ok := prefixSumCount[prefixSum-k]; ok {
            count += c
        }
        prefixSumCount[prefixSum]++
    }
    return count
}
```
- これを4回繰り返した
- 時間計算量: O(N)
  - 実行時間の見積もり: 2 * 10^4 / 10^8 = 2 / 10^4 = 0.2ms
- 空間計算量: O(N)
  - 必要メモリの見積もり: N * (int key + int value) = 2 * 10^4 * 2byte = 40kb程度．マップが2倍の空間必要だとして，80kb程度だろうか
