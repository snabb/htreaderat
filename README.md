htreaderat
==========

[![GoDoc](https://godoc.org/github.com/snabb/htreaderat?status.svg)](https://godoc.org/github.com/snabb/htreaderat)

The Go package htreaderat implements io.ReaderAt for HTTP requests.

It can be used for example with "archive/zip" package in Go standard
library. Together they can be used to access remote (HTTP accessible)
ZIP files without needing to download the whole archive.

HTTP Range Requests (see [RFC 7233](https://tools.ietf.org/html/rfc7233))
are used to retrieve the requested byte range. Currently an error is
returned if a remote server does not support Range Requests.

When using this package with "archive/zip", it is good idea to use
["github.com/avvmoto/buf-readerat"](https://github.com/avvmoto/buf-readerat)
which implements a buffered io.ReaderAt "proxy". It reduces the amount
of small HTTP requests. See the example for details.


Example
-------

The following example outputs a file list of a remote zip archive without
downloading the whole archive:

```Go
package main

import (
	"archive/zip"
	"fmt"
	"github.com/avvmoto/buf-readerat"
	"github.com/snabb/htreaderat"
	"net/http"
)

func main() {
	req, _ := http.NewRequest("GET", "https://dl.google.com/go/go1.10.windows-amd64.zip", nil)
	htrdr, _ := htreaderat.New(nil, req)
	bhtrdr := bufra.NewBufReaderAt(htrdr, 1024*1024)

	size, err := htrdr.Size()
	if err != nil {
		panic(err)
	}
	zrdr, err := zip.NewReader(bhtrdr, size)
	if err != nil {
		panic(err)
	}
	for _, f := range zrdr.File {
		fmt.Println(f.Name)
	}
}
```


License
-------

MIT
