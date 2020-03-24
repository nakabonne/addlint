```go
package main

import (
	"bytes"
	"fmt"
	"go/ast"
	"go/parser"
	"go/printer"
	"go/token"
	"io/ioutil"
	"os"
)

func main() {
	path := os.Args[1]
	src, err := ioutil.ReadFile(path)
	if err != nil {
		panic(err)
	}

	// AST内のNodeの詳細な位置情報
	fset := token.NewFileSet()
	// ファイルをASTに変換。この得られたfをソースコード内で解析
	f, err := parser.ParseFile(fset, path, src, parser.ParseComments)
	if err != nil {
		panic(err)
	}

	// ASTを深さ優先(depth-first order)探索
	ast.Inspect(f, func(n ast.Node) bool {
		be, ok := n.(*ast.BinaryExpr)
		if !ok {
			return true
		}

		if be.Op != token.ADD {
			return true
		}

		if _, ok := be.X.(*ast.BasicLit); !ok {
			return true
		}

		if _, ok := be.Y.(*ast.BasicLit); !ok {
			return true
		}

		pos := fset.Position(be.Pos())
		fmt.Fprintf(os.Stderr, "%s: addition found: %q\n", pos, render(fset, be))
		return true
	})
}

// render returns the pretty-print of the given node
func render(fset *token.FileSet, x interface{}) string {
	var buf bytes.Buffer
	if err := printer.Fprint(&buf, fset, x); err != nil {
		panic(err)
	}
	return buf.String()
}
```
