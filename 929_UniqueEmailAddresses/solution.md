# 問題
https://leetcode.com/problems/unique-email-addresses

メールアドレスは `@` で区切られた local name と domain name からなる。local name には次のルールが適用される:

- `.` は無視される（`alice.z@leetcode.com` と `alicez@leetcode.com` は同じ宛先）
- 最初の `+` 以降は無視される（`m.y+name@email.com` は `my@email.com` に届く）

domain name にはこれらのルールは適用されない。メールアドレスの配列 `emails` が与えられたとき、実際にメールを受け取る一意なアドレスの数を返す。

- 例1: `["test.email+alex@leetcode.com","test.e.mail+bob.cathy@leetcode.com","testemail+david@lee.tcode.com"]` → `2`
- 例2: `["a@leetcode.com","b@leetcode.com","c@leetcode.com"]` → `3`
- 制約: `1 <= emails.length <= 100`、`1 <= emails[i].length <= 100`、使用文字は小文字英字と `'+'` `'.'` `'@'` のみ、各 `emails[i]` はちょうど1つの `@` を含む、local name と domain name は空でない、local name は `+` で始まらない、domain name は `.com` で終わる

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。
---

# 1回目
```go
// 方針．invalidなものも含まれると考える．
// 各メールアドレスに対して，`@` で分割する．localname側のdotを除去＆`+`以降を削除する（`+`で分割し，leftのみ利用する）
// memo: 末尾の`.`の扱いが問題文から読み取れない．ドメインネームについても同様に末尾に`.`がつくようなドメインをinvalidとするかは読み取れない．
// 読み取れないものは一度無視して実装してみる

import "strings"

func numUniqueEmails(emails []string) int {
    uniqueEmails := map[string]bool{}
    for _, email := range emails {
        splited := strings.Split(email, "@")
        localname, domain := splited[0], splited[1]

        localname = strings.ReplaceAll(localname, ".", "")
        localname = strings.Split(localname, "+")[0]

        canonical := fmt.Sprintf("%s@%s", localname, domain)
        uniqueEmails[canonical] = true
    }

    return len(uniqueEmails)
}
```
- 単純だからか，わりと初回からわかりやすくかけたのではないか．
- 実行時間と時間計算量の見積もり
    - emailsの数がN，emailそれぞれの長さをMとすると，O(NM)
        - strings.ReplaceAllとstrings.Splitは文字列走査をするはずなので，O(M) と見積もった
    - 毎秒10^8ステップ程度の処理ができるとする. emails.length <= 100, emails[i].length <= 100となっているので，
    - (N * M) / 10^8 = 100 * 100 / 10^8 = 10^-4 = 0.1ms程度
- 必要メモリと空間計算量の見積もり:
    - N回のループの中で，都度変数を確保している分は定数倍
    - uniqueEmails は O(N)のサイズがあればよい
        - ひとつひとつのエントリはO(M)
    - よって全体としてはO(NM)
    - 1文字は1byteとする． emails.length <= 100, emails[i].length <= 100となっているので，
    - 100 * 100 * 1 = 10^4 byte = 10kbyte
    - hashtableはメモリを余分に必要とするので，倍として20KB程度
- reference
    - https://cs.opensource.google/go/go/+/refs/tags/go1.26.4:src/strings/strings.go;l=361
    - https://cs.opensource.google/go/go/+/refs/tags/go1.26.4:src/strings/strings.go;l=1180

# 2回目
```go
import "strings"

func numUniqueEmails(emails []string) int {
    uniqueEmails := map[string]bool{}
    for _, email := range emails {
        localname, domain, _ := strings.Cut(email, "@")

        localname = strings.ReplaceAll(localname, ".", "")
        localname, _, _ = strings.Cut(localname, "+")

        canonical := fmt.Sprintf("%s@%s", localname, domain)
        uniqueEmails[canonical] = true
    }

    return len(uniqueEmails)
}

```
- LLMより，`strings.Cut`
    という関数を教えてもらったのでこれのバージョンを書いてみたが，別にわかりやすくもないような...
    - https://pkg.go.dev/strings#Cut
- もっと実務的なvalidationが必要になる場合には `net/mail.ParseAddress` を利用するのがよさそう

# 3回目
```go
import "strings"
import "fmt"

func numUniqueEmails(emails []string) int {
    uniqueEmails := map[string]bool{}
    for _, email := range emails {
        splited := strings.Split(email, "@")
        localname, domain := splited[0], splited[1]

        localname = strings.ReplaceAll(localname, ".", "")
        localname = strings.Split(localname, "+")[0]

        canonical := fmt.Sprintf("%s@%s", localname, domain)
        uniqueEmails[canonical] = true
    }
    return len(uniqueEmails)
}
```
- 結局初回のコードを3回繰り返す
