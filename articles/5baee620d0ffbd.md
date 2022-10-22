---
title: "固定長データを持つ全二分木のシリアライズとデシリアライズ"
emoji: "🌳"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["algorithm"]
published:  true
---

# 概要

全二分木 (全てのノードが葉であるか, 2つの子を持つ二分木) をビット列にシリアライズする方法についてまとめる. なお, 全二分木のデータは固定長であるとする.

# 前提

本記事では, 二分木を次のデータ構造で表す.

```vb
structure Node
   Node left,
   Node right,
   BitArray data,
end structure
```

ここで, ノードが葉である場合, `left`と`right`には null が入る.

# シリアライズ

二分木を後置順で深さ優先探索し, その順にノードをビット列として書き込んでいく. ビット列への変換は次のルールで行う.

- 葉ノードの場合, 1 とそれに続けてデータを書き込む.
- 枝ノードの場合, 0 とそれに続けてデータを書き込む.
- 最後には, 木の終端を示す 0 を書き込む.

この処理は次の疑似コードで表される.

```vb
function serialize(Node node, BitArray bits): BitArray
   if node.left is not null and node.right is not null then
      bits <- serialize(node.left, bits)
      bits <- serialize(node.right, bits)
      bits.push(1)
   else
      bits.push(0)
   end if
   Concatenate node.data to the end of bits
   return bits
end function
```

例として, 各ノードが8ビット固定長のデータを持つ, 次のような二分木を考える.

![Binary Tree](/images/5baee620d0ffbd/binary_tree.png =400x)

これを後置順で探索した結果は
```
d -> e -> b -> f -> g -> c -> a
```
であるから, 上記のルールで変換すれば
```
1 00100011 1 11001111 0 01000101 1 10001000 1 00001100 0 00011010 0 00000001 0
```
となる.

# デシリアライズ

ビット列の先頭から見ていき, 1 ならばデータを基に葉ノードを生成し, スタックに追加する. 0 ならばスタックから2つノードを取り出し, それとデータを基に枝ノードを生成, 同様にスタックに追加する.  そして 0 を見つけたときに, スタックの長さが1ならば, 木の終端に達したとして終了となる.

この処理は次の疑似コードで表される.

```vb
function deserialize(BitArray bits): Node
   stack <- empty stack
   for i = 0, 9, ..., bits_length - 1 do
      data <- bits[i+1..i+8]
      if bits[i] = 1 then
         node <- Node { left: null, right: null, data }
      else
         if stack_length = 1 then
            return stack.pop()
         end if
         left <- stack.pop()
         right <- stack.pop()
         node <- Node { left, right, data }
      end if
      stack.push(node)
   end for
end function
```

シリアライズと同様の例を考える. シリアライズされたビット列
```
1 00100011 1 11001111 0 01000101 1 10001000 1 00001100 0 00011010 0 00000001 0
```
を先頭から x ビット目まで読んだときのスタックの中身は, 次のように変化する. 以下, 葉ノードを`Leaf(data)`, 枝ノードを`Branch(data, left, right)`と表す.
```
x : stack
0 : []
18: [ Leaf(0x23), Leaf(0xCF) ]
27: [ Branch(0x45, Leaf(0x23), Leaf(0xCF)) ]
45: [
       Branch(0x45, Leaf(0x23), Leaf(0xCF)),
       Leaf(0x88),
       Leaf(0x0C)
    ]
54: [
       Branch(0x45, Leaf(0x23), Leaf(0xCF)),
       Branch(0x1A, Leaf(0x88), Leaf(0x0C))
    ]
63: [
       Branch(
          0x01,
          Branch(0x45, Leaf(0x23), Leaf(0xCF)),
          Branch(0x1A, Leaf(0x88), Leaf(0x0C))
       )
    ]
```
よって, 0 である64ビット目を読む時のスタックの長さは1であるから, 処理が終了される. そして, このときのスタックの中身が元の木構造になっている.

