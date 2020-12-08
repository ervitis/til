# Patterns

## Worker pool

From [gobyexample worker pools example](https://gobyexample.com/worker-pools)

In one example I have used this pattern was making concurrent requests to the controller for concurrency and finding bugs.

Suppose we have our controller:

```go
func CreateUser(w http.ResponseWriter, r *http.Request) {
  ...
}
```

And we want to test it sending a lot of requests and wait for its result.

```go
const (
  numberJobs    = 10
  numberWorkers = 4
)

r := // use the app router

finishJobWorkers := make(chan bool, numberWorkers)
jobs := make(chan int, numberJobs)

fnCreateUserRequest := func(jobs <-chan int, finished chan<- bool) {
  for job := range jobs {
    rr := httptest.NewRecorder()
    body := // create the body to do the request

    req, _ := http.NewRequest(http.MethodPost, "/users", body)
    req.Header.Set("Content-Type", "application/json")

    r.ServeHTTP(rr, req)

    // do asserts

    // send the signal to the channel as the test has finished
    finished <- true
  }
}

for w := 0; w < numberWorkers; w++ {
  // execute in parallel the functions, but because inside the function the 
  // jobs channel will be blocked until we send the signal to start
  go fnCreateUserRequest(jobs, finishJobWorkers)
}

for j := 0; j < numberJobs; j++ {
  // start the jobs sending the number to the channel
  jobs <- j
}
close(jobs)

for w := 0; w < numberWorkers; w++ {
  // wait for the signal which it indicates the job has finished
  <- finished
}
```