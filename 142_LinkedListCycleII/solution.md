# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。
---

# 1回目
```go
func detectCycle(head *ListNode) *ListNode {
    seen := map[*ListNode]bool{}
    node := head
    for node != nil {
        if seen[node] {
            return node
        }
        seen[node] = true
        node = node.Next
    }
    return nil
}
```
- 検出した瞬間に終わればいいはずなので，とりあえず解くという意味では，`141.Linked List Cycle` と同じでよいのではと考えた


# 2回目
```go
func detectCycle(head *ListNode) *ListNode {
    seen := map[*ListNode]*ListNode
    node := head
    for node != nil {
        if v, ok := seen[node]; ok {
            return v
        }
        seen[node] = node
        node = node.Next
    }
    return nil
}
```
- mapのvalueを*ListNodeに変えてみたが,,,別に綺麗になった気もしない．


# 3回目
```go
func detectCycle(head *ListNode) *ListNode {
    seen := map[*ListNode]bool{}
    node := head
    for node != nil {
        if seen[node] {
            return node
        }
        seen[node] = true
        node = node.Next
    }
    return nil
}
```
- 個人的にはこちらのほうが好み．
- Goのイディオムとして，`map[XXX]bool`を使うというのはよくあることだから．
- といっても，1回目の書き方にもどしただけ．
