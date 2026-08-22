# 問題
https://leetcode.com/problems/unique-paths/

m x n のグリッドの左上 `grid[0][0]` にロボットがいる。ロボットは右か下にしか動けない。右下 `grid[m-1][n-1]` に到達する**経路の総数**を int で返す。

- 例1: m = 3, n = 7 → 28
- 例2: m = 3, n = 2 → 3（Right→Down→Down / Down→Down→Right / Down→Right→Down の3通り）
- 制約: 1 <= m, n <= 100。答えは 2 * 10^9 以下になるようにテストケースが作られている。

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。

# 1回目
```go
// 手で解いてみた．これは，下と右にしか進めない一方向なので，上と左のマスのpathの数を足し合わせていくとよい
// これは，m*nの二次元配列を足し合わせていけば良い．端の処理があるので，番兵的に，(m+1)*(n+1)配列で(1,1)からスタートするようにしてもいいかもしれない
// 1 <= m, n <= 100
// 答えは <= 2*10^9なのか
// 見積もり: 最大で100*100マス計算をして，それを記憶すればよいだけかもしれない．
// 10^8をGoの秒間実行ステップ数とすると
// 時間計算量としてはO(MN), 10^4 / 10^8 なので，10^-4 = 0.1ms程度.
// 空間計算量としては，10^4 * 8B = 80KB程度
func uniquePaths(m int, n int) int {
    boards := make([][]int, m)
    for i := 0; i < m; i++ {
        boards[i] = make([]int, n)
        if i == 0 {
            boards[0][0] = 1
        }
        for j := 0; j < n; j++ {
            var up, left int
            if i != 0 {
                up = boards[i-1][j]
            }
            if j != 0 {
                left = boards[i][j-1]
            }
            if left + up > 0 {
                boards[i][j] = left + up
            }
        }
    }
    return boards[m-1][n-1]
}
```
- まずは番兵なしでとりあえず解いた．
- `if left + up > 0` の条件に気づかず，だいぶ時間をロスした

# 2回目
```go
func uniquePaths(m int, n int) int {
    boards := make([][]int, m+1)
    boards[0] = make([]int, n+1)
    for i := 1; i < m+1; i++ {
        boards[i] = make([]int, n+1)
        boards[1][1] = 1
        for j := 1; j < n+1; j++ {
            up := boards[i-1][j]
            left := boards[i][j-1]
            if left + up > 0 {
                boards[i][j] = left + up
            }
        }
    }
    return boards[m][n]
}
```
- 番兵的な解き方
- ほかの人のコードを読む
    - https://github.com/MA-yo-TA/leetcode/pull/32
        - なんか自分と全然違う解き方だな
    - https://github.com/chryschron/codings/pull/31
        - 自分とすこし似ている．一つ前のindexの配列だけ保持しておけばいいので，さらに空間計算量を減らせるのか．賢いな
    - https://github.com/skypenguins/coding-practice/pull/35
    - https://github.com/rimokem/arai60/pull/33
    - https://github.com/h-masder/Arai60/pull/36
        - 自分のコードの`left+up > 0`のときに分岐するのではなく，`m=0 or n=0`のときに初期化してしまうほうがわかりやすいかもしれない．
        - 考え方としては，自分のほうは左上から一つずつ計算するやりかたで，`m=0 or n=0`のときに初期値をいれてしまうのは，一番上の行，一番左の列が1パターンしかないことを利用して，多少省略する考え方
- 総合して，自分は逆に組み合わせの公式？での解法を思いつかなかった

# 3回目
```go
func uniquePaths(m, n int) int {
    pathCounts := make([][]int, m+1)
    for i := range pathCounts {
        pathCounts[i] = make([]int, n+1)
    }

    pathCounts[0][1] = 1 // virtual input
    for row := 1; row < m+1; row++ {
        for col := 1; col < n+1; col++ {
            pathCounts[row][col] = pathCounts[row-1][col] + pathCounts[row][col-1]
        }
    }
    return pathCounts[m][n]
}
```
- 他の人のを読んでいて，pathCountsやgridのほうがいいなと思って変数名を修正．
- LLMからの初期化位置のアドバイスや，virtual inputのアイデアを利用してこれにした
- これを3回繰り返した
