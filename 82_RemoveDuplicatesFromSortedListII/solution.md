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

P.S.
- `prev` は `previous_node`, `cur` は `current_node` のほうがよかったかも
    - https://github.com/hemispherium/LeetCode_Arai60/pull/10#discussion_r2618518592

# 4回目（[コメント](https://github.com/miyataka/coding-practice/pull/4#discussion_r3004652135)をうけて）

```go
// dummy(sentinel) を使わないパターン
// 1. 先頭だけ処理+ループ
func deleteDuplicates(head *ListNode) *ListNode {
    // 先頭が重複している間スキップ
    for head != nil && head.Next != nil && head.Val == head.Next.Val {
        v := head.Val
        for head != nil && head.Val == v {
            head = head.Next
        }
    }
    if head == nil || head.Next == nil {
        return head
    }
    // ここから先は head が確定。prev で追跡
    prev := head
    cur := head.Next
    for cur != nil {
        if cur.Next != nil && cur.Val == cur.Next.Val {
            v := cur.Val
            for cur != nil && cur.Val == v {
                cur = cur.Next
            }
            prev.Next = cur
        } else {
            prev = cur
            cur = cur.Next
        }
    }
    return head
}

// 2. 再帰
func deleteDuplicates(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }
    if head.Val == head.Next.Val {
        // 同じ値を全部スキップ
        for head.Next != nil && head.Val == head.Next.Val {
            head = head.Next
        }
        return deleteDuplicates(head.Next) // この値のノードを全部捨てる
    }
    head.Next = deleteDuplicates(head.Next)
    return head
}
```
- 以下のコメントを受けて，LLMの協力を得て別パターンを探した．
    - https://github.com/miyataka/coding-practice/pull/4#discussion_r3004652135
-
考え方としては，
    -
前者は，「最初に先頭の重複したnodeを削除し，その後先頭以外を処理します．先頭だけが，一つ前のnodeと比較することができないため，特別扱いする必要がある」だと思う．
    - 後者は「受け取ったリストを，先頭とその次のノードの値が一致するかをチェックし，同じ値であれば，同じ値が続く限りスキップする．残りの部分リストを再度同じ処理をする．先頭と次のノードが別の値である場合，一つノードを進めた部分リストについて同じ処理を呼び出す」 だろうか...
- 再帰のほうはまだ考え方が全然しっくりこない
