# 問題
https://leetcode.com/problems/max-area-of-island

`0`（水）と `1`（陸）からなる `m x n` の2次元グリッド `grid` が与えられる。島（island）は、上下左右の4方向で連結した陸セルの集まり。グリッドの四辺の外側はすべて水とみなす。島の**面積**（その島に属する陸セルの個数）のうち、**最大の面積**を返す。島が1つも無ければ `0` を返す。

- 例1:
  ```
  grid = [
    [0,0,1,0,0,0,0,1,0,0,0,0,0],
    [0,0,0,0,0,0,0,1,1,1,0,0,0],
    [0,1,1,0,1,0,0,0,0,0,0,0,0],
    [0,1,0,0,1,1,0,0,1,0,1,0,0],
    [0,1,0,0,1,1,0,0,1,1,1,0,0],
    [0,0,0,0,0,0,0,0,0,0,1,0,0],
    [0,0,0,0,0,0,0,1,1,1,0,0,0],
    [0,0,0,0,0,0,0,1,1,0,0,0,0]
  ]
  ```
  → `6`（右側の塊が最大）
- 例2: `grid = [[0,0,0,0,0,0,0,0]]` → `0`（島が無い）
- 制約: `m == grid.length`、`n == grid[i].length`、`1 <= m, n <= 50`、`grid[i][j]` は `0` か `1`

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。
---

# 最初の見積もり
- 入力サイズ: m, n ≤ 50 なので セル数 = m*n ≤ 2.5*10^3
- 時間計算量: O(m*n)
  - 実行時間の見積もり: 2.5*10^3 / 10^8 ≈ 2.5*10^-5 s = 0.025ms
- 空間計算量: O(m*n)
  - 必要メモリの見積もり: grid ヘッダ(address(8)+cap(8)+len(8)=24) + i,j(16) = 40byte * m*n = 40 * 2.5*10^3 = 1*10^5 = 100kb

# 1回目
```go
// 以前といたNumber of Islandsと同じ解法が使える．
// つまり，全域を探索しつつ，1を見つけたらその四方を再帰探索して塗りつぶす．
// 再帰関数は，塗りつぶした面積を返す必要がある
// 最も大きな面積を記録しておき，それを返せばよい

func maxAreaOfIsland(grid [][]int) int {
    maxArea := 0
    for i := range grid {
        for j := range grid[i] {
            if grid[i][j] == 1 {
                maxArea = max(maxArea, sink(grid, i, j))
            }
        }
    }
    return maxArea
}

func sink(grid [][]int, i, j int) int {
    if i < 0 || i >= len(grid) || j < 0 || j >= len(grid[0]) || grid[i][j] != 1 {
        return 0
    }
    grid[i][j] = 0
    area := 1
    area += sink(grid, i+1, j)
    area += sink(grid, i-1, j)
    area += sink(grid, i, j+1)
    area += sink(grid, i, j-1)
    return area
}
```
- 一発で通り．ちょっと嬉しい．
- まぁ，前回解いたやり方でもあり，わりと綺麗かな
- 他の実装選択肢としては，in-placeではない方法，再帰ではなくループを使う方法，BFSを使う方法などがあったはず．これらも書いてみる

# 2回目(out-of-place)
```go
# out-of-place ver
func maxAreaOfIsland(grid [][]int) int {
    copyGrid := make([][]int, len(grid), len(grid))
    for i := range grid {
        copyGrid[i] = append([]int{}, grid[i]...)
    }
    maxArea := 0
    for i := range copyGrid {
        for j := range copyGrid[i] {
            if copyGrid[i][j] == 1 {
                maxArea = max(maxArea, sink(copyGrid, i, j))
            }
        }
    }
    return maxArea
}

func sink(grid [][]int, i, j int) int {
    if i < 0 || i >= len(grid) || j < 0 || j >= len(grid[0]) || grid[i][j] != 1 {
        return 0
    }
    grid[i][j] = 0
    area := 1
    area += sink(grid, i+1, j)
    area += sink(grid, i-1, j)
    area += sink(grid, i, j+1)
    area += sink(grid, i, j-1)
    return area
}
```


