# 問題
https://leetcode.com/problems/valid-parentheses/description/

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。
---

# 1回目
```go
import "slices"

func isValid(s string) bool {
    // stackをつかう
    // stringをsliceとしてloop処理する
    // open braketがきたときはpushし，close braketがきたときはpopして，種別が一致することを確認する
    // 途中で一致しないケースがあればreturn falseして終了
    // 全部処理できればreturn true

    pairChars := map[rune]rune{
        ')': '(',
        ']': '[',
        '}': '{',
    }
    openChars := []rune{'(', '[', '{'}
    closeChars := []rune{')', ']', '}'}

    stack := []rune{}
    for r := range s {
        if slices.Contains(openChars, r) {
            stack.push(r)
            continue
        }
        if slices.Contains(closeChars, r) {
            top := stack.pop
            if pair := pairChars[r]; pair != top {
                return false
            }
        }
    }
    if len(stack) > 0 {
        return false
    }
    return true
}
```
- これはfailed.
- ~slices.Containsはruneに対応していないのでコンパイルエラーだった~
    - これは誤りだった．
    - https://pkg.go.dev/slices#Contains
    - https://cs.opensource.google/go/go/+/refs/tags/go1.26.1:src/slices/slices.go;l=117
    - 動作確認: https://go.dev/play/p/U2qffHSuyMN

# 2回目
```go
func isValid(s string) bool {
    // stackをつかう
    // stringをsliceとしてloop処理する
    // open braketがきたときはpushし，close braketがきたときはpopして，種別が一致することを確認する
    // 途中で一致しないケースがあればreturn falseして終了
    // 全部処理できればreturn true

    pairChars := map[rune]rune{
        ')': '(',
        ']': '[',
        '}': '{',
    }
    openChars := []rune{'(', '[', '{'}
    closeChars := []rune{')', ']', '}'}

    stack := []rune{}
    for _, r := range s {
        if contains(openChars, r) {
            stack = append(stack, r)
            continue
        }
        if contains(closeChars, r) {
            if len(stack) == 0 {
                return false
            }
            top := stack[len(stack)-1]
            stack = stack[:len(stack)-1]
            if pair := pairChars[r]; pair != top {
                return false
            }
        }
    }
    if len(stack) > 0 {
        return false
    }
    return true
}

func contains(rs []rune, r rune) bool {
    for _, rr := range rs {
        if rr == r {
            return true
        }
    }
    return false
}
```
- これはpassed.
- sliceの基本的な構文をミスしまっていたのでよくなかった．

# 3回目
```go
import "slices"

func isValid(s string) bool {
    // stackをつかう
    // stringをsliceとしてloop処理する
    // open braketがきたときはpushし，close braketがきたときはpopして，種別が一致することを確認する
    // 途中で一致しないケースがあればreturn falseして終了
    // 全部処理できればreturn true

    pairChars := map[rune]rune{
        ')': '(',
        ']': '[',
        '}': '{',
    }
    openChars := []rune{'(', '[', '{'}
    closeChars := []rune{')', ']', '}'}

    stack := []rune{}
    for _, r := range s {
        if slices.Contains(openChars, r) {
            stack = append(stack, r)
            continue
        }
        if slices.Contains(closeChars, r) {
            if len(stack) == 0 {
                return false
            }
            top := stack[len(stack)-1]
            stack = stack[:len(stack)-1]
            if pair := pairChars[r]; pair != top {
                return false
            }
        }
    }
    if len(stack) > 0 {
        return false
    }
    return true
}
```
- slices.Containsが使えたので，それに置き換え

# 4回目
```go
func isValid(s string) bool {
    pairChars := map[rune]rune{
        ')': '(',
        ']': '[',
        '}': '{',
    }

    stack := []rune{}
    for _, r := range s {
        if pair, isClose := pairChars[r]; isClose {
            if len(stack) == 0 {
                return false
            }
            top := stack[len(stack)-1]
            stack = stack[:len(stack)-1]
            if pair != top {
                return false
            }
        } else {
            stack = append(stack, r)
        }
    }

    return len(stack) == 0
}
```
- LLMからのレビューを受け，最後のif判定を簡略，pairCharsがあればopenChars, closeCharsが不要にできるとのことだったので，それに対応してみた
- 問題の制約上，これは成り立つとはいえ，元の判定条件のほうが素直な気がするので好みではない．

# 5回目
```go
import "slices"

func isValid(s string) bool {
    pairChars := map[rune]rune{
        ')': '(',
        ']': '[',
        '}': '{',
    }
    openChars := []rune{'(', '[', '{'}
    closeChars := []rune{')', ']', '}'}

    stack := []rune{}
    for _, r := range s {
        if slices.Contains(openChars, r) {
            stack = append(stack, r)
            continue
        }
        if slices.Contains(closeChars, r) {
            if len(stack) == 0 {
                return false
            }
            top := stack[len(stack)-1]
            stack = stack[:len(stack)-1]
            if pair := pairChars[r]; pair != top {
                return false
            }
        }
    }
    return len(stack) == 0
}
```
- これを3回繰り返した
