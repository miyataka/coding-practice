# 問題
https://leetcode.com/problems/word-ladder

2つの単語 `beginWord` と `endWord`、および単語リスト `wordList` が与えられる。`beginWord` から `endWord` への**変換列**のうち、**最短のもの**に含まれる単語数を返す。変換列は次を満たす:
- 隣り合う単語は**ちょうど1文字だけ**異なる
- `beginWord` を除き、変換列の各単語は `wordList` に含まれていなければならない（`beginWord` は `wordList` に無くてよい）

そのような変換列が存在しなければ `0` を返す。返すのは列の**長さ（始点と終点を含む単語数）**。

- 例1: `beginWord = "hit"`, `endWord = "cog"`, `wordList = ["hot","dot","dog","lot","log","cog"]` → `5`（`hit -> hot -> dot -> dog -> cog`）
- 例2: `beginWord = "hit"`, `endWord = "cog"`, `wordList = ["hot","dot","dog","lot","log"]` → `0`（`cog` が無い）
- 制約: `1 <= beginWord.length <= 10`、`endWord.length == beginWord.length`、`1 <= wordList.length <= 5000`、`wordList[i].length == beginWord.length`、すべて小文字英字、`beginWord != endWord`、`wordList` の単語はすべてユニーク

# 前提
- 答えを見ずに考えて、5分考えて分からなかったら答えを見てください。答えを見て理解したと思ったら、答えを隠して書いてください。筆が進まず5分迷ったら答えを見てください。そして、見ちゃったら一回全部消してやり直しです。答えを送信して、正解になったら、まずは一段階目です。
- 次にコードを読みやすくするようにできるだけ整えましょう。これで動くコードになったら二段階目です。
- そしたらまた全部消しましょう。今度は、時間を測りながら、もう一回、書きましょう。書いてアクセプトされたら文字を消してもう一回書きましょう。これを10分以内に一回もエラーを出さずに書ける状態になるまで続けてください。3回続けてそれができたらその問題はひとまず丸です。
---

# 見積もり
- 入力サイズ: N = len(wordList) ≤ 5000、L = 単語長 ≤ 10、アルファベット 26 種
- 時間計算量: O(N * L * 26)
  - 実行時間の見積もり: N=5000, L=10 なので，5*10^4*26 = 1.3 * 10^6
  - 1.3 * 10^6 / 10^8 = 1.3 * 10^-2 s = 13ms
- 空間計算量: O(N * L)
  - 必要メモリの見積もり: N * L = 5000 * 10 = 5 * 10^4 byte = 50kb

# 1回目
```go
// 方針: word ladderはしりとりのようなものみたいだなと理解
// 一文字ずつ変化させて，次の単語が決まっていくようだ
// 英語の小文字だけなので，全探索すると26^lengthの探索範囲だが，1文字違いのwordのグラフを構築すればよさそう
// beginWordを始点として一文字違いのwordをつなげたグラフを構築し，endWordへの最短経路を求める問題
// 一文字違いのwordを検出するのは，文字列をループして，文字が異なった回数を数えればよい．
// begin→endへの一方向なので，逆順に辿れる必要はない

// 計算量を見積もってみる
// beginWord.length をMとする
// wordList.length をNとする
// グラフ構築は，N^2かかりそう
// 最短経路探索は，BFSは，まだ頭に入ってなくてよくわからない．

// 時間が経ちすぎたので解答をみる.
func ladderLength(beginWord, endWord string, wordList []string) int {
    words := make(map[string]bool, len(wordList))
    for _, w := range wordList {
        words[w] = true
    }
    if !words[endWord] {
        return 0
    }

    queue := []string{beginWord}
    visited := map[string]bool{beginWord: true}
    steps := 1

    for len(queue) > 0 {
        next := []string{}
        for _, word := range queue {
            if word == endWord {
                return steps
            }

            letters := []byte(word)
            for i := 0; i < len(letters); i++ {
                original := letters[i]
                for c := byte('a'); c <= byte('z'); c++ {
                    if c == original {
                        continue
                    }

                    letters[i] = c
                    candidate := string(letters)
                    if words[candidate] && !visited[candidate] {
                        visited[candidate] = true
                        next = append(next, candidate)
                    }
                }
                letters[i] = original
            }
        }
        queue = next
        steps++
    }
    return 0
}
```
- 答えを一度みたうえで，それを見ずに再現した．
- 理解を書き下しておく
- 方針は，
    - まずsetにwordListを格納し，endWordの存在確認を行う．
    - つぎに1字違いのwordが隣接するグラフをたどりながら，最短経路探索のためにBFSを行う，というもの
