# 問題
https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii/description/

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。
---

# 1回目

```go
func deleteDuplicates(head *ListNode) *ListNode {
    node := head
    // 前から順番に，次の値と比較して，一致した数値があったら，それをメモしておく．
    // 再度ループする．メモと一致したものが見つかったら，詰める．

    duplicatedNumbers := map[int]bool{}

    for node != nil && node.Next != nil {
        if node.Val == node.Next.Val {
            node.Next = node.Next.Next // 後ろを詰める
            duplicatedNumbers[node.Val] = true
        } else {
            node = node.Next
        }
    }

    // 先頭にdummy をつけたheadをループ処理
    node = &ListNode{Val: -1, Next: head}
    for node != nil && node.Next != nil {
        if duplicatedNumbers[node.Next.Val] {
            node.Next = node.Next.Next
        } else {
            node = node.Next
        }
    }

    return head
}
```
- これはfail
- 原因は，headを返してしまっていたこと


# 2回目

```go
func deleteDuplicates(head *ListNode) *ListNode {
    node := head
    // 前から順番に，次の値と比較して，一致した数値があったら，それをメモしておく．
    // 再度ループする．メモと一致したものが見つかったら，詰める．

    duplicatedNumbers := map[int]bool{}

    for node != nil && node.Next != nil {
        if node.Val == node.Next.Val {
            node.Next = node.Next.Next // 後ろを詰める
            duplicatedNumbers[node.Val] = true
        } else {
            node = node.Next
        }
    }

    // 先頭にdummy をつけたheadをループ処理
    dummy := &ListNode{Val: -1, Next: head}
    node = dummy
    for node != nil && node.Next != nil {
        if duplicatedNumbers[node.Next.Val] {
            node.Next = node.Next.Next
        } else {
            node = node.Next
        }
    }

    return dummy.Next
}
```
- これはpassed


# 3回目
```go
func deleteDuplicates(head *ListNode) *ListNode {
    dummy := &ListNode{Next: head}
    prev := dummy

    for prev.Next != nil {
        cur := prev.Next

        for cur.Next != nil && cur.Val == cur.Next.Val {
            cur = cur.Next
        }

        if prev.Next == cur {
            prev = prev.Next
        } else {
            prev.Next = cur.Next
        }
    }
    return dummy.Next
}
```
- 自力では全然すっきりした回答にならなかったので，回答を確認した
- dummyは壁に固定された杭，prevはチェック済みを示すしるしと想像するとなんとなくわかった気がする
- 最初にheadを壁に固定してやる（dummyにつなぐ），みたいな想像
- これを3回連続で書いた
