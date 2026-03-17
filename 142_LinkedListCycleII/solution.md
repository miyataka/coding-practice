# 問題
https://leetcode.com/problems/linked-list-cycle-ii/description/

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

# 4回目
```go
// スタートからループ開始点までの距離をa
// ループ開始点から出会い地点までの距離をb
// 出会い地点からループ開始点に戻るまでの距離をc
func detectCycle(head *ListNode) *ListNode {
    // TODO
}

```
- follow-upのO(1) 空間計算量制限でやってみる
- 解法の理解メモ
    - 連立方程式を作るイメージ
    - スタート位置をSとする
    - ループの開始位置をLとする
    - うさぎ(fast)が亀(slow)に追いついた場所をMとする
    - SとLの距離をaとおく
    - LからMの距離をbとおく
    - （ループになっているので）MからLに向かうルートのうち，bではないルートをcとおく
    - a + (b+c) + b = 「追いつくまでにうさぎが進んだ距離」
    - 亀が進んだ距離は，a+b
    - うさぎは倍速で進行しているから， 2(a+b)
    - うさぎが進んだ距離で方程式を立てれる
    - a + (b+c) + b = 2(a+b)
    - a + 2b + c = 2a + 2b
    - c = a
    - これを踏まえると，カメの進んだ距離 a+b は，b+c（ループになっている部分）
