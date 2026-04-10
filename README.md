# go-fAST Transformers

[![Go Reference](https://pkg.go.dev/badge/github.com/nukilabs/transform.svg)](https://pkg.go.dev/github.com/nukilabs/transform)

JavaScript AST transformation library for Go, built for [go-fast](https://github.com/t14raptor/go-fast).

## Installation

```sh
go get github.com/nukilabs/transform
```

## Packages

### deadcode

Dead code elimination for JavaScript. Removes unused functions, classes, and variable declarations through iterative scope-aware analysis. Correctly handles circular dependencies and preserves expressions with side effects.

```go
import (
	"github.com/nukilabs/transform/deadcode"
	"github.com/t14raptor/go-fast/parser"
	"github.com/t14raptor/go-fast/generator"
)

p, _ := parser.ParseFile(`
	function a() { return "a"; }
	function b() { return "b"; }
	b();
`)

deadcode.Eliminate(p, true)

fmt.Println(generator.Generate(p))
// function b() { return "b"; } b();
```

### simplifier

Constant folding and expression simplification for JavaScript. Evaluates constant expressions, folds comparisons, simplifies logical operators, optimizes member access on literals, and handles JavaScript type coercion semantics.

```go
import (
	"github.com/nukilabs/transform/simplifier"
	"github.com/t14raptor/go-fast/parser"
	"github.com/t14raptor/go-fast/generator"
)

p, _ := parser.ParseFile(`!+[]+!+[]`)

simplifier.Simplify(p, true)

fmt.Println(generator.Generate(p))
// 2
```

Selected simplifications:

| Input | Output |
|---|---|
| `![]` | `false` |
| `+!+[]` | `1` |
| `"hello".length` | `5` |
| `[1, 2, 3][0]` | `1` |
| `true ? a : b` | `a` |
| `true && x` | `x` |
| `typeof undefined` | `"undefined"` |

## API

Both packages expose a single entry point:

```go
func deadcode.Eliminate(p ast.VisitableNode, resolve bool)
func simplifier.Simplify(p ast.VisitableNode, resolve bool)
```

The `resolve` parameter controls whether symbol resolution is performed before the transformation. Set it to `true` unless you have already called `resolver.Resolve` on the AST.
