# 問題
https://leetcode.com/problems/top-k-frequent-elements/

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。
---


# 1回目
```go
// 愚直解
// heapを作り，countで最大heapを作る．その後，k回popしてappendしたものをreturn

import "container/heap"

type Int2Count struct {
    Key int
    Val int
}

type IntMaxHeap []Int2Count

func (h IntMaxHeap) Len() int {
    return len(h)
}

func (h IntMaxHeap) Less(i, j int) bool {
    return h[i].Val > h[j].Val // because max heap
}

func (h IntMaxHeap) Swap(i, j int) {
    h[i], h[j] = h[j], h[i]
}

func (h *IntMaxHeap) Push(x any) {
    *h = append(*h, x.(Int2Count))
}

func (h *IntMaxHeap) Pop() any {
    old := *h
    x := old[len(old)-1]
    *h = old[:len(old)-1]
    return x
}

func topKFrequent(nums []int, k int) []int {
    intCountMap := map[int]int{}
    for _, v := range nums {
        if _, ok := intCountMap[v]; ok {
            intCountMap[v] += 1
        } else {
            intCountMap[v] = 1
        }
    }

    fmt.Println(intCountMap)

    intMaxHeap := &IntMaxHeap{}
    heap.Init(intMaxHeap)
    for k, v := range intCountMap {
        heap.Push(intMaxHeap, Int2Count{
            Key: k,
            Val: v,
        })
    }

    fmt.Println(intMaxHeap)

    result := []int{}
    for i := 0; i < k; i++ {
        i2c := heap.Pop(intMaxHeap)
        result = append(result, i2c.(Int2Count).Key)
    }
    return result
}
```
- heapでなんとか愚直解を書いた
- debug printが残っているので消す

# 2回目
```go
import "container/heap"

type FreqEntry struct {
    Num int
    Count int
}

type FreqMinHeap []FreqEntry

func (h FreqMinHeap) Len() int {
    return len(h)
}

func (h FreqMinHeap) Less(i, j int) bool {
    return h[i].Count < h[j].Count
}

func (h FreqMinHeap) Swap(i, j int) {
    h[i], h[j] = h[j], h[i]
}

func (h *FreqMinHeap) Push(x any) {
    *h = append(*h, x.(FreqEntry))
}

func (h *FreqMinHeap) Pop() any {
    old := *h
    x := old[len(old)-1]
    *h = old[:len(old)-1]
    return x
}

func topKFrequent(nums []int, k int) []int {
    num2freq := map[int]int{}
    for _, v := range nums {
        num2freq[v]++
    }

    freqMinHeap := &FreqMinHeap{}
    heap.Init(freqMinHeap)
    for num, cnt := range num2freq {
        if k <= freqMinHeap.Len() {
            if cnt < (*freqMinHeap)[0].Count {
                continue
            }
            heap.Pop(freqMinHeap)
        }
        heap.Push(freqMinHeap, FreqEntry{
            Num: num,
            Count: cnt,
        })
    }

    result := make([]int, k)
    for i := k-1; i >= 0; i-- {
        i2c := heap.Pop(freqMinHeap)
        result[i] = i2c.(FreqEntry).Num
    }
    return result
}
```
- LLMからヒントを得ながら修正
- max heapは使う必要がなかったらしく，min heapでよかったらしい．
- 最小限の要素数だけheapに追加する方法があってなるほどなぁと.
- min heapを使うようになったので，最後のresultのloopが逆順にする必要がでたので修正．境界条件を何度かミスした．これも慣れてしまいたい
- 時間計算量
    - 最初のmapを構築するのに，O(n). n は len(nums)
    - heapを構築するループ
        - Push/PopするのにO(log N)，要素数がkなので，O(log k)
        - ループ回数はnumsの中にあるユニークな要素数 m
        - m 回の log k なので，`O(m log k)`
    - resultを構築するループ
        - k 回ループ，それぞれのループで，O(log k) の処理
        - よって `O(k log k)`
    - O(n) + O(m log k) + O(k log k)
        - O(n + (m+k) log k) = O(n + m log k)
            - n >= m >= k が成り立つので， 2m >= m+k であり，定数倍なので，m log kを残すので．
