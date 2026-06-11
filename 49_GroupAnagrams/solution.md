# 問題
https://leetcode.com/problems/group-anagrams

文字列配列 `strs` が与えられる。アナグラム（同じ文字を並べ替えてできる文字列）どうしをグループ化して返す。順序は問わない。

- 例: `["eat","tea","tan","ate","nat","bat"]` → `[["bat"],["nat","tan"],["ate","eat","tea"]]`
- 制約: `1 <= strs.length <= 10^4`、`0 <= strs[i].length <= 100`、`strs[i]` は小文字のみ

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。
---

# 1回目

```go
// 方針: 文字列を文字とそのカウントのmapとして変換する．そのmapをkeyとしてarrayをvalueとしたmapを作る

// 方針2: 文字列をソートしたものをkeyとし，valueを対象となるarrayとして保持する

import "strings"

func groupAnagrams(strs []string) [][]string {
    sortedString2array := map[string][]string{}
    for _, str := range strs {
        keys := strings.Split(str, "")
        sort.Strings(keys)
        anagramKey := strings.Join(keys, "")
        sortedString2array[anagramKey] = append(sortedString2array[anagramKey], str)
    }

    result := [][]string{}
    for _, v := range sortedString2array {
        result = append(result, v)
    }
    return result
}
```
- 最初に方針1を考えたが，mapをmapのkeyにはできないと思ったので，方針2に切り替えた

# 2回目
```go
import "strings"

func groupAnagrams(strs []string) [][]string {
    sortedString2anagrams := map[string][]string{}
    for _, str := range strs {
        keys := strings.Split(str, "")
        sort.Strings(keys)
        anagramKey := strings.Join(keys, "")
        sortedString2anagrams[anagramKey] = append(sortedString2anagrams[anagramKey], str)
    }

    result := [][]string{}
    for _, v := range sortedString2anagrams {
        result = append(result, v)
    }
    return result
}
```
- 方針を素直にコードに落とし込めてはいると思ったので，mapのrenameだけ行った `sortedString2array` -> `sortedString2anagrams`


# 3回目
```go
import "strings"
import "sort"

func groupAnagrams(strs []string) [][]string {
    sortedKey2anagrams := map[string][]string{}

    for _, str := range strs {
        ss := strings.Split(str, "")
        sort.Strings(ss)
        anagramKey := strings.Join(ss, "")
        sortedKey2anagrams[anagramKey] = append(sortedKey2anagrams[anagramKey], str)
    }

    result := [][]string{}
    for _, v := range sortedKey2anagrams {
        result = append(result, v)
    }
    return result
}
```
- これを3回繰り返した