# 2回目(DFS loop)
```go
func maxAreaOfIsland(grid [][]int) int {
    copyGrid := make([][]int, len(grid), len(grid))
    for i := range grid {
        copyGrid[i] = append([]int{}, grid[i]...)
    }
    maxArea := 0
    for i := range copyGrid {
        for j := range copyGrid[i] {
            if copyGrid[i][j] == 1 {
                maxArea = max(maxArea, sink(copyGrid, i, j))
            }
        }
    }
    return maxArea
}

func sink(grid [][]int, i, j int) int {
    area := 0
    grid[i][j] = 0 // DEBUG時に追加
    stack := [][2]int{{i, j}}

    for len(stack) > 0 {
        area += 1
        cell := stack[len(stack)-1]
        stack = stack[:len(stack)-1]

        for _, d := range [4][2]int{{1, 0}, {-1, 0}, {0, 1}, {0, -1}} {
            ni, nj := cell[0]+d[0], cell[1]+d[1]
            if ni < 0 || ni >= len(grid) || nj < 0 || nj >= len(grid[0]) || grid[ni][nj] != 1 {
                continue
            }
            grid[ni][nj] = 0
            stack = append(stack, [2]int{ni, nj})
        }
    }
    return area
}
```
- 最初のマスを塗りつぶしするのが抜けていた．
- out-of-placeと，DFSをloopにするのはできた．
- BFSはまだ実装がスッとでてくる状態じゃなかったので，次回以降にまた取り組むことにした

- 他の人のコードを読む
    - https://github.com/hiro111208/leetcode/pull/37/changes
        - 数直線のように並べるのはわかりやすそう
    - https://github.com/hiroki-horiguchi-dev/leetcode/pull/18
        - water, landを定数にというのはよさそう
    - https://github.com/nicah4o/arai60/pull/18
        - i, jではなく，r(ow), c(olumn) を使うというのはなるほど
        - union-findについて調べておくとよさそう
    - https://github.com/jjysogfy/arai60-202603/pull/8
    - https://github.com/quinn-sasha/leetcode/pull/30
    - https://github.com/h-masder/Arai60/pull/19

- Union-Find について
    - ツリー構造
    - Disjoint Set Union (DSU) とも呼ばれる
    - 主に使う場面は，グループの結合と，同じグループかどうかの判定
    - 基本操作
        - find(x): xが属している代表元を返す
        - union(x, y): xのグループとyのグループを結合する
    - 動作
        - 初期化で，全ノードをすべてバラバラ（ルートしかない）のツリーとする
        - union処理で，ノードを一つにまとめる（片側をもう片側のparentにする）
        - 小さいツリーを大きいツリーにつなげる（こうするとツリーが深くなりにくくなるらしい）
        - find処理では，基本的に代表元を返すだけだが，経路圧縮と言って，大元の代表元に繋ぎ直す処理も行う. こうすると，ツリーが短く保たれる．

# 3回目

```go
const (
    WATER = iota
    LAND
)

func maxAreaOfIsland(grid [][]int) int {
    maxArea := 0
    for r := range grid {
        for c := range grid[r] {
            if grid[r][c] == LAND {
                maxArea = max(maxArea, sink(grid, r, c))
            }
        }
    }
    return maxArea
}

func sink(grid [][]int, row, column int) int {
    area := 0
    grid[row][column] = WATER
    stack := [][2]int{{row, column}}

    for len(stack) > 0 {
        cell := stack[len(stack)-1]
        stack = stack[:len(stack)-1]
        area++

        for _, d := range [4][2]int{{1, 0}, {-1, 0}, {0, 1}, {0, -1}} {
            nr, nc := cell[0]+d[0], cell[1]+d[1]
            if nr < 0 || len(grid) <= nr {
                continue
            }
            if nc < 0 || len(grid[0]) <= nc {
                continue
            }
            if grid[nr][nc] != LAND {
                continue
            }

            grid[nr][nc] = WATER
            stack = append(stack, [2]int{nr, nc})
        }
    }
    return area
}
```
- これを3回繰り返した
