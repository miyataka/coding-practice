# 問題
https://leetcode.com/problems/unique-paths-ii/

m x n の整数グリッド `obstacleGrid` が与えられる。左上 `grid[0][0]` にロボットがいて、右か下にしか動けない。グリッドの `1` は障害物、`0` は空きマスで、障害物のマスは通れない。右下 `grid[m-1][n-1]` に到達する**経路の総数**を int で返す。

- 例1: obstacleGrid = [[0,0,0],[0,1,0],[0,0,0]] → 2（Right→Right→Down→Down / Down→Down→Right→Right）
- 例2: obstacleGrid = [[0,1],[0,0]] → 1
- 制約: m == len(obstacleGrid)、n == len(obstacleGrid[i])、1 <= m, n <= 100、obstacleGrid[i][j] は 0 か 1。答えは 2 * 10^9 以下になるようにテストケースが作られている。

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。

# 1回目
```go
// unique-pathと同じ考え方でおおよそいいはず．
// つまり，m*nのgridを受け取り，それより大きい(m+1)*(n+1)のgridを用意して，up+leftのpathCountをsumしたmapを作って，bottom-right
// に達したときのcountを返せばよい
// 異なるのは，pathCountsを更新するときに，obstacle gridをチェックして，存在するならば，0で更新する，という点
func uniquePathsWithObstacles(obstacleGrid [][]int) int {
    pathCounts := make([][]int, len(obstacleGrid)+1)
    if len(obstacleGrid) == 0 || len(obstacleGrid[0]) == 0 {
        return 0
    }
    rowLength := len(obstacleGrid)+1
    columnLength := len(obstacleGrid[0])+1
    for i := range rowLength {
        pathCounts[i] = make([]int, columnLength)
    }

    pathCounts[0][1] = 1 // virtual input
    for row := 1; row < rowLength; row++ {
        for col := 1; col < columnLength; col++ {
            if obstacleGrid[row-1][col-1] == 1 {
                pathCounts[row][col] = 0
            } else {
                pathCounts[row][col] = pathCounts[row-1][col] + pathCounts[row][col-1]
            }
        }
    }
    return pathCounts[rowLength-1][columnLength-1]
}
```
- `pathCounts[i]`の初期化ループで，rowLength ではなく，`range columnLength`でループしてしまってエラーにした．
- 前回綺麗にしたコードをベースにしているので，ある程度は整理されている
- セルフレビュー
    - obstacleGridにおける`1`がマジックナンバーになっているのでそれは修正するか

# 2回目
```go
func uniquePathsWithObstacles(obstacleGrid [][]int) int {
    if len(obstacleGrid) == 0 || len(obstacleGrid[0]) == 0 {
        return 0
    }

    rowLength := len(obstacleGrid)+1
    columnLength := len(obstacleGrid[0])+1
    pathCounts := make([][]int, rowLength)
    for i := range rowLength {
        pathCounts[i] = make([]int, columnLength)
    }

    pathCounts[0][1] = 1 // virtual input
    const EXIST = 1

    for row := 1; row < rowLength; row++ {
        for col := 1; col < columnLength; col++ {
            if obstacleGrid[row-1][col-1] == EXIST {
                pathCounts[row][col] = 0
            } else {
                pathCounts[row][col] = pathCounts[row-1][col] + pathCounts[row][col-1]
            }
        }
    }
    return pathCounts[rowLength-1][columnLength-1]
}
```
- out-of-range checkも先に済ませたほうがわかりやすいと思って修正した

# 2回目 v2
```go
// 直前のpathCountsのみを保持して空間計算量をO(columnLength)に削減する
func uniquePathsWithObstacles(obstacleGrid [][]int) int {
    if len(obstacleGrid) == 0 || len(obstacleGrid[0]) == 0 {
        return 0
    }

    pathCounts := make([]int, len(obstacleGrid[0]))
    pathCounts[0] = 1

    const EXIST = 1
    for row := 0; row < len(obstacleGrid); row++ {
        for col := 0; col < len(obstacleGrid[0]); col++ {
            if obstacleGrid[row][col] == EXIST {
                pathCounts[col] = 0
            } else {
                up := 0
                if col > 0 {
                    up = pathCounts[col-1]
                }
                pathCounts[col] = pathCounts[col] + up
            }
        }
    }
    return pathCounts[len(obstacleGrid[0])-1]
}
```
- 他の人のコードを読む
    - https://github.com/MA-yo-TA/leetcode/pull/33
        - OBSTACLEという定数名でよかったんや．．．
    - https://github.com/skypenguins/coding-practice/pull/36
        - `-1` を返すケースがあって，個人的には0でよいのではという感想．startからgoalまでのcountを求めるわけなので．
    - https://github.com/rimokem/arai60/pull/34
        - `@cache` アノテーションについて引数をkeyにしてcacheしてくれるらしい https://docs.python.org/ja/3/library/functools.html
    - https://github.com/h-masder/Arai60/pull/37
        - 自分の1回目で書いた，rowLengthよりnumRows, numColsのほうがわかりやすいかもなぁ rows, colsのほうが文字数が揃っててすこし嬉しい
        - 自分のコード(2回目 v2)も，switch文にしたり，continue挟んだりしてもうすこし綺麗にできそうだなぁ

# 3回目
```go
func uniquePathsWithObstacles(obstacleGrid [][]int) int {
    if len(obstacleGrid) == 0 || len(obstacleGrid[0]) == 0 {
        return 0
    }

    pathCounts := make([]int, len(obstacleGrid[0]))
    pathCounts[0] = 1
    const OBSTACLE = 1
    for row := range len(obstacleGrid) {
        for col := range len(obstacleGrid[0]) {
            if obstacleGrid[row][col] == OBSTACLE {
                pathCounts[col] = 0
                continue
            }
            if col == 0 {
                continue
            }
            pathCounts[col] += pathCounts[col-1]
        }
    }
    return pathCounts[len(obstacleGrid[0])-1]
}
```
- これを3回繰り返した
