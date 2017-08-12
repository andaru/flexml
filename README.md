# flexml #

Golang library `github.com/andaru/flexml` is a fork of the golang
standard library's `encoding/xml` package offering additional features
and fixes, particularly to XML namespace related issues. It is API
compatible with the `encoding/xml` package.

## Installation ##

Install the latest version of the library like so:

```
$ go get -u github.com/andaru/flexml
```

## Usage ##

To use `flexml`, replace all references in your program to the
import:

```
import (
	"encoding/xml"
)
```

with the API compatible import of `flexml`:

```
import (
	xml "github.com/andaru/flexml"
)
```

Any custom decoding loop using `(*Decoder).Token()` can then
access the additional features of `flexml` (see below).

If your program receives types from code using `encoding/xml`
types, you will need to convert these to the types in the
`flexml` package when using `flexml`'s types, e.g.:

```go
import (
	"encoding/xml"

	"github.com/andaru/flexml"
)

func newName(n xml.Name)                         flexml.Name { return flexml.Name(n) }
func newStartElement(e xml.StartElement) flexml.StartElement { return flexml.StartElement(e) }

```

## Additional features ##

`flexml` provides the following features above and beyond
`encoding/xml`:

### Prefix to namespace registration accessors ###

The `Decoder`'s current prefix to namespace registrations are
available via the new functions:
- `(*Decoder).Namespace(prefix string) (namespace string)`
	- Returns the namespace associated with the prefix (if any)
- `(*Decoder).Prefix(namespace string) (prefix string)`
	- Returns the (first) prefix associated with the namespace (if any)

`prefix` to `namespace` registrations (via the `Namespace()`
function) are typically used while decoding `Attr` or `CharData`
values within specific elements where the values represent
identifiers defined in XML namespaces (e.g., via a schema such
as [XSD](https://www.w3.org/XML/Schema)
or [YANG](https://tools.ietf.org/html/rfc7950)).

### Unknown element handling while unmarshaling into structs ###

During struct unmarshaling performed by `Unmarshal()`,
`(*Decoder).Decode()` or `(*Decoder).DecodeElement()`, if an
element name is seen on the input stream which does not
correspond a struct field on the type being unmarshaled into, an
`UnknownElementHandler` callback can be fired.

An `UnknownElementHandler` can return an error (allowing for
"strict" struct unmarshaling) or perform custom processing of the
element, record or simply ignore the element by returning with no
error.

Strict struct field handling example:

```go
import (
	"io"
	"errors"

	"github.com/andaru/flexml"
)

var r io.Reader = ...
d := flexml.NewDecoder(r)

// strict field handling: elements in the struct must be matched by the decoder
d.UnknownElementHandler = func(*flexml.Decoder, flexml.StartElement) error {
	return errors.New("unexpected element")
}
// decode input
var item SomeStruct
if err := d.Decode(&item); err != nil { ... }
```

### Input column number tracking ###

The column number of the current input stream reader position is
recorded by the `*Decoder`. This joins the line number and
absolute input offset values previously stored. The column number
reports the last character of the most recently read `Token()`.

### Access to Decoder's current line number and column number ###

New functions are offered to expose the current line number and
column number of the Decoder's location within the input stream.
The decoder's position is at the end of the most recent `Token()`
returned.
- `(*Decoder).LineNo() int`
    - Returns the current line number of the decoder
- `(*Decoder).ColumnNo() int`
    - Returns the current column number of the decoder

### Exported SyntaxError constructor ###

The function `DecoderSyntaxError(d *Decoder, msg string)`
generates a custom `SyntaxError` at `Decoder`'s current position
in the input stream.
