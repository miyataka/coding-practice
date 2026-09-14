# 問題
https://leetcode.com/problems/house-robber-ii/

198 と同じ設定の泥棒だが、家が**円環状に並んでいる**。つまり**先頭の家と末尾の家も隣接している**。隣接する 2 軒を同時には選べないという条件のもとで、奪える**金額の最大値**を int で返す。

- 例1: nums = [2,3,2] → 3（1軒目と3軒目は隣接しているので両取りできない。3軒目単独でもなく、2軒目の 3 が最大）
- 例2: nums = [1,2,3,1] → 4（1軒目の 1 と 3軒目の 3）
- 例3: nums = [1,2,3] → 3
- 制約: 1 <= len(nums) <= 100、0 <= nums[i] <= 1000

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。

# 1回目（Not Passed）
```go
// 方針: house robberが直線に並んだhouseのrobbed or notのDPだった．
// 今回は，最初のhouseの結果を，本来の末尾のさらにあとに，付与しておくことで解決したりするだろうか？

// 最初に，先頭(first)のmoneyと末尾(last)のmoneyを比較して，二者択一にする．
// やりかたとしては，一番最初にfirstとlastのmoneyの大きなほうを取得することにして，robbedに保存する．取らなかったほうは0で上書きする
func rob(nums []int) int {
    var robbed, notRobbed int
    if len(nums) == 0 {
        return 0
    }
    if len(nums) == 1 {
        return nums[0]
    }

    switch {
        case nums[0] >= nums[len(nums)-1]:
            nums[len(nums)-1] = 0
        default:
            nums[0] = 0
    }

    for i := 0; i < len(nums); i++ {
        robbed, notRobbed = notRobbed+nums[i], max(robbed, notRobbed)
    }
    return max(robbed, notRobbed)
}
```

# 1回目
```go
func rob(nums []int) int {
    // 先頭を取らないと決めたverと
    // 末尾を取らないと決めたverでそれぞれ最大値を見つけ，それらの大きいほう（max)をとる
    if len(nums) == 0 {
        return 0
    }
    if len(nums) == 1 {
        return nums[0]
    }

    withoutFirst := robLinear(nums[1:])
    withoutLast := robLinear(nums[:len(nums)-1])
    return max(withoutFirst, withoutLast)
}

func robLinear(nums []int) int {
    var robbed, notRobbed int
    for _, n := range nums {
        robbed, notRobbed = notRobbed+n, max(robbed, notRobbed)
    }
    return max(robbed, notRobbed)
}
```

- セルフレビュー
    - robbed, notRobbedという命名を改善したい，くらいかな
- 見積もりを忘れていた
- input: 1 <= N <= 100
- 時間: ループを2回行うので，O(2N) = O(N). Goの秒間ステップ数を10^8回とすると，100 / 10^8 = 10^-6 = 1us
- 空間: ローカル変数を使い回すので定数値 O(1). inputのほかにはintの変数が2つ．16byte程度．無視できる

# 2回目

```go
func rob(nums []int) int {
    if len(nums) == 0 {
        return 0
    }
    if len(nums) == 1 {
        return nums[0]
    }

    withoutFirst := robLinear(nums[1:])
    withoutLast := robLinear(nums[:len(nums)-1])
    return max(withoutFirst, withoutLast)
}

func robLinear(nums []int) int {
    var takeMax, skipMax int
    for _, n := range nums {
        takeMax, skipMax = skipMax+n, max(takeMax, skipMax)
    }
    return max(takeMax, skipMax)
}
```
- 他の人のコードをみる
- https://github.com/Hiroto-Iizuka/coding_practice/pull/36
    - max_lootがその時点までの全部の結果をもっているのがすこし冗長に感じた．
- https://github.com/MA-yo-TA/leetcode/pull/35
    - step1, なぜこれで正答なのかちょっとわからなかった．with_beginのときに，末尾をとれる場合ととれない場合の分岐はどこでなされているのだろう
    - ああ，これ最後のmaxで-2のindexを使っている．つまり，末尾は絶対使えないので，それを入れないところまでの計算値までを使っているからだ
- https://github.com/chryschron/codings/pull/34
    - コード，めちゃくちゃわかりやすい
    - ただし着想がスムーズすぎて追いつけない.のと，最初をとるとらないに加えて，末尾をどこまで考えるか，の考慮についてはもうすこし説明があってよかったかも．くらいに思った
- https://github.com/skypenguins/coding-practice/pull/48
    - rob_linearという命名がわかりづらいとの指摘があった．うーむ，そうかぁ．．俺も見直すか．
    - step1のこのくらいの長さの関数内関数なら，別のメソッドとして分けて欲しい気もした
- https://github.com/rimokem/arai60/pull/36
    - 考え方は自分と同じだったので素直に読めた．
    - ただし，このくらいの長さの関数内関数なら，別のメソッドとして分けて欲しい気もした．もうちょっと言葉にするなら，関数のなかに別関数がそれなりの長さである場合，主要なフローだけ把握したい場合には関数名だけ把握すれば読み飛ばせる可能性があるもの（関数内関数）を無理やり読ませられている気になるから？
- https://github.com/h-masder/Arai60/pull/39
    - メモリアクセスを数十ステップとして見積もっているのが，なるべく厳密さを追求しているようで好ましく感じる
    - tom4649さんのコメントに同意で，ループは分けてしまった方が理解はしやすく思う
    - 関数内関数の定義からは一行空けた方が読みやすい
- https://github.com/colorbox/leetcode/pull/49
    - 変数名が長いなと一瞬感じる．
    - sizeが1と2のときは考慮されているのに，0のときは考慮されていないのはすこし不思議に感じる．leetcodeの制約上はいいんだけど
    - step3, vectorのlengthをどの程度と見積もるかによるが，性能の改善目的だとするとやらなくてもいい程度だなという
    - c++のコードはあまり詳しくないので，上記の点についてclaudeに読ませてみた．非破壊的だという点を強調する意図だとしたらconstをつけるべきとのこと．なるほどね．
- 典型コメント集
    - https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0#heading=h.8gnsrornagjc

# 3回目
```go
func rob(nums []int) int {
    if len(nums) == 0 {
        return 0
    }
    if len(nums) == 1 {
        return nums[0]
    }

    withoutFirst := robLinear(nums[1:])
    withoutLast := robLinear(nums[:len(nums)-1])
    return max(withoutFirst, withoutLast)
}

func robLinear(nums []int) int {
    var takeMax, skipMax int
    for _, n := range nums {
        takeMax, skipMax = skipMax+n, max(takeMax, skipMax)
    }
    return max(takeMax, skipMax)
}
```
