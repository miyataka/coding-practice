# coding-practice

[Arai60](https://1kohei1.com/leetcode/) を上から順に解いている練習リポジトリ。
各問題ディレクトリの `solution.md` に、1〜3 回目のコードと、その過程で考えたこと・詰まったことを記録している。

進め方は各 `solution.md` の「前提」に書いてあるとおり、
「① AC する → ② 読みやすく整える → ③ 10 分以内・ノーエラーで 3 回書ける」の 3 段階。

## 要復習リスト

自分のメモ・コード・PR レビューの 3 つを突き合わせて、**理解が未完了のまま残っている論点**を抜き出したもの。
★ は複数の観点が一致して指摘した最優先項目。

### ヒープの内部動作

- ★ [703. Kth Largest Element in a Stream](703_KthLargestElementInAStream/) — [PR #8](https://github.com/miyataka/coding-practice/pull/8)
  「最小ヒープを利用する方法（まだ理解はできていない）」がそのまま残っている。
  レビューで出された 3 つの問い（データ構造は何か / push・pop の処理はどのようなものか / 完全にソートされていないのに最小が取れるのはなぜか）に、文章で答えられる状態にする。
  宣言した 3 パターン（都度ソート / 二分探索挿入 / 最小ヒープ）× 3 回のドリルも未実施。
- [347. Top K Frequent Elements](347_TopKFrequentElements/) — [PR #9](https://github.com/miyataka/coding-practice/pull/9)
  「min heap でよかったらしい」と伝聞で終わっている。なぜ最小ヒープに k 個だけ保持すると上位 k 件が残るのかを説明できるように。先送りした quick-select / bucket sort も。
- [373. Find K Pairs with Smallest Sums](373_FindKPairsWithSmallestSums/) — [PR #10](https://github.com/miyataka/coding-practice/pull/10)
  703・347 と同じヒープ系。上の 2 問を消化してから通しで復習すると効率がよい。

### 連結リスト: 再帰版・番兵・入力の破壊

- ★ [142. Linked List Cycle II](142_LinkedListCycleII/) — [PR #2](https://github.com/miyataka/coding-practice/pull/2)
  follow-up（O(1) 空間）が `// TODO` の空実装のまま。`c = a` の導出メモは書けているので、**それをコードに落とすだけ**の状態で止まっている。
- ★ [82. Remove Duplicates from Sorted List II](82_RemoveDuplicatesFromSortedListII/) — [PR #4](https://github.com/miyataka/coding-practice/pull/4)
  回答を見て解いた問題。「再帰のほうはまだ考え方が全然しっくりこない」が未解消。
- [83. Remove Duplicates from Sorted List](83_RemoveDuplicatesFromSortedList/) — [PR #3](https://github.com/miyataka/coding-practice/pull/3)
  自分で「再帰や番兵といった選択肢も挙げられるようになる必要がある」と書いたまま未着手。
- [206. Reverse Linked List](206_ReverseLinkedList/) — [PR #7](https://github.com/miyataka/coding-practice/pull/7)
  `# 4回目 (TBC)` が空のコードブロックのまま。follow-up の再帰版が未実装。

補足: 83 と 206 では「**入力を破壊するか**」をレビューで 2 回指摘されている（2 回目は未返信）。
再帰版を書くときに、破壊的か非破壊的かを毎回意識して選ぶ練習にするとよい。

### 木の走査と、その計算量

- ★ [105. Construct Binary Tree from Preorder and Inorder](105_ConstructBinaryTreeFromPreorderAndInorderTraversal/) — [PR #28](https://github.com/miyataka/coding-practice/pull/28)
  **解法そのものが O(N²) のまま**。「予め値とインデックスのマッピングを作っておくことで、時間計算量を削減できる」というレビュー指摘が未着手（`slices.Index` による線形探索が残っている）。
  自分でも「たまたま問題の制約上 Pass したロジックを作れただけという気がしてきた」と書いている。O(N) 版に書き直すこと。
- [98. Validate Binary Search Tree](98_ValidateBinarySearchTree/) — [PR #27](https://github.com/miyataka/coding-practice/pull/27)
  解法を LLM から受け取っている。「in-order, pre-order って自分とは全然違う解法だった．あとでもう一度読む」「それぞれのイメージがつかめていないので，全然頭に入らない」が保留のまま。
  走査順の定義を押さえ直し、「inorder が狭義単調増加なら BST」を自力で書く。
- [103. Binary Tree Zigzag Level Order Traversal](103_BinaryTreeZigzagLevelOrderTraversal/) — [PR #26](https://github.com/miyataka/coding-practice/pull/26)
  記録が残っている唯一の計測が **18 分**（完了条件の 10 分超過）。DFS 版が腑に落ちず 3 回目から脱落している。
  先頭挿入ではなく末尾追加＋最後に reverse で組み直す。

### スタックフレーム / ローカル変数の見積もり

計算量の見積もりのうち、**時間**は書けるようになっている（「Go の秒間ステップ数を 10^8 とすると」の明示は指摘後に定着した）が、**空間の内訳**が繰り返し漏れている。

- ★ [617. Merge Two Binary Trees](617_MergeTwoBinaryTrees/) — [PR #22](https://github.com/miyataka/coding-practice/pull/22)
  非破壊版・ループ版が LLM 出力。ポインタのポインタ版は「頭が追いつかない感じがある」で 3 回ドリルから脱落。
  メモリ見積もりも「（のちほど追記します）」のまま。`**TreeNode` を渡す反復 DFS を parent/isLeft なしで自力で書く。
- [111. Minimum Depth of Binary Tree](111_MinimumDepthOfBinaryTree/) — [PR #21](https://github.com/miyataka/coding-practice/pull/21)
  「スタックフレームは 1000B より十分小さい／100B 程度で見積もることが多い」という指摘が未返信。**1 フレーム ≒ 100B** を自分の標準にする。
- [112. Path Sum](112_PathSum/) — [PR #24](https://github.com/miyataka/coding-practice/pull/24)
  「ローカル変数はあとで考慮するとして」で先送りしたまま。短絡評価版への書き換えも未実施。

### 個別

- ★ [300. Longest Increasing Subsequence](300_LongestIncreasingSubsequence/) — [PR #29](https://github.com/miyataka/coding-practice/pull/29)
  **O(N log N)（tails + 二分探索）がピンとこないまま終了**。O(N²) の DP は 3 回ドリル済み。
  復習の要点は 3 つ。`tails[k]` の定義（長さ k+1 の増加部分列の、末尾として実現可能な最小値）／ `tails` が単調増加になるのはアルゴリズムが維持しているのではなく定義から従うこと／ lower bound を置換する理由（狭義増加だから「x 以上」を探す）。
- [200. Number of Islands](200_NumberOfIslands/) — [PR #17](https://github.com/miyataka/coding-practice/pull/17)
  「方針を立ててみたがコーディングできず」LLM に回答をもらった問題。グリッド DFS を何も見ずに書けるか、再帰 → 明示スタックの変換を「わかった気がした」で止めずに自力で。

## 常設のセルフチェック項目

特定の問題ではなく、毎回の見直しに使うもの。

- **命名は役割ベースにする。** 5 か月・11 PR にわたって同じクラスの指摘が続いている（`resultHead` / `keys` / `stack` / `vals` / `t` など）。
  しかも転移していない証拠があり、111 で自分から `frontier` / `nextFrontier` を使ったのに、102・103 で `nextLayer` に戻して再指摘されている。
- **所要時間を記録する。** 完了条件が「10 分以内・ノーエラー・3 回」なのに、時間の記録がほぼ全問で欠落しており、達成を検証できない。
- **空間計算量はローカル変数まで内訳を書く。** 上の「スタックフレーム / ローカル変数の見積もり」を参照。
- **入力を破壊するかを毎回選ぶ。** 破壊的でよい場面か、非破壊であるべき場面かを一言書く。

## 運用メモ

- PR は問題ごとに `<番号>_<PascalCase問題名>` ブランチを main から切って作成する。タイトルは `<番号>. <問題名>`、本文は LeetCode の URL のみ。
- PR は他の学習者からレビューを受けるためのもの。
