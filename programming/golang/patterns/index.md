# Patterns

# Design patterns

## Creational patterns

- [Abstract factory](#abstract-factory)
- Builder
- [Factory Method](#factory-method)
- Singleton
- Pooling

## Structural patterns

- Bridge
- Composite
- Decorator
- Facade
- Flyweight
- Proxy

## Behavioral patterns

- Chain of responsability
- Command
- Mediator
- Observer
- Strategy
- Memento

## Messaging patterns

- [FanIn-FanOut](#fanin---fanout)

## Idioms

- [Functional pattern](#functional-pattern)

## Concurrency pattern

- [Worker pool](#worker-pool)

---

### Abstract factory

Create familiarity group of objects

```go
package main

import "fmt"

const (
  windowsGui int = iota
  macGui
)

type (
  event func()

  button struct{}

  buttonIface interface {
    onClick(event)
  }

  checkbox struct{}

  checkboxIface interface {
    onClick(event)
  }

  guiFactory interface {
    createButton() buttonIface
    createCheckbox() checkboxIface
  }

  winFactory struct{
    buttonIface
    checkboxIface
  }

  winFactoryIface interface {
    createButton() buttonIface
    createCheckbox() checkboxIface
  }

  macFactory struct{
    buttonIface
    checkboxIface
  }

  macFactoryIface interface {
    createButton() buttonIface
    createCheckbox() checkboxIface
  }

  application struct {
    button
    guiFactory
  }

  applicationIface interface {
    createUI(int) guiFactory
    paint()
  }
)

func (b *button) onClick(e event) {
  e()
  fmt.Println("button clicked")
}

func (c *checkbox) onClick(e event) {
  e()
  fmt.Println("checkbox clicked")
}

func (w *winFactory) createButton() buttonIface {
  return new(button)
}

func (w *winFactory) createCheckbox() checkboxIface {
  return new(checkbox)
}

func (m *macFactory) createButton() buttonIface {
  return new(button)
}

func (m *macFactory) createCheckbox() checkboxIface {
  return new(checkbox)
}

func newApplication() applicationIface {
  return new(application)
}

func (a *application) paint() {
  fmt.Println("paint application")
}

func (a *application) createUI(typeGui int) guiFactory {
  if typeGui == windowsGui {
    return &winFactory{}
  }

  if typeGui == macGui {
    return &macFactory{}
  }
  return nil
}

func main() {
  application := newApplication()

  windowsFactory := application.createUI(windowsGui)
  winButton := windowsFactory.createButton()
  winCheckbox := windowsFactory.createCheckbox()

  winButton.onClick(func() {fmt.Println("clicky")} )
  winCheckbox.onClick(func() {fmt.Println("selected")})
}

```

### Builder

```go
package main

import "fmt"

type (
  engine struct{}
  car    struct {
    seats        int
    engine       *engine
    tripComputer bool
    gps          bool
  }

  carBuilder struct {
    *car
  }
  carBuilderIface interface {
    reset()
    setSeats(int)
    setEngine(*engine)
    setTripComputer()
    setGPS()
    build() *car
  }

  directorIface interface {
    makeSUV() carBuilderIface
    makeSportsCar() carBuilderIface
  }
  director struct {
    carBuilder carBuilderIface
  }
)

func newDirector(carBuilder carBuilderIface) directorIface {
  return &director{carBuilder: carBuilder}
}

func newCarBuilder() carBuilderIface {
  return &carBuilder{car: &car{}}
}

func (d *director) makeSUV() carBuilderIface {
  d.carBuilder.setEngine(&engine{})
  d.carBuilder.setGPS()
  d.carBuilder.setSeats(4)
  d.carBuilder.setTripComputer()

  return d.carBuilder
}

func (d *director) makeSportsCar() carBuilderIface {
  d.carBuilder.setEngine(&engine{})
  d.carBuilder.setGPS()
  d.carBuilder.setSeats(2)
  d.carBuilder.setTripComputer()

  return d.carBuilder
}

func (c *car) build() *car {
  return c
}

func (b *carBuilder) reset() {
  b.engine = nil
  b.seats = 0
  b.gps = false
  b.tripComputer = false
}

func (b *carBuilder) setSeats(seats int) {
  b.seats = seats
}

func (b *carBuilder) setEngine(engine *engine) {
  b.engine = engine
}

func (b *carBuilder) setTripComputer() {
  b.tripComputer = true
}

func (b *carBuilder) setGPS() {
  b.gps = true
}

func (b *carBuilder) build() *car {
  return b.car
}

func (c *car) engineOn() {
  fmt.Println("start engine")
}

func main() {
  director := newDirector(newCarBuilder())
  carBuilder := director.makeSportsCar()
  car := carBuilder.build()
  car.engineOn()
}
```


### Factory method

Creating objects with a base class or interface. The object created is an interface type

```go
package main

import "fmt"

const (
  windowsDialogType int = iota
  webDialogType
)

type (
  event func()

  button interface {
    onClick(...event)
    render()
  }

  windowsDialog struct{}
  webDialog struct{}

  dialog struct{}
  dialogIface interface {
    render()
    createButton(int) button
  }
)

func executeEventCommand(events ...event) {
  for _, ev := range events {
    ev()
  }
}

func (windows *windowsDialog) render() {
  fmt.Println("rendering windows dialog")
}

func (windows *windowsDialog) onClick(events ...event) {
  executeEventCommand(events...)
}

func (web *webDialog) render() {
  fmt.Println("rendering web dialog")
}

func (web *webDialog) onClick(events ...event) {
  executeEventCommand(events...)
}

func (d *dialog) createButton(typeButton int) button {
  if typeButton == windowsDialogType {
    return &windowsDialog{}
  }

  if typeButton == webDialogType {
    return &webDialog{}
  }

  return nil
}

func (d *dialog) render() {
  fmt.Println("rendering dialog")
}

func newDialog() dialogIface {
  return &dialog{}
}

func main() {
  dialog := newDialog()

  dialog.render()
  windowsButton := dialog.createButton(windowsDialogType)
  windowsButton.render()
  windowsButton.onClick(func() {
    fmt.Println("clicked windows close")
  })

  htmlButton := dialog.createButton(webDialogType)
  htmlButton.render()
  htmlButton.onClick(func() {
    fmt.Println("clicked html close")
  })
}
```


### Worker pool

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

<button><a href="#top">Back to top</a></button>

### FanIn - FanOut

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

<button><a href="#top">Back to top</a></button>

### Pooling

This pattern is used in database libraries when it needs to create the connection and maintain the resource until its released and get back to the "pool" of resources.

In Golang there is a `sync.Pool()` function but it releases randomly the elements inside of the pool.

Let's take an example. How can we administrate the tables of a restaurant when clients go in or book them.

I have decided to use `channels` but it can be implemented with `slices` too.

```go
type (
  Table struct {
    Seats int
    ID    int
  }

  fnFactory func() (TablePoolIface, error)

  TablePoolIface interface {
    GetID() int
    Close()
    ServingFood()
    Cleaning()
  }

  TablePool struct {
    mtx      sync.Mutex

    closed    bool
    resources chan TablePoolIface
    factory   fnFactory
  }

  PoolIface interface {
    Release(TablePoolIface)
    Close()
    Acquire() (TablePoolIface, error)
  }
)

var (
  ErrPoolEmpty  = errors.New("Empty table pool")
  ErrPoolClosed = errors.New("The pool has been closed")
)

func (t *Table) GetID() int {
  return t.ID
}

func (t *Table) Close() {
  fmt.Printf("Closed table id %d\n", t.ID)
}

func (t *Table) ServingFood() {
  fmt.Printf("Serving food in table id %d\n", t.ID)
  time.Sleep(4 * time.Second)
  fmt.Printf("Finished food in table id %d\n", t.ID)
}

func (t *Table) Cleaning() {
  fmt.Printf("Cleaning table id %d\n", t.ID)
}

func poolingFactory() (TablePoolIface, error) {
  return &Table{}, nil
}

func New(fn fnFactory, tablePools []TablePoolIface) (PoolIface, error) {
  if len(tablePools) == 0 {
    return nil, ErrPoolEmpty
  }

  resources := make(chan TablePoolIface, len(tablePools))
  for _, t := range tablePools {
    resources <- t
  }

  return &TablePool{
    factory:   fn,
    resources: resources,
    mtx:       sync.Mutex{},
  }, nil
}

func (p *TablePool) Acquire() (TablePoolIface, error) {
  select {
  case t, ok := <-p.resources:
    if !ok {
      return nil, ErrPoolClosed
    }
    return t, nil
  default:
    return p.factory()
  }
}

func (p *TablePool) Release(table TablePoolIface) {
  p.mtx.Lock()
  defer p.mtx.Unlock()

  if p.closed {
    p.Close()
    return
  }

  select {
  case p.resources <- table:
  default:
    table.Close()
  }
}

func (p *TablePool) Close() {
  p.mtx.Lock()
  defer p.mtx.Unlock()

  if p.closed {
    return
  }

  close(p.resources)

  for table := range p.resources {
    table.Close()
  }
}

func main() {
  tablesRestaurant := make([]TablePoolIface, 0)
  for i := 0; i < 4; i++ {
    table := &Table{Seats: 4, ID: i + 1}
    tablesRestaurant = append(tablesRestaurant, table)
  }

  pool, err := New(poolingFactory, tablesRestaurant)
  if err != nil {
    panic(err)
  }

  bookedTable, err := pool.Acquire()
  if err != nil {
    panic(err)
  }

  bookedTable.ServingFood()
  bookedTable.Cleaning()

  moreTables := make([]TablePoolIface, 0)
  for range []int{2, 3, 4, 5} {
    t, err := pool.Acquire()
    if err != nil {
      fmt.Println(err)
      break
    }
    moreTables = append(moreTables, t)
  }

  pool.Release(bookedTable)

  pool.Close()
}
```

<button><a href="#top">Back to top</a></button>

## Functional pattern

When we want to build an object with optional parameters. We have two versions:
From [Uber code style](https://github.com/uber-go/guide/blob/master/style.md#functional-options)

#### Using currying functions

```go
type (
  FullName struct {
    name    string
    surName string
  }

  Option func(*FullName)
)

// Functional pattern
func WithName(name string) Option {
  return func(parameter *FullName) {
    parameter.name = name
  }
}

func WithSurName(surName string) Option {
  return func(parameter *FullName) {
    parameter.surName = surName
  }
}

func defaultFullName() *FullName {
  return &FullName{
    name:    "",
    surName: "",
  }
}

func NewFullName(opts ...Option) *FullName {
  fn := defaultFullName()

  for _, opt := range opts {
    opt(fn)
  }

  return fn
}
```

#### Using types and an interface for better testing

```go
type (
  FullName struct {
    name    string
    surName string
  }

  Option interface {
    apply(*FullName)
  }

  nameOption    string
  surNameOption string
)

func (n nameOption) apply(opts *FullName) {
  opts.name = string(n)
}

func WithName(name string) Option {
  return nameOption(name)
}

func (sn surNameOption) apply(opts *FullName) {
  opts.surName = string(sn)
}

func WithSurName(surName string) Option {
  return surNameOption(surName)
}

func defaultFullName() *FullName {
  return &FullName{}
}

func NewFullName(opts ...Option) *FullName {
  fn := defaultFullName()

  for _, opt := range opts {
    opt.apply(fn)
  }

  return fn
}
```

<button><a href="#top">Back to top</a></button>