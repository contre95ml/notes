## Slices

Go has arrays but those are hardly used. Unlike arrays, slices are typed only by the elements they contain (not the number of elements). These are dynamically allocated while arrays have a fixed length.

Declare an empty array with `make`, fill it and print it by accessing it.

```go
s := make([]string, 3),
s[0] = "a"
 s[1] = "b"
 s[2] = "c"
 fmt.Println("set:", s)
 fmt.Println("get:", s[2])
```

The built-in `append`, which returns a slice containing one or more new values. Note that we need to accept a return value from `append` as we may get a new slice value.

```go
    s = append(s, "d")
    s = append(s, "e", "f")
    fmt.Println("apd:", s)
```

Declare and define a slice in the same line

```go
mySlice := []string{"A","B","C"}
```

Slices can also be `copy`’d. Here we create an empty slice `c` of the same length as `s` and copy into `c` from `s`.

```go
c := make([]string, len(s))
copy(c, s)
fmt.Println("cpy:", c)
```

Slices also support a “slice” operator with the syntax `slice[low:high]`.

```go
l := s[2:5]
fmt.Println("sl1:", l)
```

  