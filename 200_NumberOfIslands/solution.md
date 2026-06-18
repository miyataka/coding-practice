# 問題
https://leetcode.com/problems/number-of-islands

`'1'`（陸）と `'0'`（水）からなる `m x n` の2次元グリッド `grid` が与えられる。島（island）の**個数**を返す。島は水に囲まれており、隣接する陸（上下左右の4方向）が連結してできる。グリッドの四辺の外側はすべて水とみなしてよい。

- 例1:
  ```
  grid = [
    ["1","1","1","1","0"],
    ["1","1","0","1","0"],
    ["1","1","0","0","0"],
    ["0","0","0","0","0"]
  ]
  ```
  → `1`
- 例2:
  ```
  grid = [
    ["1","1","0","0","0"],
    ["1","1","0","0","0"],
    ["0","0","1","0","0"],
    ["0","0","0","1","1"]
  ]
  ```
  → `3`
- 制約: `m == grid.length`、`n == grid[i].length`、`1 <= m, n <= 300`、`grid[i][j]` は `'0'` か `'1'`

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。
---

# 最初の見積もり
- 入力サイズ: m, n ≤ 300 なので セル数 = m*n ≤ 9*10^4
- 時間計算量: O(m*n) * 4 →O(m*n)
  - 実行時間の見積もり: 最大セル数(300*300)だとすると 9*10^4 / 10^8 = 約0.9ms
- 空間計算量: O(m*n) 全部'1'
  - 必要メモリの見積もり: 関数スタックのメモリサイズ
      - アドレス(1byte)+2つのint(8byte) とすると，24 byte * 9 * 10^4 = 2.16 MB

# 1回目
```go
func numIslands(grid [][]byte) int {
    count := 0
    for i := range grid {
        for j := range grid[i] {
            if grid[i][j] == '1' {
                count++
                sink(grid, i, j)
            }
        }
    }
    return count
}

func sink(grid [][]byte, i, j int) {
    if i < 0 || i >= len(grid) || j < 0 || j >= len(grid[0]) || grid[i][j] != '1' {
        return
    }
    grid[i][j] = '0'
    sink(grid, i-1, j)
    sink(grid, i+1, j)
    sink(grid, i, j-1)
    sink(grid, i, j+1)
}

```
- 方針を立ててみたがコーディングできず．
- LLMに相談して回答をしてもらう．理解できたと思ったので書いてみる
- 理解を文字に起こすと
    - `numIslands` は二重ループで陸地を探す．見つけたら，count-upして，sinkを呼び出す．
    - `sink` は
        gridの当該マスが範囲外でないこと，陸地でないことを確認する．該当するようならそのままreturn
        - ガード節を突破しているなら陸地のはずなので，当該マスを上書きしたうえで，その四方に対してsinkを再帰的に呼び出す
        - ガード節と上書きが，再帰停止するための条件
- 読んだうえで感じること
    - in-placeで更新しているので，これをin-placeでないversionを作るのは現実的なのだろうか？
        - sinkに入る前にcopyして，sink以降はin-placeにすれば，作れそう
    - 再帰とループは書き換え可能だとどっかでみたので，それをやればよさそう．
    - ループに書き換えることができれば，スタックオーバーフローも心配しなくてよくなる
    - sinkのガード節はiとjに対する判定条件は同じ構造なので，改行とかいれてもよいかもしれない
        - インデントなんか微妙だなと思って調べたらこれは非推奨のようだ．
        - https://google.github.io/styleguide/go/decisions#conditionals-and-loops

# 2回目
```go
# in-placeじゃないver
func numIslands(grid [][]byte) int {
    count := 0
    gridCopy := make([][]byte, len(grid), len(grid))
    for i, v := range grid {
        gridCopy[i] = append([]byte{}, v...)
    }

    for i := range gridCopy {
        for j := range gridCopy[i] {
            if gridCopy[i][j] == '1' {
                count++
                sink(gridCopy, i, j)
            }
        }
    }
    return count
}

func sink(grid [][]byte, i, j int) {
    if i < 0 || i >= len(grid) || j < 0 || j >= len(grid[0]) || grid[i][j] != '1' {
        return
    }
    grid[i][j] = '0'
    sink(grid, i-1, j)
    sink(grid, i+1, j)
    sink(grid, i, j-1)
    sink(grid, i, j+1)
}


# loop ver (sinkのみ変更)
func sink(grid [][]byte, si, sj int) {
    stack := [][2]int{{si, sj}}
    grid[si][sj] = '0'

    for len(stack) > 0 {
        // pop
        cell := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        i, j := cell[0], cell[1]

        // 四方をみる
        for _, d := range [4][2]int{{-1, 0}, {1, 0}, {0, -1}, {0, 1}} {
            ni, nj := i+d[0], j+d[1]
            if ni < 0 || ni >= len(grid) || nj < 0 || nj >= len(grid[0]) || grid[ni][nj] != '1' {
                continue
            }
            grid[ni][nj] = '0'
            stack = append(stack , [2]int{ni, nj})
        }
    }
}
```
- DFSをloopにするには，関数の呼び出しスタックを明示的なstackで置き換える必要があるとのこと
    - 言葉だけで言われてもなんのこっちゃと思ったが，コードをみるとわかった気がした．

# 3回目
```go
# recursive
func numIslands(grid [][]byte) int {
    count := 0
    for i := range grid {
        for j := range grid[i] {
            if grid[i][j] == '1' {
                count++
                sink(grid, i, j)
            }
        }
    }
    return count
}

func sink(grid [][]byte, i, j int) {
    if i < 0 || i >= len(grid) || j < 0 || j >= len(grid[0]) || grid[i][j] != '1' {
        return
    }

    grid[i][j] = '0'
    sink(grid, i+1, j)
    sink(grid, i-1, j)
    sink(grid, i, j+1)
    sink(grid, i, j-1)
}

# loop
func numIslands(grid [][]byte) int {
    count := 0
    for i := range grid {
        for j := range grid[i] {
            if grid[i][j] == '1' {
                count++
                sink(grid, i, j)
            }
        }
    }
    return count
}

func sink(grid [][]byte, i, j int) {
    grid[i][j] = '0'
    stack := [][2]int{{i, j}}

    for len(stack) > 0 {
        cell := stack[len(stack)-1]
        stack = stack[:len(stack)-1]

        for _, d := range [4][2]int{{1, 0}, {-1, 0}, {0, 1}, {0, -1}} {
            ni, nj := cell[0]+d[0], cell[1]+d[1]
            if ni < 0 || ni >= len(grid) || nj < 0 || nj >= len(grid[0]) || grid[ni][nj] != '1' {
                continue
            }

            grid[ni][nj] = '0'
            stack = append(stack, [2]int{ni, nj})
        }
    }
}
```
- 3回ずつ繰り返した
- Goのスタックサイズ上限は1GBのようだ
    - https://cs.opensource.google/go/go/+/refs/tags/go1.26.4:src/runtime/proc.go;l=156-163
