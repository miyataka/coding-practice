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
- O(N^2)コード解釈メモ
  - ループでなにをしているか
  - inputをループしている．かつ二重ループとして，j<iとなる部分をループしている
    - iより小さいindex(j)のうち，その値がi番目より小さい場合（つまり`nums[j] < nums[i]`が成り立つ時）の最大のものに+1した値を`dp[i]`として記録する
    - このとき，`dp[i]`は「i番目を末尾としたときの狭義単調増加が成り立つ部分数列の最大長」になる
- メモ
    - 部分列を全列挙すると，各数字をとる/とらないの組み合わせ全体なので，2^Nとなる. この方針は結構厳しいことがわかる
    - DPは，やりかたを全探索から複数回参照される部分問題をmemoizationすることで効率化する手法
    - DPでは，なにが決まれば状態が確定するのか，をはっきりさせることが大事らしい．ピンとこなかったが，あとで戻ってくることにする
- とりあえずO(N^2)は3回書いてみた
- O(N log N)コード解釈メモ
    - tailsという変数を定義する
        - tails[k] = 長さ k+1 の増加部分列の，末尾として実現可能な最小値，らしい
        - 増加部分列としてその長さに至ったときの，最小値を集めたのが，tailsらしい．実際のLISになるとは限らない
    - sort.Searchという関数で二分探索をして，indexを特定し，そこを置き換える. indexがtailsの長さと一致したら，末尾にappendする. を繰り返す
    - 最後にlen(tails)を返す
- あまりにO(N log N)の解法がピンときていない，自分のモノになっていない気がするので，O(N^2)の解法でstep3まで通す
- セルフレビュー
    - 今回はLLMに回答を出力してもらったので，LLMのコードにケチをつける形になる．
    - dpという変数名はなんとかしたい
    - あとはDP系の問題には個人的にはコメントをいれるようにしようと思った．コードから，意図や考えていることを読み取るのが結構難しいように感じる

P.S.
- 実行時間とメモリの見積もり
    - input: 1 <= nums.length <= 2500
    - 二重ループの解法なので，O(N^2)
    - 2500*2500=625*10^4=6.3*10^6
    - Go言語で1秒間に10^8ステップ実行できるとすると，6.3*10^6 / 10^8 = 6.3 * 10^-2 = 0.063 s = 63ms程度．~100ms程度あたりか
    - メモリは，長さNのint配列があればよいので，O(N)，2500*8byte=20000B=20KB程度

# 2回目
```go
func lengthOfLIS(nums []int) int {
    longestLength := make([]int, len(nums))
    best := 0
    for i := range nums {
        longestLength[i] = 1
        for j := 0; j < i; j++ {
            if nums[j] < nums[i] {
                longestLength[i] = max(longestLength[i], longestLength[j]+1)
            }
        }
        best = max(best, longestLength[i])
    }
    return best
}
```
- 他の人のコードを読む
    - 見積もりをやっていないことに気づいた．あとで書く
    - https://github.com/jjysogfy/arai60-202603/pull/19
        - `ところで、どんな`nums`のせいでこういうアルゴリズムが必要になるのか、わかっていないと気づく`
        - これにはかなり共感した．問題があるので解いているけど，現実世界での使い所が全くピンときていない
    - https://github.com/MA-yo-TA/leetcode/pull/30
        - 典型コメント集を読んでないことに気づいたので読んだ．
        - python勢のbisect_leftの解法は，binarysearchの解法ということだろう
        - セグメントツリーの解法があるのか．
    - https://github.com/skypenguins/coding-practice/pull/50
        - この人も思いつかなかったようで，この問題は`常識を掃き切った`人は解ける類なのかなと思った．
    - https://github.com/chryschron/codings/pull/29
    - https://github.com/nicah4o/arai60/pull/29
        - ~この人のメモをみていたら．急に二分探索の方法の考え方がわかった気がする~ →書いてみたが勘違いがあった. `tails`の中身は最終的なLISと一致しないので自分の考え方では破綻する
    - https://github.com/rimokem/arai60/pull/31
        - > https://github.com/rimokem/arai60/pull/31/changes#diff-eff571cb186bef5632bec6a0a2f1c85fdda7ee181df4da02e1edcfa75e1a0058R37-R47
        - この部分，納得感あった
    - https://github.com/hiro111208/leetcode/pull/29
    - https://github.com/h-masder/Arai60/pull/34

# 3回目
```go
func lengthOfLIS(nums []int) int {
    longestLength := make([]int, len(nums))
    best := 0
    for i := range nums {
        longestLength[i] = 1
        for j := 0; j < i; j++ {
            if nums[j] < nums[i] {
                longestLength[i] = max(longestLength[i], longestLength[j]+1)
            }
        }
        best = max(best, longestLength[i])
    }
    return best
}
```
- これを3回繰り返した
- O(N log N)の解法がピンとこないままだったので，要復習．時間かけすぎなので，いったんここで終わり
