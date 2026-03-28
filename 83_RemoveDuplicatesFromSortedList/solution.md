# 1回目

```go
func deleteDuplicates(head *ListNode) *ListNode {
    // setに全部いれる．全部取得して，小さい順に並べ直す. いやこれもやる必要はないか．
    // すでにソートされているのだから，順番に舐めていって，直前とことなったらpickすればよい

    node := head
    var resultHead, resultTail *ListNode
    var prev *ListNode
    for node != nil {
        if prev == nil {
            resultHead = node
            resultTail = node
        }
        if prev != nil && prev.Val != node.Val {
            resultTail.Next = node
            resultTail = node
        }
        node = node.Next
    }
    return resultHead
}
```
- これはfail.


# 2回目

```go
func deleteDuplicates(head *ListNode) *ListNode {
    // すでにソートされているのだから，順番に舐めていって，直前とことなったらpickすればよい

    node := head
    var resultHead, resultTail *ListNode
    var prev *ListNode
    for node != nil {
        if prev == nil {
            n := &ListNode{Val: node.Val, Next: nil}
            resultHead = n
            resultTail = n
            prev = node
            node = node.Next
            continue
        }
        if prev.Val == node.Val {
            prev = node
            node = node.Next
            continue
        }

        n := &ListNode{Val: node.Val, Next: nil}
        resultTail.Next = n
        resultTail = n

        prev = node
        node = node.Next
    }
    return resultHead
}
```
- これはpass
- 1回目がfailだった原因は，resultに元のListNodeを直接いれてしまっていたことが大きい
- 後続のノードがresultにはいってしまっていた．
- コードが冗長なので，次のステップで改善する


# 3回目

```go
func deleteDuplicates(head *ListNode) *ListNode {
    // すでにソートされているのだから，順番に舐めていって，直前とことなったらpickすればよい

    if head == nil {
        return nil
    }

    node := head
    resultHead := &ListNode{Val: node.Val, Next: nil}
    resultTail := resultHead

    prev := node
    node = node.Next

    for node != nil {
        if prev.Val != node.Val {
            resultTail.Next = &ListNode{Val: node.Val, Next: nil}
            resultTail = resultTail.Next
        }

        prev = node
        node = node.Next
    }
    return resultHead
}
```
- ちょっとマシに．
- prevとかheadのガードとかうまいこと消せそうな


# 4回目

```go
func deleteDuplicates(head *ListNode) *ListNode {
    // すでにソートされているのだから，順番に舐めていって，**直後**と異なったらpickすればよい
    node := head
    if node == nil {
        return nil
    }

    resultHead := &ListNode{Val: node.Val, Next: nil}
    resultTail := resultHead

    for node != nil {
        if node.Next != nil && node.Val != node.Next.Val {
            resultTail.Next = &ListNode{Val: node.Next.Val, Next: nil}
            resultTail = resultTail.Next
        }

        node = node.Next
    }
    return resultHead
}
```
- あまり綺麗にならなかった
- のでLLMにアドバイスもらって，「削除なんだからin-placeでやるのが自然だろ」と言われた．いや〜そうだね．ということで次．


# 5回目

```go
func deleteDuplicates(head *ListNode) *ListNode {
    // すでにソートされているのだから，順番に舐めていって，**直後**と異なったらpickすればよい
    node := head
    for node != nil && node.Next != nil {
        if node.Val == node.Next.Val {
            // node.Nextを削除するようにリストを繋ぎ変え
            node.Next = node.Next.Next
        } else {
            node = node.Next
        }
    }
    return head
}

```
- 短く，単純になった．
- ので，これをなにも見ずに5回一発で通すようにした．

---

# after memo

- 直近の別の方のPRをいくつかみると，再帰や番兵と言った手法が挙げられている．これらの選択肢も挙げられるようになる必要がある．
    - https://github.com/attractal/leetcode/pull/3/changes
    - https://github.com/Manato110/LeetCode-arai60/pull/3/changes
    - https://github.com/TakashiMITANI/SWEAssociationOnlineCodingPractice/pull/17/changes
- あとはこれまで書いてなかったけど，時間計算量や空間計算量について検討してメモしたほうがよさそうだ（間違ったら指摘してもらえる）
    - 長さNのリストを一度ループするだけなので，時間計算量としてはO(n)
    - in-placeにできたので，空間計算量はO(1)
