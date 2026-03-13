# 1回目
```go
package main

type ListNode struct {
	Val int
	Next *ListNode
}

func hasCycle(head *ListNode) bool {
    seen := map[*ListNode]bool{}
	node := head

    for node != nil {
        if seen[node] {
            return true
        }
		seen[node] = true
        node = node.Next
    }

    return false
}
```

## memo
- valでの判定をつくってしまった
- for文での判定を`node.Next != nil`にしてしまって，冗長な判定をしていた．あわせて`head != nil`の判定もしてしまっていたのでこれをなくす必要があった


# 2回目

```go
func hasCycle(head *ListNode) bool {
    seen := map[*ListNode]bool{}
    node := head

    for node != nil {
        if seen[node] {
            return true
        }
        seen[node] = true
        node = node.Next
    }
    return false
}
```
- 引数の型を書き忘れた

# 3回目
```go
func hasCycle(head *ListNode) bool {
    seen := map[*ListNode]bool{}
    node := head

    for node != nil {
        if seen[node] {
            return true
        }
        seen[node] = true
        node = node.Next
    }
    return false
}
```

- syntaxエラーもなく一発で通せた．これを3回繰り返した．

# 4回目，冗長に感じるが，passed
```go
func hasCycle(head *ListNode) bool {
    slow := head
    fast := head

    for fast != nil {
        tmp := fast.Next
        if tmp == nil {
            return false
        }
        if slow == tmp {
            return true
        }
        fast = tmp.Next
        if slow == fast {
            return true
        }
        slow = slow.Next
    }
    return false
}
```

- LLMからのヒントを得て，O(1)のアルゴリズムを実装．
- ポインタだけをメモリに持っているので空間計算量がO(1)の固定値．ポインタ二つ分．
- たしかgoのポインタはuint8のはずなので，*2 で 128bitということだ．
    - 確認作業．unsafe.Sizeofで全部8. byteだよな？ということでドキュメントもみた
    - https://go.dev/play/p/oT_6akFNsN3
    - https://pkg.go.dev/unsafe#Sizeof
        - > Sizeof takes an expression x of any type and returns the size in bytes of a hypothetical variable v as if v was declared via var v = x.
- フロイドのアルゴリズムというらしいが，調べてみると発見者がフロイドかは諸説あるようだ

# 5回目
```go
func hasCycle(head *ListNode) bool {
    slow := head
    fast := head

    for fast != nil && fast.Next != nil {
        slow = slow.Next
        fast = fast.Next.Next
        if slow == fast {
            return true
        }
    }
    return false
}
```

- ずいぶんシンプルになった．
