# 問題
https://leetcode.com/problems/first-unique-character-in-a-string

文字列 `s` が与えられる。文字列中で最初に1回だけ出現する文字の**インデックス**を返す。存在しない場合は `-1` を返す。

- 例1: `s = "leetcode"` → `0`（`l` が最初の一意な文字）
- 例2: `s = "loveleetcode"` → `2`（`v`）
- 例3: `s = "aabb"` → `-1`
- 制約: `1 <= s.length <= 10^5`、`s` は小文字英字のみ

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。
---

# 見積もり

- 入力サイズ: 1 = len(s) ≤ 10^5
- 時間計算量: O(N)
    - 実行時間の見積もり: 10^5 / 10^8 = 10^-3 s ≈ 1ms
- 空間計算量: O(1)
    - 必要メモリの見積もり: <アルファベット小文字26文字> * <intを2つもつ構造体 16byte> = 416byte*
    - mapのオーバーヘッドを考慮して832byteといったところか，1kb未満.
    - あとはstructにすることで，アドレスを保持する部分があるけど，それも8byte*26=208byte程度．

# 1回目
```go
// 方針．characterをkeyとしたhashmapにいれて，countとindexを保持する．最後にcountが1のものを取り出し，その中での最小を返す．

type CountAndIndex struct {
    Count int
    Index int
}

func firstUniqChar(s string) int {
    charToCountAndIndex := map[rune]*CountAndIndex{}
    for i, r := range s {
        if cai, ok := charToCountAndIndex[r]; ok {
            cai.Count++
        } else {
            charToCountAndIndex[r] = &CountAndIndex{Count: 1, Index: i}
        }
    }

    result := -1
    for _, cai := range charToCountAndIndex {
        if cai.Count == 1 {
            if result == -1 {
                result = cai.Index
            } else if result > cai.Index {
                result = cai.Index
            }
        }
    }
    return result
}
```
- structでなくても，intが2つ保持できればいいだけなので，mapのentryを[]intにすれば，多少節約はできる
    - けどよほどの制約がない限りは今のままで十分
- 2回現れたときにはすでに対象外なので，保持する必要がなくなるので，実はカウントは保持する必要はないかもしれない．
- が，結局対象外になった，という事実を別の形で保持する必要はあるので，これもあまり良いアイデアではなさそう
-
ひとつのアイデアとして，出現箇所のindexだけ保持して，すでに複数回出現して対象外の場合は`-1`とするのはありかもしれない．がこれもまた可読性は落とす気がしてとりたくない．
- 他の人のPR
    - https://github.com/n6o/leetcode_arai60/pull/15/changes
    - https://github.com/Shunii85/arai60/pull/15/changes
    - https://github.com/h-masder/Arai60/pull/16/changes
- LLMに相談すると，
    - 全走査してcountを数え上げ，2回目の走査で最初にでてきたcount=1のものをreturnする，でよいとのこと
    - 全要素を再走査するのが気になったが，10^5程度であることと，mapだと，hash計算するのが重いので，速度を最適化するならアリみたい

# 2回目
```go
# 再走査ver
func firstUniqChar(s string) int {
    counts := map[rune]int{}
    for _, r := range s {
        counts[r]++
    }
    for i, r := range s {
        if counts[r] == 1 {
            return i
        }
    }
    return -1
}

# 再走査 & 英小文字の制約を利用するver
func firstUniqChar(s string) int {
    counts := [26]int{}
    for _, r := range s {
        counts[r-'a']++
    }
    for i, r := range s {
        if counts[r-'a'] == 1 {
            return i
        }
    }
    return -1
}
```
- 再走査verはコードがシンプルになるのでこれはこれでアリ．入力サイズがやはり判断のポイントにはなりそう．


# 3回目
```go
func firstUniqChar(s string) int {
    counts := map[rune]int{}
    for _, r := range s {
        counts[r]++
    }
    for i, r := range s {
        if counts[r] == 1 {
            return i
        }
    }
    return -1
}
```
