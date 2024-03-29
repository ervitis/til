# Design patterns

## Creational patterns

- [Abstract factory](#abstract-factory)
- [Builder](#builder)
- [Factory Method](#factory-method)
- [Prototype](#prototype)
- [Singleton](#singleton)
- [Pooling](#pooling)

## Structural patterns

- [Adapter](#adapter)
- [Bridge](#bridge)
- [Composite](#composite)
- [Decorator](#decorator)
- [Facade](#facade)
- [Flyweight](#flyweight)
- [Proxy](#proxy)

## Behavioral patterns

- [Chain of responsibility](#chain-of-responsibility)
- [Command](#command)
- [Iterator](#iterator)
- [Mediator](#mediator)
- [Memento](#memento)
- [Observer](#observer)
- [Strategy](#strategy)
- [PubSub]()

## Concurrency patterns

- [FanIn-FanOut](#fanin---fanout)
- [Or channel](#or-channel)
- [HealthChecks with channels](#healthchecks-with-channels)

## Idioms

- [Functional pattern](#functional-pattern)

## Concurrency pattern

- [Worker pool](#worker-pool)

---

## Creational patterns

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

<button><a href="#top">Back to top</a></button>

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

<button><a href="#top">Back to top</a></button>

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

### Singleton

Single instance of an object used globally

```go
type (
  database struct {
    isConnected bool
  }

  databaseIface interface {
    connect()
  }
)

var (
  db databaseIface
)

func newDatabase() databaseIface {
  return &database{isConnected: false}
}

func (db *database) connect() {
  db.isConnected = true
}

func init() {
  db = newDatabase()
}

func main() {
  db.connect()
  fmt.Println("database is connected")
}
```

<button><a href="#top">Back to top</a></button>

### Prototype

Creating objects using a copy of them

```go
const (
  black colorEnum = iota
  blue
  red
  yellow

  blackName  string = "black"
  blueName   string = "blue"
  redName    string = "red"
  yellowName string = "yellow"
)

type (
  colorEnum int
  color     map[colorEnum]string

  shape struct {
    x, y  int
    color colorEnum
  }

  shapeIface interface {
    clone() shapeIface
    draw()
  }

  rectangle struct {
    shape
  }

  circle struct {
    shape
  }
)

var (
  colors color
)

func init() {
  colors = color{
    black:  blackName,
    blue:   blueName,
    red:    redName,
    yellow: yellowName,
  }
}

func (c *color) getKey(value string) colorEnum {
  for k, v := range colors {
    if v == value {
      return k
    }
  }
  return -1
}

func (c *color) getValue(key colorEnum) string {
  if v, exists := colors[key]; !exists {
    return ""
  } else {
    return v
  }
}

func newCircle(color string, x, y int) *circle {
  return &circle{shape{color: colors.getKey(color), x: x, y: y}}
}

func newRectangle(color string, x, y int) *rectangle {
  return &rectangle{shape{color: colors.getKey(color), x: x, y: y}}
}

func (r *rectangle) clone() shapeIface {
  return &rectangle{shape{
    x:     r.x,
    y:     r.y,
    color: r.color,
  }}
}

func (r *rectangle) draw() {
  fmt.Printf("draw rectangle %d %d in %s\n", r.x, r.y, colors.getValue(r.color))
}

func (c *circle) clone() shapeIface {
  return &circle{shape{
    x:     c.x,
    y:     c.y,
    color: c.color,
  }}
}

func (c *circle) draw() {
  fmt.Printf("draw circle %d %d in %s\r\n", c.x, c.y, colors.getValue(c.color))
}

func main() {
  myCircle := newCircle(blackName, 4, 5)
  myCircle.draw()
  copyOfCircle := myCircle.clone()
  copyOfCircle.draw()

  myRectangle := newRectangle(blueName, 2, 3)
  myRectangle.draw()
  copyOfRectangle := myRectangle.clone()
  copyOfRectangle.draw()
}
```

<button><a href="#top">Back to top</a></button>

### Pooling

This pattern is used in database libraries when it needs to create the connection and maintain the resource until its released and get back to the "pool" of resources.

In Golang there is a `sync.Pool()` function, but it releases randomly the elements inside the pool.

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


## Structural patterns

### Adapter

Integrate different types of implementation using a common API. For example, we have two types of payment proccesses: paypal and bank account. Each one has different implementation but has a common method: `pay`.

The client is the shopping cart where it calls the `checkout` function to process the payment. The adaptee we are using is the paypal in this case.

```go
type (
  money struct {
    amount   float64
    currency string
  }

  payment struct {
    apiKey string
  }
  paymentIface interface {
    pay(string, string, float64)
  }

  account struct {
    owner, email, currency string
    balance                float64
  }

  transaction struct {
    from   *account
    to     *account
    amount float64
    date   time.Time
    reason string
  }

  gateway struct {
    token    string
    accounts []*account
  }

  item struct {
    name  string
    price float64
  }

  shoppingCart struct {
    items            []*item
    paymentMethod    paymentIface
    shopEmailAddress string
  }

  bankAdapter struct {
    gateway *gateway
  }

  payPalAdapter struct {
    paymentMethod *payment
  }
)

func (p *payment) sendMoneyTo(sender, receipt string, money *money) {
  fmt.Printf("send %f %s from %s to %s\n", money.amount, money.currency, sender, receipt)
}

func (g *gateway) findAccountByEmail(email string) (*account, error) {
  for _, account := range g.accounts {
    if account.email == email {
      return account, nil
    }
  }
  return nil, errors.New("no account found")
}

func (g *gateway) processTransaction(tx *transaction) {
  fmt.Printf("transfered %f %s from %s to %s at %v\n", tx.amount, tx.from.currency, tx.from.owner, tx.to.owner, tx.date)
  tx.from.balance -= tx.amount
}

func (sc *shoppingCart) checkout(payeeEmail string) {
  var total float64

  for _, item := range sc.items {
    total += item.price
  }

  sc.paymentMethod.pay(payeeEmail, sc.shopEmailAddress, total)
}

func (b *bankAdapter) pay(fromEmail, toEmail string, amount float64) {
  fromAccount, err := b.gateway.findAccountByEmail(fromEmail)
  if err != nil {
    fmt.Printf("error %v\n", err)
    return
  }

  toAccount, err := b.gateway.findAccountByEmail(toEmail)
  if err != nil {
    fmt.Printf("error %v\n", err)
    return
  }

  tx := &transaction{
    from:   fromAccount,
    to:     toAccount,
    amount: amount,
    date:   time.Now(),
    reason: "payment to store",
  }
  b.gateway.processTransaction(tx)
}

func (p *payPalAdapter) pay(fromEmail, toEmail string, amount float64) {
  p.paymentMethod.sendMoneyTo(fromEmail, toEmail, &money{
    amount:   amount,
    currency: "EUR",
  })
}

func main() {
  items := []*item{
    {
      name:  "jam",
      price: 34,
    },
    {
      name:  "peanuts",
      price: 5,
    },
  }
  cart := &shoppingCart{
    items:            items,
    paymentMethod:    new(payPalAdapter),
    shopEmailAddress: "example@example.com",
  }

  cart.checkout("shop@amazon.com")
}
```

<button><a href="#top">Back to top</a></button>

### Bridge

When we want to extend a class of other not using inheritance but composition.

```go
type (
    channel uint16
    volume  float64
    
    device struct {
        enabled bool
        volume  volume
        channel channel
    }
    deviceIface interface {
        isEnabled() bool
        enable()
        disable()
        getVolume() volume
        setVolume(volume)
        getChannel() channel
        setChannel(channel)
    }
    
    remote struct {
        device deviceIface
    }
    remoteIface interface {
        togglePower()
        volumeDown()
        volumeUp()
        channelDown()
        channelUp()
    }
    
    advancedRemote struct {
        *remote
    }
    advancedRemoteIface interface {
        mute()
    }
)

func (d *device) isEnabled() bool {
    return d.enabled
}

func (d *device) enable() {
    d.enabled = true
}

func (d *device) disable() {
    d.enabled = false
}

func (d *device) getVolume() volume {
    return d.volume
}

func (d *device) setVolume(v volume) {
    d.volume = v
}

func (d *device) getChannel() channel {
    return d.channel
}

func (d *device) setChannel(c channel) {
    d.channel = c
}

func newDevice() *device {
    return &device{}
}

func newRemote(device deviceIface) *remote {
    return &remote{
        device: device,
    }
}

func newAdvanceRemote(device deviceIface) *advancedRemote {
    ar := &advancedRemote{
        remote: newRemote(device),
    }
    return ar
}

func (r *remote) togglePower() {
    if r.device.isEnabled() {
        r.device.disable()
    } else {
        r.device.enable()
    }
    
    fmt.Printf("toggle power %v to device\n", r.device.isEnabled())
}

func (r *remote) volumeDown() {
    fmt.Printf("sending volume down (%f) to device\n", r.device.getVolume())
    if r.device.getVolume() == 0 {
        return
    }
    r.device.setVolume(r.device.getVolume() - 1)
}

func (r *remote) volumeUp() {
    fmt.Printf("sending volume up (%f) to device\n", r.device.getVolume())
    if r.device.getVolume() == 10 {
        return
    }
    r.device.setVolume(r.device.getVolume() + 1)
}

func (r *remote) channelDown() {
    fmt.Printf("sending channel down to device (%d)\n", r.device.getChannel())
    if r.device.getChannel() == 0 {
        r.device.setChannel(10)
    }
    r.device.setChannel(r.device.getChannel() - 1)
}

func (r *remote) channelUp() {
    fmt.Printf("sending channel up to device (%d)\n", r.device.getChannel())
    if r.device.getChannel() == 10 {
        r.device.setChannel(0)
    }
    r.device.setChannel(r.device.getChannel() + 1)
}

func (ar *advancedRemote) mute() {
    ar.device.setVolume(0)
}

func main() {
    radio := newDevice()
    tv := newDevice()
    
    radioRemote := newRemote(radio)
    tvRemote := newAdvanceRemote(tv)
    
    radioRemote.togglePower()
    radioRemote.channelDown()
    radioRemote.volumeUp()
    
    tvRemote.togglePower()
    tvRemote.volumeUp()
    tvRemote.mute()
}
```

<button><a href="#top">Back to top</a></button>

### Composite

Wrap inside an interface common operations that will be implemented by other components

```go
import (
    "fmt"
    "github.com/google/uuid"
)

type (
    coordinate float64
    
    graphicIface interface {
        move(coordinate, coordinate)
        draw()
        getID() int
    }
    
    dot struct {
        ID   int
        x, y coordinate
    }
    
    circle struct {
        radius float64
        *dot
    }
    
    compoundGraphic struct {
        children []graphicIface
    }
    compoundGraphicIface interface {
        add(graphicIface)
        remove(graphicIface)
    }
)

var (
    uidGenerator uuid.UUID
)

func newDot(x, y coordinate) graphicIface {
    uidGenerator = uuid.New()
    return &dot{ID: int(uidGenerator.ID()), x: x, y: y}
}

func newCircle(radius float64, x, y coordinate) graphicIface {
    uidGenerator = uuid.New()
    return &circle{radius: radius, dot: &dot{ID: int(uidGenerator.ID()), x: x, y: y}}
}

func newCompoundGraphic() *compoundGraphic {
    return &compoundGraphic{children: make([]graphicIface, 0)}
}

func (d *dot) move(x, y coordinate) {
    fmt.Printf("moving to %f,%f\n", x, y)
    d.x = x
    d.y = y
}

func (d *dot) draw() {
    fmt.Printf("dot at %f,%f\n", d.x, d.y)
}

func (d *dot) getID() int {
    return d.ID
}

func (c *circle) draw() {
    fmt.Printf("circle at %f,%f with radius %f\n", c.x, c.y, c.radius)
}

func (c *circle) getID() int {
    return c.ID
}

func (cg *compoundGraphic) add(child graphicIface) {
    cg.children = append(cg.children, child)
}

func (cg *compoundGraphic) remove(child graphicIface) {
    for k, v := range cg.children {
        if child.getID() == v.getID() {
            cg.children = append(cg.children[:k], cg.children[k+1:]...)
        }
    }
}

func (cg *compoundGraphic) move(x, y coordinate) {
    fmt.Printf("moving to %f,%f\n", x, y)
    for _, v := range cg.children {
        v.move(x, y)
    }
}

func (cg *compoundGraphic) draw() {
    for _, v := range cg.children {
        v.draw()
    }
}

func main() {
    dot1 := newDot(4, 2)
    dot2 := newDot(19, 11)
    circle1 := newCircle(2, 10, 5)
    
    dot1.draw()
    dot1.move(2, 4)
    cg := newCompoundGraphic()
    cg.add(dot1)
    cg.add(dot2)
    cg.add(circle1)
    cg.move(1, 8)
    cg.draw()
    
    cg.remove(dot1)
    cg.draw()
}
```

<button><a href="#top">Back to top</a></button>

### Decorator

When we want to add more functionality adding or transforming the input data into something else.

```go
type (
	dataSourceIface interface {
		writeData([]byte)
		readData() []byte
		close()
	}

	fileDataSource struct {
		f        *os.File
		fileName string
	}
	fileDataSourceIface interface {
		dataSourceIface
	}

	dataSourceDecorator struct {
		wrappee dataSourceIface
	}

	encryptionDecorator struct {
		*dataSourceDecorator
	}
)

func newFileDataSource(fileName string) *fileDataSource {
	return &fileDataSource{fileName: fileName}
}

func (f *fileDataSource) open() {
	var err error

	_, err = os.Stat(f.fileName)
	if os.IsNotExist(err) {
		if f.f, err = os.Create(f.fileName); err != nil {
			panic(err)
		} else {
			return
		}
	}

	f.f, err = os.OpenFile(f.fileName, os.O_RDWR, 0660)
	if err != nil {
		panic(err)
	}
}

func (f *fileDataSource) close() {
	if err := f.f.Close(); err != nil {
		panic(err)
	}
}

func (f *fileDataSource) writeData(data []byte) {
	f.open()

	if _, err := f.f.Write(data); err != nil {
		panic(err)
	}
	if err := f.f.Close(); err != nil {
		panic(err)
	}
}

func (f *fileDataSource) readData() []byte {
	f.open()

	stat, _ := f.f.Stat()
	data := make([]byte, stat.Size())
	if _, err := f.f.Read(data); err != nil {
		panic(err)
	}
	defer f.f.Close()
	return data
}

func newDataSourceDecorator(wrappee dataSourceIface) *dataSourceDecorator {
	return &dataSourceDecorator{wrappee: wrappee}
}

func newEncryptionDecorator(wrappee dataSourceIface) *encryptionDecorator {
	return &encryptionDecorator{newDataSourceDecorator(wrappee)}
}

func (d *dataSourceDecorator) writeData(data []byte) {
	d.wrappee.writeData(data)
}

func (d *dataSourceDecorator) readData() []byte {
	return d.wrappee.readData()
}

func (ed *encryptionDecorator) writeData(data []byte) {
	enc := make([]byte, base64.StdEncoding.EncodedLen(len(data)))
	base64.StdEncoding.Encode(enc, data)
	ed.wrappee.writeData(enc)
}

func (ed *encryptionDecorator) readData() []byte {
	data := ed.wrappee.readData()
	dec := make([]byte, base64.StdEncoding.DecodedLen(len(data)))
	if _, err := base64.StdEncoding.Decode(dec, data); err != nil {
		panic(err)
	}
	return dec
}

func main() {
	encSource := newEncryptionDecorator(newFileDataSource("example.dat"))
	encSource.writeData([]byte(`this is encrypted`))
	readData := string(encSource.readData())
	fmt.Println(readData)
}
```

<button><a href="#top">Back to top</a></button>

### Facade

When we want to abstract a complex implementation

```go
type (
	account struct {
		name string
	}

	walletFacade struct {
		account      *account
		wallet       *wallet
		securityCode *securityCode
		notification *notification
		ledger       *ledger
	}

	securityCode struct {
		code int
	}

	wallet struct {
		balance int
	}

	ledger struct{}

	notification struct{}
)

func newSecurityCode(code int) *securityCode {
	return &securityCode{
		code: code,
	}
}

func (s *securityCode) checkCode(incomingCode int) error {
	if s.code != incomingCode {
		return fmt.Errorf("Security Code is incorrect")
	}
	fmt.Println("SecurityCode Verified")
	return nil
}

func newAccount(accountName string) *account {
	return &account{
		name: accountName,
	}
}

func (a *account) checkAccount(accountName string) error {
	if a.name != accountName {
		return fmt.Errorf("Account Name is incorrect")
	}
	fmt.Println("Account Verified")
	return nil
}

func newWallet() *wallet {
	return &wallet{
		balance: 0,
	}
}

func (w *wallet) creditBalance(amount int) {
	w.balance += amount
	fmt.Println("Wallet balance added successfully")
	return
}

func (w *wallet) debitBalance(amount int) error {
	if w.balance < amount {
		return fmt.Errorf("Balance is not sufficient")
	}
	fmt.Println("Wallet balance is Sufficient")
	w.balance = w.balance - amount
	return nil
}

func (s *ledger) makeEntry(accountID, txnType string, amount int) {
	fmt.Printf("Make ledger entry for accountId %s with txnType %s for amount %d", accountID, txnType, amount)
	return
}

func (n *notification) sendWalletCreditNotification() {
	fmt.Println("Sending wallet credit notification")
}

func (n *notification) sendWalletDebitNotification() {
	fmt.Println("Sending wallet debit notification")
}

func newWalletFacade(accountID string, code int) *walletFacade {
	fmt.Println("Starting create account")
	walletFacacde := &walletFacade{
		account:      newAccount(accountID),
		securityCode: newSecurityCode(code),
		wallet:       newWallet(),
		notification: &notification{},
		ledger:       &ledger{},
	}
	fmt.Println("Account created")
	return walletFacacde
}

func (w *walletFacade) addMoneyToWallet(accountID string, securityCode int, amount int) error {
	fmt.Println("Starting add money to wallet")
	err := w.account.checkAccount(accountID)
	if err != nil {
		return err
	}
	err = w.securityCode.checkCode(securityCode)
	if err != nil {
		return err
	}
	w.wallet.creditBalance(amount)
	w.notification.sendWalletCreditNotification()
	w.ledger.makeEntry(accountID, "credit", amount)
	return nil
}

func (w *walletFacade) deductMoneyFromWallet(accountID string, securityCode int, amount int) error {
	fmt.Println("Starting debit money from wallet")
	err := w.account.checkAccount(accountID)
	if err != nil {
		return err
	}
	err = w.securityCode.checkCode(securityCode)
	if err != nil {
		return err
	}
	err = w.wallet.debitBalance(amount)
	if err != nil {
		return err
	}
	w.notification.sendWalletDebitNotification()
	w.ledger.makeEntry(accountID, "credit", amount)
	return nil
}

func main() {
	walletFacade := newWalletFacade("test", 1)
	if err := walletFacade.addMoneyToWallet("test", 1, 10); err != nil {
		panic(err)
	}

	if err := walletFacade.deductMoneyFromWallet("test", 1, 3); err != nil {
		panic(err)
	}
}
```

<button><a href="#top">Back to top</a></button>

### Flyweight

Separating some components to reduce memory usage sharing the objects. It can combine with the pool pattern.

```go
type (
	treeType struct {
		name, color, texture string
	}

	forest struct {
		trees []*tree
	}

	tree struct {
		x, y int
		treeType *treeType
	}
)

var (
	trees []treeType
)

func (*treeType) draw(canvas, x, y int) {
	fmt.Printf("drawing canvas %v at (%d,%d)\n", canvas, x, y)
}

func (t *tree) draw(canvas int) {
	t.treeType.draw(canvas, t.x, t.y)
}

func (f *forest) plantTree(x, y int, name, color, texture string) {
	tf := getTreeType(name, color, texture)
	t := &tree{
		x:        x,
		y:        y,
		treeType: tf,
	}
	f.trees = append(f.trees, t)
}

func (f *forest) draw(canvas int) {
	for _, t := range f.trees {
		t.draw(canvas)
	}
}

func newForest() *forest {
	return &forest{}
}

func getTreeType(name, color, texture string) *treeType {
	for _, tree := range trees {
		if tree.name == name && tree.color == color && tree.texture == texture {
			return &tree
		}
	}
	t := treeType{
		name:    name,
		color:   color,
		texture: texture,
	}
	trees = append(trees, t)
	return &t
}

func main() {
	forestTrees := newForest()
	forestTrees.plantTree(1, 1, "mori", "green", "texture1.jpg")
	forestTrees.plantTree(2, 3, "moriawase", "green", "texture2.jpg")
	forestTrees.plantTree(2, 3, "mori", "green", "texture1.jpg")
	forestTrees.draw(1)
}
```

<button><a href="#top">Back to top</a></button>

### Proxy

When we want to use a cache service that implements the same interface as another service.

```go
type (
	video struct {
		name string
		ID int
	}

	youtubeIface interface {
		listVideos() []video
		getVideoInfo(int) *video
		downloadVideo(int)
	}

	youtube struct{}

	youtubeManager struct {
		service youtubeIface
	}

	cachedYoutube struct{
		service youtubeIface
		videoCache *video
		listVideoCache *[]video
	}
)

var (
	youtubePlatform = map[int]video{
		1: {name: "hello.mp4", ID: 1},
		2: {name: "bye.mp4", ID: 2},
	}
)

func (y *youtube) listVideos() []video {
	var videos []video
	for _, v := range youtubePlatform {
		videos = append(videos, v)
	}
	return videos
}

func (y *youtube) getVideoInfo(ID int) *video {
	if v, exists := youtubePlatform[ID]; !exists {
		return nil
	} else {
		return &v
	}
}

func (y *youtube) downloadVideo(ID int) {
	fmt.Printf("Downloading video %d\n", ID)
}

func (cy *cachedYoutube) listVideos() []video {
	if cy.listVideoCache != nil {
		return *cy.listVideoCache
	}
	l := cy.service.listVideos()
	cy.listVideoCache = &l
	return l
}

func (cy *cachedYoutube) downloadVideo(ID int) {
	if cy.videoCache != nil && cy.videoCache.ID == ID {
		fmt.Printf("Downloading cached video\n")
	}
	cy.service.downloadVideo(ID)
}

func (cy *cachedYoutube) getVideoInfo(ID int) *video {
	if cy.videoCache != nil && cy.videoCache.ID == ID {
		return cy.videoCache
	}
	cy.videoCache = cy.service.getVideoInfo(ID)
	return cy.videoCache
}

func newYoutubeManager(service youtubeIface) *youtubeManager {
	return &youtubeManager{service: service}
}

func newYoutubeService() youtubeIface {
	return &youtube{}
}

func newYoutubeCachedService(service youtubeIface) youtubeIface {
	return &cachedYoutube{service: service}
}

func (ym *youtubeManager) renderVideoPage(ID int) {
	info := ym.service.getVideoInfo(ID)
	fmt.Printf("render video page video %#v", info)
}

func (ym *youtubeManager) renderListPanel() {
	list := ym.service.listVideos()
	fmt.Println(list)
	fmt.Println("Finish rendering")
}

func (ym *youtubeManager) reactOnUserInput(ID int) {
	ym.renderVideoPage(ID)
	ym.renderListPanel()
}

func main() {
	proxy := &cachedYoutube{service: newYoutubeService()}
	manager := newYoutubeManager(proxy)
	manager.reactOnUserInput(1)
}
```

<button><a href="#top">Back to top</a></button>

## Behavioral patterns

### Chain of responsibility

Some objects share the same interface, and they are being executed one after one.

```go
type (
	patient struct {
		name string
		doctorCheck, registrationDone, medicineReceived, paymentDone bool
	}

	reception struct {
		next department
	}

	doctor struct {
		next department
	}

	medical struct {
		next department
	}

	cashier struct {
		next department
	}

	department interface {
		execute(*patient)
		setNext(department)
	}
)

func (c *cashier) execute(p *patient) {
	if p.paymentDone {
		fmt.Println("payment done")
		return
	}
	fmt.Println("doing payment...")
	p.paymentDone = true
	if c.next != nil {
		c.next.execute(p)
	}
}

func (c *cashier) setNext(next department) {
	c.next = next
}

func (m *medical) execute(p *patient) {
	if p.medicineReceived {
		fmt.Println("medicine received")
		return
	}
	fmt.Println("receiving medicine")
	p.medicineReceived = true
	if m.next != nil {
		m.next.execute(p)
	}
}

func (m *medical) setNext(next department) {
	m.next = next
}

func (d *doctor) execute(p *patient) {
	if p.doctorCheck {
		fmt.Println("doctor checked")
	}
	fmt.Println("doctor checking patient")
	p.doctorCheck = true
	if d.next != nil {
		d.next.execute(p)
	}
}

func (d *doctor) setNext(next department) {
	d.next = next
}

func (r *reception) execute(p *patient) {
	if p.registrationDone {
		fmt.Println("patient registered")
	}
	fmt.Println("registering patient")
	p.registrationDone = true
	if r.next != nil {
		r.next.execute(p)
	}
}

func (r *reception) setNext(next department) {
	r.next = next
}

func main() {
	receptionist := &reception{}
	doctor := &doctor{}
	medical := &medical{}
	cashier := &cashier{}

	receptionist.setNext(doctor)
	doctor.setNext(medical)
	medical.setNext(cashier)

	patient1 := &patient{name: "victor"}
	receptionist.execute(patient1)
}
```

<button><a href="#top">Back to top</a></button>

### Command

From [this blog post](https://www.sohamkamani.com/golang/command-pattern/) this pattern decouples the business logic in N commands doing a single command

```go
type (
	commandIface interface {
		execute()
	}

	restaurant struct {
		totalDishes   int
		cleanedDishes int
	}

	makePizzaCommand struct {
		n int
		*restaurant
	}

	makeSaladCommand struct {
		n int
		*restaurant
	}

	cleanDishesCommand struct {
		*restaurant
	}

	cook struct {
		commands []commandIface
	}
)

func (c *cleanDishesCommand) execute() {
	c.restaurant.cleanedDishes = c.restaurant.totalDishes
	fmt.Println("dishes cleaned")
}

func (c *makeSaladCommand) execute() {
	c.restaurant.cleanedDishes -= c.n
	fmt.Printf("made %d salads\n", c.n)
}

func (c *makePizzaCommand) execute() {
	c.restaurant.cleanedDishes -= c.n
	fmt.Printf("made %d pizzas\n", c.n)
}

func newRestaurant() *restaurant {
	return &restaurant{
		totalDishes:   10,
		cleanedDishes: 10,
	}
}

func (r *restaurant) makePizza(n int) commandIface {
	return &makePizzaCommand{
		n:          n,
		restaurant: r,
	}
}

func (r *restaurant) makeSalad(n int) commandIface {
	return &makeSaladCommand{
		n:          n,
		restaurant: r,
	}
}

func (r *restaurant) cleanDishes() commandIface {
	return &cleanDishesCommand{
		restaurant: r,
	}
}

func (c *cook) executeCommands() {
	for _, c := range c.commands {
		c.execute()
	}
}

func main() {
	r := newRestaurant()

	tasks := []commandIface{
		r.makeSalad(2),
		r.makePizza(1),
		r.makeSalad(3),
		r.cleanDishes(),
		r.makePizza(5),
		r.cleanDishes(),
	}

	cooks := []*cook{{}, {}}

	for i, task := range tasks {
		cook := cooks[i%len(cooks)]
		cook.commands = append(cook.commands, task)
	}

	for i, cook := range cooks {
		fmt.Printf("cook %d:\n", i)
		cook.executeCommands()
	}
}
```

<button><a href="#top">Back to top</a></button>

### Iterator

Traverse elements of a collection without exposing its underlying representation.

```go
type (
	email struct {
		subject, content, from, to string
	}

	emailCollection struct {
		emails []*email
	}

	emailIteratorIface interface {
		getNext() *email
		hasMore() bool
	}

	emailIterator struct {
		index  int
		emails []*email
	}
)

func newEmailCollection(emails ...*email) *emailCollection {
	return &emailCollection{emails: emails}
}

func (ec *emailCollection) createIterator() emailIteratorIface {
	return &emailIterator{emails: ec.emails}
}

func (ei *emailIterator) getNext() *email {
	if ei.hasMore() {
		email := ei.emails[ei.index]
		ei.index++
		return email
	}
	return nil
}

func (ei *emailIterator) hasMore() bool {
	return ei.index < len(ei.emails)
}

func main() {
	emailCollections := newEmailCollection([]*email{
		{from: "victor", to: "ana", content: "hello", subject: "hello"},
		{from: "victor2", to: "an3a", content: "hello2", subject: "hello2"},
		{from: "victor3", to: "an3a", content: "hello3", subject: "hello3"},
	}...)
	emailIte := emailCollections.createIterator()

	for emailIte.hasMore() {
		fmt.Println(emailIte.getNext())
	}
}
```

<button><a href="#top">Back to top</a></button>

### Mediator

Instead of doing between the classes all the communications, the mediator will do the work.

```go
type (
	stock struct {
		typeStock string
		price     float64
	}

	traderService struct {
		stocks []*stock
	}

	trader struct {
		service *traderService
		stockToPlace *stock
		interestedStocks []*stock
	}
)

func (t *traderService) getStockOptions(typeStock string) []*stock {
	sts := make([]*stock, 0)

	if typeStock == "buy" || typeStock == "sell" {
		for _, s := range t.stocks {
			if s.typeStock == typeStock {
				sts = append(sts, s)
			}
		}
	}

	return sts
}

func (t *traderService) pushStock(s *stock) {
	t.stocks = append(t.stocks, s)
}

func (t *trader) placeStock(stock *stock) {
	t.service.pushStock(stock)
}

func (t *trader) getStocks(stockType string) []*stock {
	return t.service.getStockOptions(stockType)
}

func newTraderService() *traderService {
	return &traderService{stocks: make([]*stock, 0)}
}

func newTrader(service *traderService) *trader {
	return &trader{service: service}
}

func main() {
	tradeService := newTraderService()

	traderBBVA := newTrader(tradeService)
	traderSantander := newTrader(tradeService)

	traderBBVA.placeStock(&stock{"buy", 2000})
	traderBBVA.placeStock(&stock{"sell", 20})
	traderBBVA.placeStock(&stock{"sell", 25})
	traderSantander.placeStock(&stock{"buy", 20560})
	traderSantander.placeStock(&stock{"sell", 203})

	fmt.Println(traderBBVA.getStocks("buy"))
}
```

<button><a href="#top">Back to top</a></button>

### Memento

When we want to know the previous state of an object, save it or restore it.

```go
type (
	memento struct {
		state string
	}

	caretaker struct {
		mementoArray []*memento
	}

	originator struct {
		state string
	}
)

func (m *memento) getSavedState() string {
	return m.state
}

func (e *originator) createMemento() *memento {
	return &memento{state: e.state}
}

func (e *originator) restoreMemento(m *memento) {
	e.state = m.getSavedState()
}

func (e *originator) setState(state string) {
	e.state = state
}

func (e *originator) getState() string {
	return e.state
}

func (c *caretaker) addMemento(m *memento) {
	c.mementoArray = append(c.mementoArray, m)
}

func (c *caretaker) getMemento(index int) *memento {
	return c.mementoArray[index]
}

func newCaretaker() *caretaker {
	return &caretaker{mementoArray: make([]*memento, 0)}
}

func newOriginatorWithState(state string) *originator {
	return &originator{state: state}
}

func main() {
	caretaker := newCaretaker()

	originator := newOriginatorWithState("A")
	caretaker.addMemento(originator.createMemento())
	fmt.Printf("State %s\n", originator.getState())

	originator.setState("B")
	caretaker.addMemento(originator.createMemento())
	fmt.Printf("State %s\n", originator.getState())

	originator.setState("C")
	caretaker.addMemento(originator.createMemento())
	fmt.Printf("State %s\n", originator.getState())

	originator.restoreMemento(caretaker.getMemento(0))
	fmt.Printf("State %s\n", originator.getState())
}
```

<button><a href="#top">Back to top</a></button>

### Observer

The subscribe pattern.

```go
type (
	editor struct {
		events eventManagerIface
	}

	eventListenerIface interface {
		update(string)
	}

	eventManagerIface interface {
		subscribe(string, eventListenerIface)
		unsubscribe(string)
		notify(string, string)
	}

	eventManager struct {
		listeners map[string]eventListenerIface
	}

	emailAlertsListener struct {
		email string
	}
	loggingListener struct {
		fileName string
	}
)

func (l *emailAlertsListener) update(fileName string) {
	fmt.Println("updating filename using email alert ", fileName)
}

func (l *loggingListener) update(fileName string) {
	fmt.Println("updating filename in log ", fileName)
}

func (em *eventManager) subscribe(eventName string, event eventListenerIface) {
	em.listeners[eventName] = event
}

func (em *eventManager) unsubscribe(eventName string) {
	delete(em.listeners, eventName)
}

func (em *eventManager) notify(eventName, data string) {
	for k, v := range em.listeners {
		if k == eventName {
			v.update(data)
		}
	}
}

func newEventManager() *eventManager {
	return &eventManager{listeners: make(map[string]eventListenerIface)}
}

func newEditor() *editor {
	return &editor{events: newEventManager()}
}

func (e *editor) openFile(path string) {
	e.events.notify("open", path)
}

func (e *editor) saveFile(fileName string) {
	e.events.notify("save", fileName)
}

func newLoggingListener(fileName string) *loggingListener {
	return &loggingListener{fileName: fileName}
}

func newEmailAlertListener(email string) *emailAlertsListener {
	return &emailAlertsListener{email: email}
}

func main() {
	loggingListener := newLoggingListener("log.txt")
	emailAlertsListener := newEmailAlertListener("victor@example.com")

	editor := newEditor()
	editor.events.subscribe("open", loggingListener)
	editor.events.subscribe("save", emailAlertsListener)
	editor.openFile("temp.txt")
	editor.saveFile("temp.txt")
}
```

<button><a href="#top">Back to top</a></button>

### Strategy

Write algorithms into separate classes called strategies. The class named context stores the strategies and delegates the work into them.

```go
type (
	algIface interface {
		execute()
	}

	lru  struct{}
	lfu  struct{}
	fifo struct{}

	cache struct {
		storage     map[string]string
		alg         algIface
		capacity    int
		maxCapacity int
	}
)

func (l *lru) execute() {
	fmt.Println("using LRU")
}

func (l *lfu) execute() {
	fmt.Println("using LFU")
}

func (l *fifo) execute() {
	fmt.Println("using FIFO")
}

func initCache(e algIface) *cache {
	storage := make(map[string]string)
	return &cache{
		storage:     storage,
		alg:         e,
		capacity:    0,
		maxCapacity: 2,
	}
}

func (c *cache) setEvictionAlgo(e algIface) {
	c.alg = e
}

func (c *cache) add(key, value string) {
	if c.capacity == c.maxCapacity {
		c.evict()
	}
	c.capacity++
	c.storage[key] = value
}

func (c *cache) get(key string) string {
	v, ok := c.storage[key]
	if ok {
		delete(c.storage, key)
		return v
	}
	return ""
}

func (c *cache) evict() {
	c.alg.execute()
	c.capacity--
}

func main() {
	cache := initCache(&lru{})

	cache.add("key1", "hello")
	cache.add("key2", "bye")
	cache.evict()
	cache.add("key3", "hola")
	cache.setEvictionAlgo(&fifo{})
	cache.add("key4", "adios")
	fmt.Println(cache.get("key3"))
}
```

<button><a href="#top">Back to top</a></button>

---

### PubSub

```go
type (
	Message struct {
		Topic string
		Value []byte
	}

	Queue struct {
		Topics map[string]chan Message
	}
)

func NewQueue() *Queue {
	return &Queue{Topics: map[string]chan Message{}}
}

func (q *Queue) Subscribe(topic string, handler func(m *Message)) error {
	if _, exists := q.Topics[topic]; exists {
		return fmt.Errorf("topic %s not exists", topic)
	}

	q.Topics[topic] = make(chan Message)

	go func() {
		for {
			ch := <-q.Topics[topic]
			handler(&ch)
		}
	}()
	return nil
}

func (q *Queue) Publish(msg Message) error {
	if _, exists := q.Topics[msg.Topic]; !exists {
		return fmt.Errorf("topic %s not exists", msg.Topic)
	}

	q.Topics[msg.Topic] <- msg
	return nil
}

func main() {
	queue := NewQueue()

	_ = queue.Subscribe("test", func(m *Message) {
		fmt.Println(string(m.Value))
	})

	_ = queue.Publish(Message{Topic: "test", Value: []byte(`testing message pub sub`)})
}
```

<button><a href="#top">Back to top</a></button>

---

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


### or-channel

When we want to do in parallel a lot of tasks and wait for them to finish without knowing the return value, we can use this pattern.
From [Concurrency in Go: Tools and Techniques for Developers](https://www.amazon.es/Concurrency-Go-Katherine-Cox-Buday/dp/1491941197/ref=sr_1_12?__mk_es_ES=%C3%85M%C3%85%C5%BD%C3%95%C3%91&keywords=go+programming&qid=1577801642&sr=8-12) fixed a bug where the go routine may not be waited for the main program.

```go
var or func(channels ...<-chan interface{}) <-chan interface{}

func main() {
	or = func(channels ...<-chan interface{}) <-chan interface{} {
		switch len(channels) {
		case 0:
			return nil
		case 1:
			return channels[0]
		}

		orDone := make(chan interface{})
		go func() {
			defer close(orDone)
			switch len(channels) {
			case 2:
				select {
				case <-channels[0]:
				case <-channels[1]:
				default:
					select {
					case <-channels[0]:
					case <-channels[1]:
					case <-channels[2]:
					case <-or(append(channels[3:], orDone)...):
					}
				}
			}
		}()
		return orDone
	}

	sig := func(after time.Duration) <-chan interface{} {
		c := make(chan interface{})

		go func() {
			defer close(c)
			time.Sleep(after)
			fmt.Println("where am i")
			c <- "done"
		}()
		<- c
		return c
	}

	start := time.Now()
	<-or(sig(13*time.Second), sig(5*time.Second), sig(22*time.Second))
	fmt.Printf("done after %v\n", time.Since(start))
}
```

<button><a href="#top">Back to top</a></button>


### HealthChecks with channels

```go
func main() {
	doWork := func(done <-chan interface{}, pulseInterval time.Duration) (<-chan interface{}, <-chan time.Time) {
		heartbeat := make(chan interface{})
		results := make(chan time.Time)

		go func() {
			defer close(heartbeat)
			defer close(results)

			pulse := time.Tick(pulseInterval)
			workGen := time.Tick(2 * pulseInterval)

			sendPulse := func() {
				select {
				case heartbeat <- struct{}{}:
				default:
				}
			}

			sendResult := func(r time.Time) {
				for {
					select {
					case <-done:
						return
					case <-pulse:
						sendPulse()
					case results <- r:
						return
					}
				}
			}

			for {
				select {
				case <-done:
					return
				case <-pulse:
					sendPulse()
				case r := <-workGen:
					sendResult(r)
				}
			}
		}()
		return heartbeat, results
	}

	done := make(chan interface{})
	time.AfterFunc(10*time.Second, func() {
		close(done)
	})

	const timeout = 2 * time.Second
	heartbeat, results := doWork(done, timeout/2)

	for {
		select {
		case _, ok := <-heartbeat:
			if !ok {
				return
			}
			fmt.Printf("results %v\n", results)
		case <-time.After(timeout):
			return
		}
	}
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