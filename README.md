# pkginfomapper

## Description

Generates Go code from a template referencing package information such as package name, imports, and struct fields.

## Examples

### Accessors

`./foo.go` contains a struct `Foo`:

```go
package main

import (
	goimports "github.com/incu6us/goimports-reviser/v3/reviser"
	"text/template"
)

type Foo struct {
	bar  string `pkginfomapper:"setter,getter=true"`
	Baz  int
	qux  template.Template   `pkginfomapper:"getter,setter=false"`
	quux goimports.SourceDir `pkginfomapper:"setter"`
}
```

`./gen_foo_accessors_go/main.go` is a accessors generator for the struct:

```go
package main

import (
	"github.com/knaka/pkginfomapper"
)

func main() {
	structInfo, err := pkginfomapper.GetStructInfo(".", "Foo", map[string]any{
		"AdditionalComment": "This is a comment.",
	})
	if err != nil {
		panic(err)
	}
	err = pkginfomapper.GenerateForStruct(pkginfomapper.AccessorsTemplate, structInfo)
	if err != nil {
		panic(err)
	}
}
```

`go run ./gen_foo_accessors/` will generate the following code as `./foo_accessors.go`:

```foo_accessors.go
// Code generated by gen_foo_accessors; DO NOT EDIT.

package main

// This is a comment.

import (
	"text/template"

	"github.com/incu6us/goimports-reviser/v3/reviser"
)

func (rcv *Foo) GetBar() string {
	return rcv.bar
}

func (rcv *Foo) SetBar(value string) {
	rcv.bar = value
}

func (rcv *Foo) GetQux() template.Template {
	return rcv.qux
}

func (rcv *Foo) SetQuux(value reviser.SourceDir) {
	rcv.quux = value
}
```

While the above example is using a template `AccessorsTemplate`, you can use your own template by passing it to `GenerateForStruct`. `AccessorsTemplate` is defined as follows:

```gotemplate
// Code generated by {{.GeneratorName}}; DO NOT EDIT.

package {{$.PackageName}}

import (
{{range $.Imports}}
	"{{.}}"
{{end}}
)

{{range $.PrivateFields}}
{{if eq .Params.getter "true"}}
func (rcv *{{$.StructName}}) Get{{.CapName}}() {{.Type}} {
	return rcv.{{.Name}}
}
{{end}}{{/* if */}}

{{if eq .Params.setter "true"}}
func (rcv *{{$.StructName}}) Set{{.CapName}}(value {{.Type}}) {
	rcv.{{.Name}} = value
}
{{end}}{{/* if */}}
{{end}}{{/* range */}}
```
