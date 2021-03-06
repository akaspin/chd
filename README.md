# chd

Package chd implements the compress, hash, and displace (CHD) minimal perfect
hash algorithm described in [Hash, displace, and compress][1] by Botelho et al.
It provides a map builder that manages adding of items and map creation.

[1]: http://cmph.sourceforge.net/papers/esa09.pdf

## Installation
```sh
go get github.com/robskie/chd
```

## Example

To create a map, you need to first use a map builder which is where you will
add your items. After you have added all your items, you'll need to call Build
from the map builder to create a map. If you have a large number of items, like
more than a hundred thousand, building the map would take a while (from a few
seconds to several minutes) to finish.

```go
import "github.com/robskie/chd"

// Sample Item structure
type Item struct {
  Key   string
  Value int
}

items := []Item{
  {"ab", 0},
  {"cd", 1},
  {"ef", 2},
  {"gh", 3},
  {"ij", 4},
  {"kl", 5},
}

// Create a builder and add keys to it
builder := NewBuilder(nil)
for _, item := range items {
  builder.Add([]byte(item.Key))
}

// Build the map
m, _ := builder.Build()

// Rearrange items according to its map index
items = append(items, make([]Item, m.Cap()-len(items))...)
for i := 0; i < len(items); {
  idx := m.Get([]byte(items[i].key))

  if i != idx && len(items[i].key) > 0 {
    items[i], items[idx] = items[idx], items[i]
    continue
  }

  i++
}

// Do something useful with the map
```

You can also serialize a map and deserialize it later by using Map.Read and
Map.Write methods. Like the Builder.Build method, you need to pass a succinct
array that implements CompactArray when deserializing. This should be the same
as the one used in building the map.

```go
// Serialize the map
w, _ := os.Create("mymap.dat")
m.Write(w)

// Afterwards, you can deserialize it
r, _ := os.Open("mymap.dat")
nm := chd.NewMap()
nm.Read(r)

// Do something useful with the map
```

## API Reference

Godoc documentation can be found [here][2].

[2]: https://godoc.org/github.com/robskie/chd

## Benchmarks

A Core i5 running at 2.3GHz is used for these benchmarks. You can see here that
Builder.Build's running time is directly proportional to the number of keys
while the Map.Get's execution time is dependent on the speed of the CompactArray
used to create the map.

You can run these benchmarks on your machine by typing this command
```go test github.com/robskie/chd -bench=.*``` in terminal.

```
BenchmarkBuild10KKeys-4           30       46166731 ns/op
BenchmarkBuild100KKeys-4           2      672838604 ns/op
BenchmarkBuild1MKeys-4             1    13144765689 ns/op
BenchmarkMapGet100KKeys-4   10000000            133 ns/op
```
