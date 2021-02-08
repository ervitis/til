# Tricks

## Discard body after reading it

```go
defer func() {
    defer res.Body.Close()
    io.Copy(ioutil.Discard, res.Body)
}()
```