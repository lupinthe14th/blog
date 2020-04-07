---
title: "順番が一定ではないm×n行列文字列スライスの比較"
date: 2020-04-07T14:45:43+09:00
tags: ["Memo", "leetcode", "matrix", "string", "diff", "sort", "備忘録"]
draft: false
---

## はじめに

leetcodeの問題 [Group Anagrams](https://leetcode.com/problems/group-anagrams/) は出力がm×n行列のstringのスライスで、Acceptする解では順番は関係ありません。
単に解法を実装してAcceptするまでSubmitするなら考える必要ないのですが、ExampleにあったInputとOutputをテストコードに実装することも行っているので、この順不同が厄介です。

本稿では、今回とったテストコードの回避策について備忘録かねてメモします。


## [reflect.DeepEqual](https://golang.org/pkg/reflect/#DeepEqual) でダメなの？

出力される順番が常に一定ならば、 [reflect.DeepEqual](https://golang.org/pkg/reflect/#DeepEqual) で比較すれば十分です。
しかし順番が関係ない場合、例えば以下のようなgotのパターンでもすべて等しいと判定させる必要があります。

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

とのことなので、[cmpopts.SortSlices](https://pkg.go.dev/github.com/google/go-cmp@v0.4.0/cmp/cmpopts?tab=doc#SortSlices) を使うべきなのでしょうが、理解が及ばずとりあえず一つのExampleのテストケースを通すことができたのでヨシとしました。
