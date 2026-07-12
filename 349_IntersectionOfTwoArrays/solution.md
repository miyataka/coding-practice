# 問題
https://leetcode.com/problems/intersection-of-two-arrays

2つの整数配列 `nums1` と `nums2` が与えられる。両方に含まれる要素（積集合）を配列で返す。結果の各要素は一意でなければならない。順序は問わない。

- 例1: `nums1 = [1,2,2,1]`, `nums2 = [2,2]` → `[2]`
- 例2: `nums1 = [4,9,5]`, `nums2 = [9,4,9,8,4]` → `[9,4]`（`[4,9]` でも可）
- 制約: `1 <= nums1.length, nums2.length <= 1000`、`0 <= nums1[i], nums2[i] <= 1000`

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。
---

# 1回目

```go
// 方針．nums1をloopして，hashmapを作る．nums2をloopしながら，hashmapをチェックし，存在していたらresult(hashmap)に追加していく．最後に返す．

func intersection(nums1 []int, nums2 []int) []int {
    nums1Map := map[int]bool{}
    for _, v := range nums1 {
        nums1Map[v] = true
    }
    intersections := map[int]bool{}
    for _, v := range nums2 {
        if exist := nums1Map[v]; exist {
            intersections[v] = true
        }
    }

    result := []int{}
    for k, _ := range intersections {
        result = append(result, k)
    }
    return result
}
```

# 2回目
```go
func intersection(nums1 []int, nums2 []int) []int {
    seen := map[int]bool{}
    for _, v := range nums1 {
        seen[v] = true
    }
    intersections := map[int]bool{}
    for _, v := range nums2 {
        if seen[v] {
            intersections[v] = true
        }
    }

    result := []int{}
    for k := range intersections {
        result = append(result, k)
    }
    return result
}
```
- `nums1Map`を`seen`にリネーム
- `if _, ok := nums1Map[v]; ok` を `if nums1Map[v]` に簡略化

# 3回目
```go
func intersection(nums1, nums2 []int) []int {
    seen := map[int]bool{}
    for _, v := range nums1 {
        seen[v] = true
    }

    intersections := map[int]bool{}
    for _, v := range nums2 {
        if seen[v] {
            intersections[v] = true
        }
    }

    result := []int{}
    for k := range intersections {
        result = append(result, k)
    }
    return result
}
```
- これを3回繰り返した