- BFS部分をさらに自分の理解で書き下す
- まず，queueとvisitedを用意する．最終的にstep数を返す必要があるのでそれも
- queueをloopする．
    - 層ごとに処理するため，next を用意する．最後にqueueを置き換えるのを忘れずに書いておく．stepsをインクリメントする．
- 層に含まれるものを全部処理するためのループを作る．
    - 終了条件，今回の場合はendWordとの一致を調べる．一致したらstepsを返して終了
    - word文字列を，先頭から一文字ずつ置き換えた文字列を作ってwordList/wordsに存在するかをチェックし，存在すれば次の探索層に追加．かつvisitedとしてマーク
- 全部の処理が終わって抜けてきた処理は，経路なしとしてreturn 0

# 2回目
- 典型コメント https://docs.google.com/document/d/11HV35ADPo9QxJOpJQ24FcZvtvioli770WWdZZDaLOfg/edit?tab=t.0
    - pythonのgenerator概念はいまいちわからん．Goではあまりやらなそうだなと感じる． 自分の使える言語だとrubyではできたような記憶があるが，そのときはあまりそこまでの理解がなかった．ので覚えていない
- 他の人のコードをみる
    - https://github.com/h-masder/Arai60/pull/21
        - やっぱ初見では悩むよなと，ただの共感．
        - 典型コメントでも思ったが，pythonのgeneratorについてあまりわかっていないが深追いせず．
    - https://github.com/rimokem/arai60/pull/20
    - https://github.com/komdoroid/arai60/pull/16
        - used_wordsという命名はよさそうだ．自分のvisitedという命名はBFSに引っ張られすぎていたかも．
        - いや，やっぱvisitedでも変わらんなという気持ちになった．
    - https://github.com/hiroki-horiguchi-dev/leetcode/pull/19
        - MiddleWordAndDepthというのはどこに定義されているのか一瞬わからなかった．（ただの感想）
        - あらたにユーザー定義型として作るのはちょっとやりすぎと感じた．おそらく，定義を探すのは手間なわりに，あまりコードの理解に貢献しないので．
    - https://github.com/nicah4o/arai60/pull/19
        - 今回の制約を使うと，eraseする方法があるというのはなるほど．
    - https://github.com/tNita/arai60/pull/19
        - 1文字を任意にしたhashmap, なるほどと思うけど，構築コストの割にあうのかは微妙かもという感じをうける
        - 探索範囲がascii小文字に限らない場合，割にあうようになるのだろう．
        - 双方向BFSという解法があるようだ. LLMの解説を読んでおく
    - https://github.com/quinn-sasha/leetcode/pull/32
- みんなあまりLLMに回答を出力させないのかな，という感想．
    - 自分はわからないときは，生成させて，それに自分で日本語コメントをつけて，つけたところはコードを消す，を繰り返して，日本語だけ残す.
    - 日本語に対応するコードをなんとなく思い出せるようになったら書く．という感じにしてる．
- 一字変えたものを判定する処理は関数に切り出そうかな
    - と思ったが，効率的なisNeighborやoneLetterNeighborsのような名称の関数を書くにはwordsを渡す必要があり，ちょっとそれだと関数名に現れてこない意味を含んでしまうと思って関数化をやめた
    - O(N^2)なら作れるけど，悪化しすぎる．

# 3回目
```go
func ladderLength(beginWord, endWord string, wordList []string) int {
    words := map[string]bool{}
    for _, w := range wordList {
        words[w] = true
    }
    if !words[endWord] {
        return 0
    }

    queue := []string{beginWord}
    visited := map[string]bool{beginWord: true}
    step := 1

    for 0 < len(queue) {
        next := []string{}
        for _, word := range queue {
            if endWord == word {
                return step
            }

            letters := []byte(word)
            for i := range letters {
                original := letters[i]
                for c := byte('a'); c <= byte('z'); c++ {
                    if c == original {
                        continue
                    }
                    letters[i] = c
                    candidate := string(letters)
                    if words[candidate] && !visited[candidate] {
                        visited[candidate] = true
                        next = append(next, candidate)
                    }
                }
                letters[i] = original
            }
        }

        queue = next
        step++
    }
    return 0
}
```
- これを3回書いた
