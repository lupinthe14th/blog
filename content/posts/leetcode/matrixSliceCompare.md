---
title: "順番が一定ではないm×n行列文字列スライスの比較"
date: 2020-04-07T14:45:43+09:00
tags: ["Go", "golang", "Memo", "leetcode", "matrix", "string", "int", "diff", "sort", "備忘録"]
Math: true
draft: false
---

## はじめに

leetcodeの問題 [Group Anagrams](https://leetcode.com/problems/group-anagrams/) は出力が \\( m \times n \\) 行列のstringのスライスで、Acceptする解では順番は関係ありません。
単に解法を実装してAcceptするまでSubmitするなら考える必要ありませんが、個人的にExampleにあったInputとOutputをテストコードに実装することも行っているので、この順不同が厄介です。

本稿では、今回とったテストコードの回避策について備忘録かねてメモします。


## [reflect.DeepEqual](https://golang.org/pkg/reflect/#DeepEqual) でダメなの？

出力される順番が常に一定ならば、 [reflect.DeepEqual](https://golang.org/pkg/reflect/#DeepEqual) で比較すれば十分です。
しかし順番が関係ない場合、たとえば次のようなgotのパターンでもすべて等しいと判定させる必要があります。

```golang
package main

import (
	"fmt"
	"reflect"
	"testing"
)

func TestMatrix(t *testing.T) {
	want := [][]string{
		{"abc", "def", "ghi"},
		{"jk", "lm"},
		{"n"},
	}

	tests := []struct {
		got [][]string
	}{
		{
			got: [][]string{
				{"abc", "def", "ghi"},
				{"jk", "lm"},
				{"n"},
			},
		},
		{
			got: [][]string{
				{"jk", "lm"},
				{"n"},
				{"abc", "def", "ghi"},
			},
		},
		{
			got: [][]string{
				{"n"},
				{"abc", "def", "ghi"},
				{"jk", "lm"},
			},
		},
		{
			got: [][]string{
				{"def", "ghi", "abc"},
				{"jk", "lm"},
				{"n"},
			},
		},
	}
	for i, tt := range tests {
		t.Run(fmt.Sprint(i), func(t *testing.T) {
			if !reflect.DeepEqual(tt.got, want) {
				t.Fatalf("got: %v, want: %v", tt.got, want)
			}
		})
	}
}
```

https://play.golang.org/p/j29ADH5ndC1


## 回避策

出力された結果の順序を揃えて比較させます。

今回最終的にとった方法は、この調査の過程で利用することにした [package cmp](https://pkg.go.dev/github.com/google/go-cmp/cmp?tab=doc) の [Example (SortedSlice)](https://pkg.go.dev/github.com/google/go-cmp/cmp?tab=doc#example-Option-SortedSlice) を参考に実装しました。


```golang
package main

import (
	"fmt"
	"sort"
	"testing"

	"github.com/google/go-cmp/cmp"
)

func TestMatrix(t *testing.T) {
	want := [][]string{
		{"abc", "def", "ghi"},
		{"jk", "lm"},
		{"n"},
	}

	tests := []struct {
		got [][]string
	}{
		{
			got: [][]string{
				{"abc", "def", "ghi"},
				{"jk", "lm"},
				{"n"},
			},
		},
		{
			got: [][]string{
				{"jk", "lm"},
				{"n"},
				{"abc", "def", "ghi"},
			},
		},
		{
			got: [][]string{
				{"n"},
				{"abc", "def", "ghi"},
				{"jk", "lm"},
			},
		},
		{
			got: [][]string{
				{"def", "ghi", "abc"},
				{"jk", "lm"},
				{"n"},
			},
		},
	}
	// This Transformer sorts a [][]string.
	trans := cmp.Transformer("Sort", func(in [][]string) [][]string {
		out := append([][]string(nil), in...) // Copy input to avoid mutating it
		sort.Slice(out, func(i, j int) bool {
			for _, v := range out {
				sort.Strings(v)
			}
			return out[i][0] < out[j][0]
		})
		return out
	})
	for i, tt := range tests {
		t.Run(fmt.Sprint(i), func(t *testing.T) {
			if !cmp.Equal(tt.got, want, trans) {
				t.Fatalf("got: %v, want: %v", tt.got, want)
			}
		})
	}
}
```

https://play.golang.org/p/n6-dG4h_kkO

## 補足

> Two slices may be considered equal if they have the same elements, regardless of the order that they appear in. Transformations can be used to sort the slice.
>
> This example is for demonstrative purposes; use [cmpopts.SortSlices](https://pkg.go.dev/github.com/google/go-cmp@v0.4.0/cmp/cmpopts?tab=doc#SortSlices) instead.

とのことなので、[cmpopts.SortSlices](https://pkg.go.dev/github.com/google/go-cmp@v0.4.0/cmp/cmpopts?tab=doc#SortSlices) を使うべきなのでしょうが、理解が及ばずとりあえず1つのExampleのテストケースを通すことができたのでヨシとしました。[^1]

[^1]: 簡単な使い方が実装できましたので後述しました。

## 追記

[July LeetCoding Challenge](https://leetcode.com/explore/featured/card/july-leetcoding-challenge/) をやっていたら、 [Subsets](https://leetcode.com/explore/featured/card/july-leetcoding-challenge/545/week-2-july-8th-july-14th/3387/)の問題で同様の出力結果判定がありました。
同じ感じでいけると思いましたが、こちらは空の配列も含まれているので厄介でした。

一応実装したのが次の感じです。


```golang
package main

import (
	"fmt"
	"sort"
	"testing"

	"github.com/google/go-cmp/cmp"
)

func TestMatrix(t *testing.T) {

	want := [][]int{{3}, {1}, {2}, {1, 2, 3}, {1, 3}, {2, 3}, {1, 2}, {}}

	tests := []struct {
		got [][]int
	}{
		{
			got: [][]int{
				{3},
				{1},
				{2},
				{1, 2, 3},
				{1, 3},
				{2, 3},
				{1, 2},
				{},
			},
		},
		{
			got: [][]int{
				{3},
				{1},
				{2},
				{3, 2, 1},
				{1, 3},
				{2, 3},
				{1, 2},
				{},
			},
		},
		{
			got: [][]int{
				{},
				{3},
				{1},
				{2},
				{1, 2, 3},
				{1, 3},
				{2, 3},
				{1, 2},
			},
		},
		{
			got: [][]int{
				{1, 2, 3},
				{3, 1},
				{3, 2},
				{2, 1},
				{3},
				{2},
				{1},
				{},
			},
		},
	}
	// This Transformer sorts a [][]int.
	trans := cmp.Transformer("Sort", func(in [][]int) [][]int {
		out := append([][]int(nil), in...)
		sort.SliceStable(out, func(i, j int) bool {
			for _, v := range out {
				sort.Ints(v)
			}
			if len(out[i]) == 0 && len(out[j]) == 0 {
				return true
			}
			if len(out[i]) == 0 && len(out[j]) != 0 {
				return true
			}

			if len(out[i]) != 0 && len(out[j]) == 0 {

				return false
			}
			if len(out[i]) != len(out[j]) {
				return len(out[i]) < len(out[j])
			}
			r := false
			for k := 0; k < len(out[i]); k++ {
				if out[i][k] != out[j][k] {
					r = out[i][k] < out[j][k]
					break
				}
			}
			return r
		})
		return out
	})
	for i, tt := range tests {
		t.Run(fmt.Sprint(i), func(t *testing.T) {
			if !cmp.Equal(tt.got, want, trans) {
				t.Fatalf("got: %v, want: %v", tt.got, want)
			}
		})
	}
}
```

https://play.golang.org/p/LshQ5pyW44j


## ついで

実装する過程で、[cmpopts](github.com/google/go-cmp/cmp/cmpopts) パッケージを使ったスライスの比較も実装できるようになりましたので紹介します。
Githubでいろいろコードを見れるのは本当にありがたいことです。

`[]cmp.Option` にパッケージ定義済の関数を定義することで調整が簡単になりますね。

```golang
package main

import (
	"fmt"
	"testing"

	"github.com/google/go-cmp/cmp"
	"github.com/google/go-cmp/cmp/cmpopts"
)

func TestMatrix(t *testing.T) {
	tests := []struct {
		got, want []int
	}{

		{
			got: []int{}, want: []int{},
		},
		{
			got: []int{}, want: []int(nil),
		},
		{
			got: []int{1, 2, 3}, want: []int{1, 2, 3},
		},
		{
			got: []int{3, 2, 1}, want: []int{1, 2, 3},
		},
	}
	// opts
	opts := []cmp.Option{
		cmpopts.EquateEmpty(),
		cmpopts.SortSlices(func(x, y int) bool { return x < y }),
	}
	for i, tt := range tests {
		i, tt := i, tt
		t.Run(fmt.Sprint(i), func(t *testing.T) {
			t.Parallel()
			if !cmp.Equal(tt.got, tt.want, opts...) {
				t.Fatalf("got: %v, want: %v", tt.got, tt.want)
			}
		})
	}
}
```

https://play.golang.org/p/-LF5dcNbniq
