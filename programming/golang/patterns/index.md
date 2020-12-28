# Patterns

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