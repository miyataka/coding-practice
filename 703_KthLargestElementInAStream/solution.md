# 問題
https://leetcode.com/problems/kth-largest-element-in-a-stream/description/


# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。
---

# 1回目

```go
import "slices"

type KthLargest struct {
    nums []int
    nth int
}


func Constructor(k int, nums []int) KthLargest {
    slices.SortFunc(nums, func(a, b int) int {
        return b-a
    })
    return KthLargest{
        nums,
        k,
    }
}


func (this *KthLargest) Add(val int) int {
    arr := append(this.nums, val)
    slices.SortFunc(arr, func(a, b int) int {
        return b-a
    })
    this.nums = arr
    return arr[this.nth-1]
}
```
- 考え方: structに逆順ソート済みのsliceを持つ．`Add()`される度にsliceにappendして逆順ソートをやり直す．nthを返す．
- `slices.Sort()` では逆順にならんので，`slices.SortFunc`を調べた．なにも見ずにはかけない
    - https://pkg.go.dev/slices#SortFunc
    - 不安定ソート. 安定ソートは同じ順序ならそのオブジェクトの相対位置が保ったままになるかどうかの違い（という理解）
- 何度かfailさせた．numsの上書きを忘れたのと, return次の`nth-1`を`nth`にしてしまってた

# 2回目

```go
import "slices"

type KthLargest struct {
    nums []int
    nth int
}


func Constructor(k int, nums []int) KthLargest {
    return KthLargest{
        nums,
        k,
    }
}


func (this *KthLargest) Add(val int) int {
    arr := append(this.nums, val)
    slices.SortFunc(arr, func(a, b int) int {
        return b-a
    })
    this.nums = arr
    return arr[this.nth-1]
}

```
- Costructor時点でのSortFuncは無意味だなということで削除
- あまり綺麗にならないし，性能も悪いようだ 1000ms以上かかっている

# 3回目

```go
import "slices"

type KthLargest struct {
    nums []int
    limit int
}


func Constructor(k int, nums []int) KthLargest {
    slices.SortFunc(nums, func(a, b int) int {
        return b-a
    })
    return KthLargest{
        nums,
        k,
    }
}


func (this *KthLargest) Add(val int) int {
    this.nums = append(this.nums, val)
    slices.SortFunc(this.nums, func(a, b int) int {
        return b-a
    })
    this.nums = this.nums[:this.limit]
    return this.nums[len(this.nums)-1]
}
```
- あまりに遅いので，他の人のコードを参照してみた
    - https://github.com/n6o/leetcode_arai60/pull/8/changes
- 自分の実装がnumsを必要以上に保持する作りになっていたことを理解したので，その時点でのnthより後ろを捨てるように．そしてnthをlimitにリネーム
- 選択肢として以下がそれぞれあったことを理解した．
    - インプットを全保持，降順でソート，nthを返す
    - 上記の改良版として降順ソートしてn個だけ保持しておき，末尾を返す方針
    - 最初にソートして，末尾からn個分だけ保持する．その後はAddの度に2分探索を使って挿入位置を特定し，挿入する．この時点で昇順でn+1の長さになっているので，先頭から2番目(i=1)を返すと，大きい数値からn番目を返したことになる
    - 最小ヒープを利用する方法（まだ理解はできていない）


# 4回目

## 都度降順ソート
```go
import "slices"

type KthLargest struct {
    nums []int
    limit int
}


func Constructor(k int, nums []int) KthLargest {
    localNums := nums
    slices.SortFunc(localNums, func(a, b int) int {
        return b-a
    })
    if len(localNums) > k {
        localNums = localNums[:k]
    }

    return KthLargest{
        nums: localNums,
        limit: k,
    }
}


func (this *KthLargest) Add(val int) int {
    localNums := append(this.nums, val)
    slices.SortFunc(localNums, func(a, b int) int {
        return b-a
    })
    if len(localNums) > this.limit {
        this.nums = localNums[:this.limit]
    } else {
        this.nums = localNums
    }
    return this.nums[len(this.nums)-1]
}
```

## 2分探索
```go
type KthLargest struct {
    nums []int
    limit int
}


func Constructor(k int, nums []int) KthLargest {
    localNums := append([]int{}, nums...)
    slices.Sort(localNums)
    if k < len(localNums) {
        localNums = localNums[len(localNums)-k:]
    }

    return KthLargest{
        nums: localNums,
        limit: k,
    }
}


func (this *KthLargest) Add(val int) int {
    insertIndex, _ := slices.BinarySearch(this.nums, val)
    this.nums = slices.Insert(this.nums, insertIndex, val)
    if this.limit < len(this.nums) {
        this.nums = this.nums[1:]
    }
    return this.nums[0]
}
```

## 最小ヒープ
```go
import "container/heap"

type IntMinHeap []int

func (h IntMinHeap) Len() int {
    return len(h)
}

func (h IntMinHeap) Less(i, j int) bool {
    return h[i] < h[j]
}

func (h IntMinHeap) Swap(i, j int) {
    h[i], h[j] = h[j], h[i]
}

func (h *IntMinHeap) Push(x any) {
    *h = append(*h, x.(int))
}

func (h *IntMinHeap) Pop() any {
    old := *h
    n := len(old)
    x := (*h)[n-1]
    *h = (*h)[:n-1]
    return x
}

type KthLargest struct {
    topK *IntMinHeap
    limit int
}


func Constructor(k int, nums []int) KthLargest {
    topK := &IntMinHeap{}
    heap.Init(topK)

    for _, n := range nums {
        heap.Push(topK, n)
        if k < topK.Len() {
            heap.Pop(topK)
        }
    }

    return KthLargest{
        topK: topK,
        limit: k,
    }
}


func (this *KthLargest) Add(val int) int {
    heap.Push(this.topK, val)
    if this.limit < this.topK.Len() {
        heap.Pop(this.topK)
    }
    return (*this.topK)[0]
}
```

- ゴールが「選択肢が見えていて，選べて，書けて，説明できる状態」なので，とりあえず3パターン全部を3回繰り返す
- 最小ヒープの使い所を初めて知った．
    - 自分のヒープの理解は「完全にソートされているわけではない．ただし最小（あるいは最大）がとれることは保証されている．」というもの
