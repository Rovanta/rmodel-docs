+++
title = "Installation"
weight = 3
description = """
This page tells you how to get started with the Compose theme.
"""
+++

With [Go module](https://github.com/golang/go/wiki/Modules) support, simply add the following import to your code, and then `go mod [tidy|download]` will automatically fetch the necessary dependencies.

```go
import "github.com/rModel/rModel"
```

Otherwise, run the following Go command to install the `rModel` package:

```sh
$ go get -u github.com/rModel/rModel