- 空間計算量
    - num2freqはO(m): ユニークな要素数
    - freqMinHeapはO(k)
    - resultはO(k)
    - よって，O(m+k). n >= m >= kなので，O(m) とするのも可能


- 他の人のPRをみてみる
    - https://github.com/attractal/leetcode/pull/12
        - pythonには `Counter()` という便利なやつがある
        - quick-selectというのがある．quicksortと合わせて常識の範疇らしい https://discord.com/channels/1084280443945353267/1183683738635346001/1185972070165782688
    - https://github.com/hiro111208/leetcode/pull/9/changes
    - https://github.com/rimokem/arai60/pull/9/changes
    - https://github.com/h-masder/Arai60/pull/10/changes
    - https://github.com/Manato110/LeetCode-arai60/pull/9
        - 上記4つまとめての感想
        - pythonは，dict, Counter, heapqを使うケースがあった
    - https://github.com/TaisukeFujise/leetcode_tafujise/pull/5/changes
        - cppは，priority_queueを使っていた
    - https://github.com/hiroki-horiguchi-dev/leetcode/pull/9/changes
        - java
        - bucketソートというやりかたがあるようだ
        - HashMapと書いている箇所に コメントがついていた https://github.com/hiroki-horiguchi-dev/leetcode/pull/9/changes#r2969363286
            - javaにおいて， Mapはinterface, HashMapはclassのようだ
            - https://docs.oracle.com/javase/jp/8/docs/api/java/util/Map.html
            - https://docs.oracle.com/javase/jp/8/docs/api/java/util/HashMap.html
    - https://github.com/rihib/leetcode/pull/20
        - goのはずだが，コードdiffがみれない
        - 解法が列挙してある
            - bucket sort, 計数ソート，counting-sort
                - https://judge.u-aizu.ac.jp/onlinejudge/description.jsp?id=ALDS1_6_A&lang=ja
            - quick-select
            - PDQ (Pattern Defeating Quicksort)
            - min heap
- みた感想
    - quick-select, bucket-sort, pdq はまた今度じっくりみる


# 3回目
```go
import "container/heap"

type FreqEntry struct {
    Num int
    Count int
}

type FreqMinHeap []FreqEntry

func (h FreqMinHeap) Len() int {
    return len(h)
}

func (h FreqMinHeap) Less(i, j int) bool {
    return h[i].Count < h[j].Count
}

func (h FreqMinHeap) Swap(i, j int) {
    h[i], h[j] = h[j], h[i]
}

func (h *FreqMinHeap) Push(x any) {
    *h = append(*h, x.(FreqEntry))
}

func (h *FreqMinHeap) Pop() any {
    old := *h
    x := old[len(old)-1]
    *h = old[:len(old)-1]
    return x
}

func topKFrequent(nums []int, k int) []int {
    num2freq := make(map[int]int)
    for _, v := range nums {
        num2freq[v]++
    }

    freqMinHeap := &FreqMinHeap{}
    heap.Init(freqMinHeap)

    for num, cnt := range num2freq {
        if k <= (*freqMinHeap).Len() {
            if cnt < (*freqMinHeap)[0].Count {
                continue
            }
            heap.Pop(freqMinHeap)
        }
        heap.Push(freqMinHeap, FreqEntry{
            Num: num,
            Count: cnt,
        })
    }

    result := make([]int, k)
    for i := k-1; i >= 0; i-- {
        x := heap.Pop(freqMinHeap)
        result[i] = x.(FreqEntry).Num
    }
    return result
}
```
- これを3回繰り返した
- 改めて方針を言語化しておくと
    - heapを実装する
    - heapに保管する要素は，NumとCountを持つ
    - heapは，k個分だけ保持する
    - heapを構築するときに，k個以下ならPush，k個以上になったら，最小のものとCount比較してより大きい方を残す
    - 最後に，kからdecrementしつつループして，resultを作って返す
