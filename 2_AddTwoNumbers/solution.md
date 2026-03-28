# 問題
https://leetcode.com/problems/add-two-numbers/description/

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。
---


# 1回目
```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
    // listの先頭が1の位から10, 100の位となっていく．
    // 足し算の繰り上がりの考慮が必要
    // 最初にからのsliceを作る

    // 両方がnilになるまでループする
    dummy := &ListNode{}
    sentinel := dummy

    var carry bool
    for l1 != nil || l2 != nil {
        node := &ListNode{}
        var v1, v2 int
        if l1 != nil {
            v1 = l1.Val
            l1 = l1.Next
        }
        if l2 != nil {
            v2 = l2.Val
            l2 = l2.Next
        }

        sum := v1+v2
        if carry {
            sum += 1
        }

        if sum >= 10 {
            node.Val = sum - 10
            carry = true
        } else {
            node.Val = sum
            carry = false
        }
        sentinel.Next = node
        sentinel = sentinel.Next
    }
    if carry {
        sentinel.Next = &ListNode{Val: 1}
    }
    return dummy.Next
}
```
- 時間は5分以上かけた．30分くらいかもしれないが，とりあえず思いついたやつでpass.
- 一度failはした．ループ終わったあとのcarryの考慮漏れにより．


# 2回目
```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
    // listの先頭が1の位から10, 100の位となっていく．
    // 足し算の繰り上がりの考慮が必要

    // 両方がnilになるまでループする
    dummyHead := &ListNode{}
    previousNode := dummyHead

    var carry int
    for l1 != nil || l2 != nil {
        sum := carry
        var v1, v2 int
        if l1 != nil {
            v1 = l1.Val
            l1 = l1.Next
        }
        if l2 != nil {
            v2 = l2.Val
            l2 = l2.Next
        }

        sum += v1+v2
        carry = sum / 10
        previousNode.Next = &ListNode{Val: sum % 10}
        previousNode = previousNode.Next
    }
    if carry > 0 {
        previousNode.Next = &ListNode{Val: carry}
    }
    return dummyHead.Next
}
```
- LLMからのフィードバックをうけ，carryをintで表すことができること，ループ内でのnode変数が不要なことなどを改善

# 3回目
```go
func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {
    dummyHead := &ListNode{}
    previousNode := dummyHead

    var carry int
    for l1 != nil || l2 != nil {
        sum := carry
        if l1 != nil {
            sum += l1.Val
            l1 = l1.Next
        }
        if l2 != nil {
            sum += l2.Val
            l2 = l2.Next
        }

        carry = sum / 10
        previousNode.Next = &ListNode{Val: sum % 10}
        previousNode = previousNode.Next
    }
    if carry > 0 {
        previousNode.Next = &ListNode{Val: carry}
    }
    return dummyHead.Next
}
```
- これを3回繰り返した
