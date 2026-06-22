# 問題
https://leetcode.com/problems/find-k-pairs-with-smallest-sums

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。
---

# 1回目

**これはACしないコードです．**
```go
// 方針:
// - nums1, nums2をそれぞれmin-heapにいれる. heap1, heap2と呼ぶ
// - 一つずつ取り出して，pairを作る. それぞれから取り出した要素をa, bとする．
// - k回ループする
//   - aとbを比較する．大きいほうを選ぶ．
//   - aが大きい時は，heap1から取り出して，これをaとする
//   - bが大きい時は，heap2から取り出して，これをbとする
//   - (a, b)をpairとして記録する
// - kつ集まったら returnする

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
    x := old[len(old)-1]
    *h = old[:len(old)-1]
    return x
}

func kSmallestPairs(nums1 []int, nums2 []int, k int) [][]int {
    heap1 := &IntMinHeap{}
    heap.Init(heap1)
    for _, x := range nums1 {
        heap.Push(heap1, x)
    }

    heap2 := &IntMinHeap{}
    heap.Init(heap2)
    for _, x := range nums2 {
        heap.Push(heap2, x)
    }

    result := [][]int{}
    a := heap.Pop(heap1).(int)
    b := heap.Pop(heap2).(int)
    result = append(result, []int{a, b})
    for i := 1; i < k; i++ {
        if a < b {
            b = heap.Pop(heap2).(int)
        } else if a > b {
            a = heap.Pop(heap1).(int)
        } else {
            var _a, _b int
            if len(*heap1) > 0 {
                _a = (*heap1)[0]
            }
            if len(*heap2) > 0 {
                _b = (*heap2)[0]
            }

            if _a < _b {
                a = heap.Pop(heap1).(int)
            } else if _a > _b {
                b = heap.Pop(heap2).(int)
            }
        }
        result = append(result, []int{a, b})
    }

    return result
}
```
- そもそも問題文を正しく理解できていなかった．
- 他の人の解法を見にいく
    - https://github.com/h-masder/Arai60/pull/11/changes
    - https://github.com/Shunii85/arai60/pull/10/changes
    - https://github.com/dorxyxki/arai60/pull/10/changes
    - https://github.com/n6o/leetcode_arai60/pull/10/changes


# 2回目

```go
type pair struct {
    index1 int
    index2 int
    sum int
}

type SumMinHeap []pair

func (h SumMinHeap) Len() int {
    return len(h)
}

func (h SumMinHeap) Less(i, j int) bool {
    return h[i].sum < h[j].sum
}

func (h SumMinHeap) Swap(i, j int) {
    h[i], h[j] = h[j], h[i]
}

func (h *SumMinHeap) Push(x any) {
    *h = append(*h, x.(pair))
}

func (h *SumMinHeap) Pop() any {
    old := *h
    x := old[len(old)-1]
    *h = old[:len(old)-1]
    return x
}

func kSmallestPairs(nums1 []int, nums2 []int, k int) [][]int {
    result := [][]int{}
    h := &SumMinHeap{}

    if len(nums1) == 0 || len(nums2) == 0 || k == 0 {
        return result
    }

    for i := 0; i < len(nums1) && i < k; i++ {
        heap.Push(h, pair{
            index1: i,
            index2: 0,
            sum: nums1[i] + nums2[0],
        })
    }

    for len(result) < k && h.Len() > 0 {
        node := heap.Pop(h).(pair)
        result = append(result, []int{nums1[node.index1], nums2[node.index2]})

        if node.index2+1 < len(nums2) {
            heap.Push(h, pair{
                index1: node.index1,
                index2: node.index2+1,
                sum: nums1[node.index1]+nums2[node.index2+1],
            })
        }
    }
    return result
}
```
- 方針:
    - heapを用意して，nums1, nums2のindexとsumを持ったstructを保持する．sumのmin-heapを作る
    - 全組み合わせを考慮することは計算量的にできない
    - なので，nums1,
    nums2がそれぞれ昇順でソートされていることを利用して，heapのサイズをなるべく小さく保つようにする
    - 最初に(i,0)をheapに積む.
    その後，一つpickするごとに(i,j+1)をpush（次の1列分）して，heapからひとつ取り出す．k回取り出したら終了

- index2を`+1`してPushすべきところを，sumの行で，index2+1してなくてエラーになってしまった．

# 3回目
```go
type pair struct {
    index1 int
    index2 int
    sum int
}

type SumMinHeap []pair

func (h SumMinHeap) Len() int {
    return len(h)
}

func (h SumMinHeap) Less(i, j int) bool {
    return h[i].sum < h[j].sum
}

func (h SumMinHeap) Swap(i, j int) {
    h[i], h[j] = h[j], h[i]
}

func (h *SumMinHeap) Push(x any) {
    *h = append(*h, x.(pair))
}

func (h *SumMinHeap) Pop() any {
    old := *h
    x := old[len(old)-1]
    *h = old[:len(old)-1]
    return x
}

func kSmallestPairs(nums1 []int, nums2 []int, k int) [][]int {
    result := [][]int{}
    h := &SumMinHeap{}

    if len(nums1) == 0 || len(nums2) == 0 || k == 0 {
        return result
    }

    for i := 0; i < len(nums1) && i < k; i++ {
        heap.Push(h, pair{
            index1: i,
            index2: 0,
            sum: nums1[i] + nums2[0],
        })
    }

    for len(result) < k && h.Len() > 0 {
        node := heap.Pop(h).(pair)
        result = append(result, []int{nums1[node.index1], nums2[node.index2]})

        if node.index2+1 < len(nums2) {
            heap.Push(h, pair{
                index1: node.index1,
                index2: node.index2+1,
                sum: nums1[node.index1]+nums2[node.index2+1],
            })
        }
    }
    return result
}
```
