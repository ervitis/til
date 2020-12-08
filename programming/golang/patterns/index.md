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

## FanIn - FanOut

Or pipelines, when we want to paralelize work using I/O and CPU execution.
In my example I wanted to read files from a directory and convert them in pdf.

From
- [Pipelines pattern](https://blog.friendsofgo.tech/posts/patrones-de-concurrencia-pipeline/) (Spanish)
- [Concurrency patterns](https://medium.com/@thejasbabu/concurrency-patterns-golang-5c5e1bcd0833)

The code to generate the random files

> Don't forget to delete previously any files created before inside the `/tmp/files` folder

```bash
#!/usr/bin/env bash

mkdir -p /tmp/files

for i in {0..10000}; do echo "hello world, this is a test file ${i}" > "/tmp/files/File$(printf "%03d" "$i").txt"; done
```

Then, the code to process the files. I'm using the library [gofpdf](github.com/jung-kurt/gofpdf), it's unmantained but it's ok for this example.

```go
const (
  path = "/tmp/files"
)

type (
  StreamData struct {
    data     []byte
    fileName string
  }
)

func loadFileNames() []string {
  info, err := ioutil.ReadDir(path)
  if err != nil {
    panic(err)
  }

  var fileNames []string
  for _, i := range info {
    fileNames = append(fileNames, path+"/"+i.Name())
  }
  return fileNames
}

func openFiles(paths []string) <-chan StreamData {
  streamOut := make(chan StreamData)
  go func() {
    for _, p := range paths {
      f, err := os.Open(p)
      if err != nil {
        fmt.Println(err)
      }
      b, _ := ioutil.ReadAll(f)
      streamData := StreamData{data: b, fileName: f.Name()}
      streamOut <- streamData
    }
    close(streamOut)
  }()
  return streamOut
}

func convertToPdf(done chan bool, streamIn <-chan StreamData) <-chan StreamData {
  streamOut := make(chan StreamData)
  go func() {
    for stream := range streamIn {
      generatePdf(stream.data, stream.fileName)
    }
    close(streamOut)
    done <- true
  }()
  return streamOut
}

func generatePdf(data []byte, fileName string) {
	pdfFile := gofpdf.New("P", "mm", "A4", "arial")
	pdfFile.AddPage()
	pdfFile.SetFont("arial", "", 12)
	pdfFile.Cell(40,10, string(data))
	if err := pdfFile.OutputFileAndClose(fileName + ".pdf"); err != nil {
		fmt.Println(err)
	}
}

func main() {
	start := time.Now()
	files := loadFileNames()

	done := make(chan bool)

	inputStream := openFiles(files)
	convertToPdf(done, inputStream)
	<-done

	fmt.Printf("\nTime finished in %f\n", time.Since(start).Seconds())
}
```