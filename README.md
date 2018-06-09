**Table of Content**

- [Best Practices](#best-practices)
    - [Error-Handling in Custom Package](#error-handling-in-custom-package)
    - [Error-Handling Scheme with Closure](#error-handling-scheme-with-clouse)
    - [Testing](#testing)
    - [Channel](#channel)
    - [Code Snippets](#code-snippets)
    - [Advices](#advices)

- [Go Project with Makefile](#go-project-with-makefile)

<h1 id="best-practices">Best Practices</h1>
<h2 id="error-handing-in-custom-package">Error-Handling in Custom Package</h2>

-  *always recover from panic from your package*
-  *return errors to the caller of your package*

```go
package parse 
import (
    "fmt"
    "strings"
    "strconv"
)
type ParseError struct {
    Index int
    Word string
    Error err
}
func (e *ParseError) String() string {
    return fmt.Sprintf("pkg parse: error parse %q as int", e.Word)
}
func Parse(input string)(number []int, err error){
    defer func(){
        if r := recover(); r != nil {
            var ok bool
            err, ok = r.(error)
            if !ok {
                err = fmt.Errorf("pkg:%v", r)
            }
        }
    }()
    fields := strings.Fields(input)
    numbers = fields2numbers(fields)
    return
}
func field2numbers(fields []string)(number []int){
    if len(fields) == 0 {
        panic("no words to parse")
    }
    for idx, field := range fields {
        num, err := strconv.Atoi(field)
        if err!=nil{
            panic(&ParseError{idx, field, err})
        }
        numbers = append(numbers, num)
    }
    return
}
```

<h2 id="error-handing-schem-wtih-closures">Error-Handling Scheme with Closure</h2>

Suppose all functions have the signature:

```go
func f(a type1, b type2)
```

Scheme uses 2 helper functions:
-  `check` a function to test whether an error occurred.
```go
func check(err error){
    if err!=nil{
        panic(err)
    }
}
```
-  `errorhandler` a wrapper function.
```go
type fType1 func(a type1, b type2)
func errorHandler(fn fType1) fType1 {
    return func(a type1, b type2){
        defer func(){
            if e, ok := recover().(error); ok {
                log.Printf("run time panic: %v", err)
            }
        }()
        fn(a, b)
    }
}
```
- Custom function
```go
func f1(a type1, b type2){
    f, err := // call other function or method
    check(err)
    t, err := // call other function or method
    check(err)
}
```
- Usage
```go
func main(){
    errorHandler(f1)
    errorHandler(f2)
}
```

<h2 id="test">Testing</h2>

- Fail(): making test function failed
- FailNow(): making test function failed and stop execution.
- Log(args ... interface{}): log test
- Fatal(args ... interface{}): combined `FailNow` and `Log`

**Table-Driven Tests**
```go
var tests = []struct {
    input string
    expected string
}{
    {"in1", "exp1"},
    {"in2", "exp2"},
    //....
}
for _, tt := range tests {
    //...
}
```

<h2 id="concurrence-parallelism-and-goroutine">Concurrence, Parallelism and Goroutine</h2>

- *concurrence* application can execute on 1 processor or core using a number of threads.
- *parallelism* same application process executes at the same point in time on a number of cores or processors.
- *goroutine* is not one-to-one correspondence between a goroutine and an operating system thread.


### Set GOMAXPROCS
if GOMAXPROCS is greater than 1, you application can execute simultaneously by more cores. 

**GOMAXPROCS** is equal to the number of threads, on a machine with more than 1 core, as many as thread as there cores can run in parallel.


<h2 id="channel">Channel</h2>

### Semaphore pattern
```go
ch := make(chan int)
go func() {
    //dosomething
    ch <- 1
}
doSomethingElseForAWhile()
<- ch
```

### Channel Factory pattern
```go
func main(){
    stream := pump()
    go suck(stream)
    time.Sleep(1e9)
}
func pump() chan int {
    ch := make(chan int)
    go func(){
        for i:=0; ; i++{
            ch <- i
        }
    }()
    return ch
}
func suck(ch chan int){
    for {
        fmt.Println(<- ch)
    }
}
```

### Channel Iterator pattern
```go
func (c *container) Iter() <-chan items{
    ch := make(chan item)
    go func () {
        for i := 0; i < c.Len(); i++ {
            ch <- c.items[i]
        }
    }()
    return ch 
}

for x := range container.Iter() {
    // ....
}
```

### Producer Consumer pattern
The `produce` putting the value on a channel which is read by `consumer`. Both of them run a separate goroutine.
```go
for {
    Consumer(Produce())
}
```

### Pipe and Filter pattern
```go 
sendChan := make(chan int)
receiveChan := make(chan int)
go processChannel(sendChan, receiveChan)
func processChannel(in <- chan int, out chan<- string){
    for inValue := range in {
        result:= ....// process inValue
        out<- result
    }
}
```

### Use and in- and out- channel insteal of locking
```go
func Worker(in, out chan *Task) {
    for {
        t := <- in
        process(t)
        out<- t
    }
}
```

<h2 id="code-snippets">Code Snippets</h2>
### Strings
- change a character
```go
str := "hello"
c := []byte(str)
c[0] = 'c'
s2 := string(c)
```
- take a part of string
```go
substr := str[n:m]
```
- loop over a string 
```go
// bytes
for i:=0; i < len(str); i++ {
    _ = str[i]
}
//unicode
for ix, ch := range str {
    
}
```
- number of string str
```go 
// bytes fo a string
len(str)
// character in a string
utf8.RuneCountInString(str) // fastest
len([]int(str))
```

- concatenating strings
```go
//fast
var buffer bytes.Buffer
for s := range strs {
    buffer.WriteString(s)
}
buffer.String()

// strings
Strings.join(strs)

// slowest
var str string
for s := range strs {
    str = str + s
}
```
### Slice
- making

`slice1 =  make([]type, len)`

- initialization

`slice1 = []type{val1, val2, val3}`

- searching for a value `V` in a 2 dimensional array/slice
```go
found := false
Found: for row := range arr2Dim {
    for column := range arr2Dim[row]
        if arr2Dim[row][column] == V {
            found = true
            break Found
        }
}
```

### map
- making
```go
map1 := make(map[keyType]ValueType)
```
- initialization
```go
map1 := map[string]int{
   "one" : 1,
   "two" : 2
}
```

- testing if a key exisits in a map 
```go
val1, isPresent = map1[key1]
```

- deleting a key
```go
delete(map1, key1)
```

### struct
- making
```go
type struct1 struct {
    field1 type1
    field2 type2
    ...
}
```
- initialization
```go
ms := new(struct1)
ms := &strcut1{....}
```
- factroy function 
```go
func Newstrcut1(n int, f float32, name string) *strcut1 {
    return &struct1{n, f, name}
}
```

### interface
- test a value implements an interface
```go
if v, ok := v.(Stringer); ok {
    //....
}
```
- type classifier
```go
func classifier(item interface{}){
    switch node:=item.(type){
    case bool: // boolean
    case int: // int 
    case string: //string
    default:
        //default value
    }
}
```
### function
- use cover to stop panic 
```go 
func protect(g func()){
    defer func(){
        if x:=recover(); x!= nil {
            // log it
        }
    }()
    g()
}
```

### file
- open and read a file 
```go
file, err := os.Open("input.dat")
if err != nil {
    fmt.Printf("Open a file with an error: %s", err)
    return
}
defer file.Close()
ireader := bufio.NewReader(file)
for {
    str, err := ireader.ReadString('\n')
    if err != nil {
        return 
    }
    fmt.Printf(str)
}
```
- read and write a file with a sliced buffer
```go
func cat(f *file.File){
    const BUF = 512
    var buf [BUF]byte
    for {
        switch nr, er := f.Read(buf[:]); true{
        case nr<0:
            oes.Exit(1)
        case nr==0: //EOF
            return 
        case nr > 0:
            if nw, ew := file.Stdout.Write(buf[0:nr]); nw!=nr{
                fmt.Printf("wrong")
            }
        }
    }
}
```

### channel
- buffered channels for performance
- limit the number of items in a channel and packing them in array
- loop over a channel 
```go
for v:= range ch {
    // do something with v
}
```
- test if a channel ch is closed
```go
for {
    if input, open := <- ch; !open{
        break
    }else{
        //do something with input
    }
}
```
- timeout pattern
```go
timeout := make(chan bool, 1)
go func(){
    time.sleep(1e9)
    timeout<-true
}()
select {
    case <- ch:
        // a read from ch 
    case <- timeout:
        // time out
}
```
- use in- and out- channel instead of locking
```go
func worker(in, out chan *Task){
    for {
        t := <- in
        process(t)
        out <- t
    }
}
```

<h2 id="advices">Advices</h2>

- Use initializing declaration form `:=` wherever possible.
- Use bytes instead of strings if possible
- Use slices instead of arrays
- Use slices or array instead of map where possible
- Use `for range` over a slice
- When array is sparse, using the map brings out lower memory consumption
- specify an initial capacity for maps
- When define methods, using a pointer to tyepe as a receiver.
- Using caching


<h1 id="go-project-with-makefile">Go Project with Makefile</h1>

Though go provides many tools for us to build project, we still get benifit from `Makefile`

```Makefile
# Go Parameter
GOCMD=go
GOBUILD=$(GOCMD) build
GOCLEAN=$(GOCMD) clean
GOTEST=$(GOCMD) test
GOGET=$(GOCMD) get
BINARY_NAME=myBinary-$$(git describe)
BINARY_LINUX=$(BINARY_NAME)_linux
BINARY_DARWIN=$(BINARY_NAME)_darwin
BINARY_WINDOW=$(BINARY_NAME)_windows

all: deps test build
build:
	$(GOBUILD) -o $(BINARY_NAME) -v

test:
	$(GOTEST) -v ./...

clean:
	$(GOCLEAN)
	rm -rf $(BINARY_NAME)
	rm -rf $(BINARY_LINUX)
	rm -rf $(BINARY_WINDOW)
	rm -rf $(BINARY_DARWIN)
run:
	$(GOBUILD) -o $(BINARY_NAME) -v
	./$(BINARY_NAME)

deps:
	$(GOGET) -u github.com/gorilla/mux
	$(GOGET) -u github.com/go-redis/redis
	$(GOGET) github.com/koding/cache
	$(GOGET) gopkg.in/jarcoal/httpmock.v1

# cross compilation
build-linux:
	CGO_ENABLED=0 GOOS=linux GOARCH=amd64 $(GOBUILD) -o $(BINARY_LINUX) -v

build-darwin:
	CGO_ENABLED=0 GOOS=darwin GOARCH=amd64 $(GOBUILD) -o $(BINARY_DARWIN) -v

build-windows:
	CGO_ENABLED=0 GOOS=windows GOARCH=amd64 $(GOBUILD) -o $(BINARY_WINDOW) -v
```

