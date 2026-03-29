# 問題
https://leetcode.com/problems/reverse-linked-list/description/

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。
---

# 1回目
```go
func reverseList(head *ListNode) *ListNode {
    stack := []*ListNode{}

    for head != nil {
        stack = append(stack, head)
        head = head.Next
    }

    reversedDummyHead := &ListNode{}
    prev := reversedDummyHead
    for len(stack) > 0 {
        poped := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        poped.Next = nil

        prev.Next = poped
        prev = prev.Next
    }

    return reversedDummyHead.Next
}
```
- `poped.Next = nil` が抜けてて一度failed
- 考え方は「与えられたlistをstackにためておき，popしながら再構築する」というもの

# 2回目
```go
 // 先頭からループする
 // ひとつdummyノードを作っておき，
 // ループしているnodeをdummyNodeのNextにつなぐ．末尾のNextは断ち切っておく
 // 次のループで，dummyNode.Nextをnode.Nextにつなぎ，dummyNode.Next = nodeにする
 // ループする
func reverseList(head *ListNode) *ListNode {
    if head == nil {
        return nil
    }
    node := head.Next
    head.Next = nil
    reversedDummyHead := &ListNode{Next: head}

    for node != nil {
        nodeNext := node.Next
        node.Next = reversedDummyHead.Next
        reversedDummyHead.Next = node
        node = nodeNext
    }

    return reversedDummyHead.Next
}
```
- メモリをあまり食わない形で一度書き直した

# 3回目
```go
func reverseList(head *ListNode) *ListNode {
    var prev *ListNode
    for head != nil {
        next := head.Next
        head.Next = prev
        prev = head
        head = next
    }

    return prev
}
```
- `head == nil`要らなそうだなと思ってたらもっと簡素にできた
- これを3回繰り返した

# 4回目 (TBC)
```go
```
- follow-upの再帰verを書いてみようと思っているがとりあえずレビュー依頼する
