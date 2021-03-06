**Table of Content**

- [1 Best Practices](#best-practices)
    - [1.1 Error-Handling in Custom Package](#error-handling-in-custom-package)
    - [1.2 Error-Handling Scheme with Closure](#error-handling-scheme-with-clouse)
    - [1.3 Testing](#testing)
    - [1.4 Channel](#channel)
    - [1.5 Code Snippets](#code-snippets)
    - [1.6 Advices](#advices)

- [2 Go Project with Makefile](#go-project-with-makefile)
- [3 Go Web Unit Test](#go-web-unit-test)
- [4 HTTP Mock](#http-mock)
- [5 GO Libraries](#go-libraries)
    - [5.1 Testing](#testing-library)
- [6 Go Concurrenty Pattern](#go-concurrency-pattern)
    - [6.1 Prevent Goroutine Leaks](#prevent-goroutine-leaks)
    - [6.2 Error Handling](#error-handling)
    - [6.3 Pipeline](#pipeline)
    - [6.4 Fan-out, fan-in](#fan-out-fan-in)
    - [6.5 The Tee-channel](#the-tee-channel)
    - [6.6 Context](#context)
- [7 Concurrency at Scale](#concurrency-at-scale)
    - [7.1 Hearbets](#heartbeats)
- [8 Traps](#traps)
    - [8.1 Never Guarantee Concurrency](#never-guarantee-concurrency)
    - [8.2 Methods of Type Pointer Receiver](#type-point-receiver-method)

<h1 id="best-practices">1 Best Practices</h1>
<h2 id="error-handling-in-custom-package">1.1 Error-Handling in Custom Package</h2>

-  *always recover from panic from your package*
-  *return errors to the caller of your package*
-  *error message with low-case sentences*

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

<h2 id="error-handling-scheme-with-clouse">1.2 Error-Handling Scheme with Closure</h2>

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

<h2 id="test">1.3 Testing</h2>

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


<h2 id="channel">1.4 Channel</h2>

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

<h2 id="code-snippets">1.5 Code Snippets</h2>

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

<h2 id="advices">1.6 Advices</h2>

- Use initializing declaration form `:=` wherever possible.
- Use bytes instead of strings if possible
- Use slices instead of arrays
- Use slices or array instead of map where possible
- Use `for range` over a slice
- When array is sparse, using the map brings out lower memory consumption
- specify an initial capacity for maps
- When define methods, using a pointer to tyepe as a receiver.
- Using caching
- closing http response body
```go
resp, err := http.Get("https://api.ipify.org?format=json")
if err != nil {
    fmt.Println(err)
    return
}
defer resp.Body.Close()//ok, most of the time :-)
body, err := ioutil.ReadAll(resp.Body)
```
- When you create a type declaration by defining a new type from an existing type, you don't inherit the methods defined for that existing type.

<h1 id="go-project-with-makefile">2 Go Project with Makefile</h1>

Though go provides many tools for us to build project, we still get benifit from `Makefile`.

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

<h1 id="go-web-unit-test">3 Go Web Unit Test</h1>
As you complete some http request handlers, you want to test them to check they work as expected.

```go
func MyHandler(w http.ResponseWriter, r *http.Request){
    // http logic that you havt to do.
}

// unit-test
import "github.com/gorila/mux"
request, err := http.NewReuqest("GET", url, nil)
if err != nil {
    t.Fatal(err)
}
rr := httptest.NewRecorder()
router := mux.NewRouter()
router.HandleFuc(url, MyHandler)
router.ServeHTTP(rr, req)
if rr.code != exptectCode{
    t.Errorf("....")
}
```

<h1 id="http-mock">4 Http Mock</h1>
Once you want to mock a http server, you do not rewrite your codes to adjust to the mock interfaces

```go
import "gopkg.in/jarcoal/httpmock.v1"
httpmock.Activate()
defer httmock.DeactivateAndReset()
httpmock.RegisterReponder("GET", url,
    httpmock.NewStringResponder(200, `{'request':10}`))

//...
```

<h1 id="go-libraries">5 Go Libraries</h1>

<h2 id="testing-library">5.1 Testing</h2>

### Unit Test
- `SkipNow` Skip test and break down test
- `Skip` Skip test
- `parallel`: Run test with other tests that also have  `t.Parallel()`。

```go
func TestWriteToMap(t *testing.T) {
    t.Parallel()
    for _, tt := range pairs {
        WriteToMap(tt.k, tt.v)
    }
}
func TestReadFromMap(t *testing.T) {
    t.Parallel()
    for _, tt := range pairs {
        actual := ReadFromMap(tt.k)
        if actual != tt.v {
            t.Errorf("the value of key(%s) is %s, expected: %s", tt.k, actual, tt.v)
        }
    }
}
```

### Benchmark Test
```go
func BenchmarkHello(b *testing.B){
    for i:=0; i<b.N;i++{
        fmt.Sprintf("hello")
    }
}
```
The test will run code by `b.N` times and monitor the time and memory consumption  per loop.
`go test -bench=. --benchmem`

```
BenchmarkHello 100000 282 ns/op  1 alloc/op 
```

However, if you want to run the tests in parallel, you can refer to `RunParalle` method. Go will
create a some goroutines and divide `n` into these goroutines.
```go
func BenchmarkTmplExucte(b *testing.B) {
    b.ReportAllocs()
    templ := template.Must(template.New("test").Parse("Hello, {{.}}!"))
    b.RunParallel(func(pb *testing.PB) {
        // Each goroutine has its own bytes.Buffer.
        var buf bytes.Buffer
        for pb.Next() {
            // The loop body is executed b.N times total across all goroutines.
            buf.Reset()
            templ.Execute(&buf, "World")
        }
    })
}
```

### Examples
Verify the example code, Example functions may including line comment that begin with "Output:"

```go
func ExampleHello() {
    fmt.Println("Hello")
    // Output: Hello
}
```
Example convention
- package example: `func Example(){...}`
- function example: `func ExampleF(){...}`
- type example: `func ExampleT(){...}`
- method of type example: `func ExampleT_M(){...}`
For various proposes, you can also add suffix to name the above functions.

### Test setup and teardown
If you get used to `JUnit` framework, you are quite familiar with `setup` and `teardown` to 
initialize and destrory resources at every time to run tests. Go also provide these features for 
go test.
```go
var db struct {  
    Dns string
}
func TestMain(m *testing.M) {
    // initialization
    db.Dns = os.Getenv("DATABASE_DNS")
    if db.Dns == "" {
        db.Dns = "root:123456@tcp(localhost:3306)/?charset=utf8&parseTime=True&loc=Local"
    }

    flag.Parse()
    exitCode := m.Run()

    // clean resources
    db.Dns = ""

    // report test
    os.Exit(exitCode)
}
```

<h1 id="go-concurrency-pattern">6 Go Concurrency Pattern</h1>
<h2 id="prevent-goroutine-leaks">6.1 Prevent Goroutine Leaks</h2>

> if a goroutine is responsiable for creating another goroutine, it is also responsiable for ensuring it can stop the goroutine


- Creating consumer goroutine, using `done` as signal.

```go
doWork := func (
    done <- chan interface{},
    strings <- chan string,
)<- chan interface{} {
    terminated := make(chan interface{})
    go func () {
        defer fmt.Println("dowork exited. ")
        defer close(terminated)
        for {
            select {
                case s := <- strings
                    fmt.Println(s)
                case <- done
                    return
            }
        }
    }()
    return terminated
}
done := make(chan interface{})
terminated := doWork(done, nil)
go func(){
    time.Sleep(1 * time.Second)
    fmt.Println("Cancel doWork goroutine")
    close(done)
}()
<- terminated
fmt.Println("done")
```

- Creating producer goroutine, using `done` as signal

```go
newRandStream := func(done<-chan interface{}) <- chan int {
    randStream := make(chan int)
    go func(){
        defer fmt.Println("newRandStream clousre exited.")
        defer close(randStream)
        for {
            select {
                case randStream <- rand.Int():
                case <- done:
                    return
            }
        }
    }()
    return randStream
}()
done := make(chan interface{})
randStream := newRandStream(done)
fmt.Println("3 random int")
for i:=0; i < 3 ; i++ {
    fmt.Println(<-randStream)
}
close(done)
```

<h2 id="error-handling">6.2 Error Handling </h2>

Sending errors to another part of program which has complete information about the state of sender.

```go
type Result struct {
    Error error
    Response *http.Response
}
checkStatus := func(done<-chan interface{}, urls ... string)<-chan Result {
    results := make(chan Result)
    go func() {
        defer close(results)
        for _, url := range urls {
            var result Result
            resp, err := http.Get(url)
            result := Result{Error: err, Response: resp}
            select {
                case <- done:
                    return
                case results <- result:
            }
        }
    }()
    return results
}
done := make(chan interface{})
defer close(done)
urls := []string{"www.google.com", "https://badhost"}
for result := range checkStatus(done, urls...){
    if result.Error != nil {
        fmt.Printf("error: %v", result.Error)
        continue
    }
    fmt.Printf("Respnse: %v\n", result.Response.Status)
}
```

<h2 id="pipeline">6.3 Pipeline</h2>
Each stage takes input from upstream, processes it and sends it to downstream.

```go
generator := func(done<-chan interface{}, integers ...int) <- chan int {
    intStream := make(chan int)
    go func(){
        defer close(intstream)
        for _, i := range integers {
            select {
                case <- done:
                    return
                case intStream <- i:
            }
        }
    }()
    return intStream
}
multiply := func(
    done <- chan interface{},
    intStream <- chan int,
    multiplier int,
)<- chan int {
    multipliedStream := make(chan int)
    go func() {
        defer close(multipliedStream)
        for i := range intStream {
            select {
                case <-done:
                    return
                case multipliedStream <- i * multiplier
            }
        }
    }
    return multipliedStrem
}
add := func(
    done <-chan interface{},
    intStream <- chan int,
    additive int,
)<- chan int {
    addedStream := make(chan int)
    go func(){
        defer close(addedStream)
        for i := range intStream {
            select {
                case <- done:
                    return
                case addedStream <- i + additive:
            }
        }
    }
    return addedStream
}
done := make(chan interface{})
defer close(done)

intStream := genertor(done, 1, 2, 3, 4)
pipeline := multiply(done, add(done, multiply(done, intStream, 2), 1), 2)
for v := range pipeline {
    fmt.Println(v)
}
```

<h2 id="fan-out-fan-in">6.4 Fan-Out, Fan-In</h2>

**Criterias** of fan-out

- order independence
- duration

```go
func gen(ctx context.Context, nums... int) <-chan int {
	out := make(chan int)
	go func(){
		defer close(out)
		for _, n := range nums {
			select {
			case <- ctx.Done:
				return 
			default:
				if n % n == 0 {
					out <- n
				}
			}
		}
	}()
	return out
}

func sq(ctx context.Context, in <- chan int) <- chan float64 {
	out := make(chan float64)
	go func() {
		defer clsoe(out)
		for n:= range in {
			select {
			case <- ctx.Done():
				return
			default:
				out <- math.Sqrt(float64(n))
			}
		}
	}()
	return out 
}

func merge(ctx context.Context, cs ... <- chan float64) <- chan float64 {
	var wg sync.WaitGroup
	out := make(chan float64)
	output := func(c <-chan float64) {
		defer wg.Done()
		for n := range c {
			select {
			case <- ctx.Done():
				return
			default:
				out <- n * n
			}
		}
	}
	wg.Add(len(cs))
	for _, c := range cs {
		go ouput(c)
	}
	
	go func() {
		wg.Wait()
		close(out)
	}()
	return out
}

func main(){
	ctx, cancel := context.WitchCancel(context.Backgroud())
	defer cancel()
	
	a := []int {2, 3, 4, 5, 6, 7}
	in := gen(ctx, a...)
	c := make([]<- chan float64, 3)
	for i:=0; i<3; i++ {
		c[i] = sq(ctx, in)
	}
	for n := range merge(ctx, c...){
		fmt.Printf("%d\s", n)
	}
}
```

<h2 id="the-tee-channel">6.5 The Tee-channel </h2>

`tee` command in Unix-like system

```go
tee := func(
    done <- chan interface{},
    in <- chan interface{},
)(_, _ <-chan interface{}) {<-chan interface{}} {
    out1 := make(chan interface{})
    out2 := make(chan interface{})
    go func() {
        defer close(out1)
        defer close(out2)
        for val := range orDone(done, in){
            var out1, out2 = out1, out2
            for i:= 0; i<2; i++ {
                select {
                    case <- done:
                    case out1<-val:
                        out1 = nil
                    case out3<- val:
                        out2 =nil
                }
            }
        }
    }
    return out1, out2
}
```


<h2 id="context">6.6 Context</h2>

`context` pacakge was brought into standard library since Go 1.7. It helps to make Go cocurrency idioms easily.

```go
var Canceled = errors.New("context canceled")
var DeadlineExceeded error = deadlineExceededError{}
type CancelFunc func()
type Context interface {}
func Background() Context
func TODO() Context
func WithCancel(parent Context)(Context, CancelFunc)
func WithDeadline(parent Context, deadline time.Time)(Context, CancelFunc)
func WithTimeout(parent Context, timeout time.Duration)(Context, CancelFunc)
func WithValue(parent Contex, key, val interface{}) Context
```

**Usage**

```go
var wg sync.WaitGroup
ctx, cancel := context.WithCancel(context.Background())
defer cancel()
wg.Add(1)
go func() {
	defer wg.Done()
	if err := printGreeting(ctx); err != nil {
		fmt.Printf("cannot print greeting: %v\n", err)
		cancel()
	}
}()
wg.Add(1)
go func() {
	defer wg.Done()
	if err := printFarewell(ctx); err != nil {
		fmt.Printf("cannot print farewell: %v\n", err)
	}
}()
wg.Wait()
processRequest("jane", "abc123")

func printGreeting(ctx context.Context) error {
	if greeting, err := genGreeting(ctx); err != nil {
		return err
	} else {
		fmt.Printf("%s world!\n", greeting)
		return nil
	}
}

func printFarewell(ctx context.Context) error {
	if farewell, err := genFarewell(ctx); err != nil {
		return err
	} else {
		fmt.Printf("%s world!\n", farewell)
		return nil
	}
}

func genGreeting(ctx context.Context) (string, error) {
	ctx, cancel := context.WithTimeout(ctx, 1*time.Second)
	defer cancel()
	switch locale, err := locale(ctx); {
	case err != nil:
		return "", err
	case locale == "EN/US":
		return "hello", nil
	}
	return "", fmt.Errorf("unsupport locale")
}

func genFarewell(ctx context.Context) (string, error) {
	switch locale, err := locale(ctx); {
	case err != nil:
		return "", err
	case locale == "EN/US":
		return "goodbye", nil
	}
	return "", fmt.Errorf("unsupport locale")
}

func locale(ctx context.Context) (string, error) {
	select {
	case <-ctx.Done():
		return "", ctx.Err()
	case <-time.After(1 * time.Minute):
	}
	return "EN/US", nil
}

type ctxKey int

const (
	ctxUserId ctxKey = iota
	ctxAuthToken
)

func UserId(c context.Context) string {
	return c.Value(ctxUserId).(string)
}
func Authoken(c context.Context) string {
	return c.Value(ctxAuthToken).(string)
}

func processRequest(userId, authToken string) {
	ctx := context.WithValue(context.Background(), ctxUserId, userId)
	ctx = context.WithValue(ctx, ctxAuthToken, authToken)
	handleResponse(ctx)
}
func handleResponse(ctx context.Context) {
	fmt.Printf("handling response for %v (%v)", UserId(ctx), Authoken(ctx))
}
```

<h1 id="concurrency-at-scale">7 Concurrency at Scale</h1>
<h2 id="heartbeats">7.1 Heartbeats</h2>

- invoke heartbeats at specific time interval
```go
doWork := func(
    done <-chan interface{},
    pluseInterval time.Duration,
)(<-chan interface{}, <-chan time.Time){
    heartbeat := make(chan interface{})
    results := make(chan time.Time)
    go func(){
        defer close(heartbeat)
        defer close(results)

        pluse := time.Tick(plusInterval)
        workGen := time.Tick(2 * plusInterval)
        sendPluse := func(){
            select {
                case heartbeat <- struct{}{}:
                default:
            }
        }
        sendResult := func(r time.Time){
            for {
                select {
                case <- done:
                    return
                case <-plus:
                    sendPluse()
                case result <- r:
                    return
                }
            }
        }
        for {
            select {
            case <-done:
                return
            case <- pluse:
                sendPluse()
            case r := <- workGen:
                sendResult(r)
            }
        }
    }()
    return heartbeat, result
}
```


<h1 id="traps">8 Traps</h1>
<h2 id="never-guarantee-concurrency">8.1 Never Guarantee Concurrency</h2>
What will output of the following code?

```go
func main() {
    names := []string{"tom", "allen", "lili"}
    for _, name := range names {
        go func(){
            fmt.Println(name)
        }()
    }
    runtime.GOMAXPROCS(1)
    runtime.Gosched()
}
```
The output is:
```
lili
lili
lili
```
We can find out that goroutine do not start at the time it is created. You MUST never guarantee the begining of concurrency. How we can rectify the code to meet our expectation.

```go
func main(){
    names := []string{"tom", "allen", "lili"}
    for _, name := range names {
        go func(){
            fmt.Println(name)
        }()
        time.Sleep(time.Second)
    }
    runtime.GOMAXPROCS(1)
    runtime.Gosched()
}
```
The output will be:
```
tom
allen
lili
```
Main goroutine `sleeps` one second to wait sub goroutine starts.

<h2 id="type-point-receiver-method">8.2 Methods of Type Pointer Receiver</h2>
Can this code will work?

```go
type Lili struct {

}
func (li *Lili) printPointer(){
    fmt.Println("Pointer")
}
func main(){
    li := Lili{}
    li.printPointer()
}
```

It will print `Pointer`， which means `li` can call `printPointer` method because `li` is addressable.
What about the following code:

```go
func main() {
    Lili{}.printPointer()
}
```
You will get compile error as `Lili{}` is not addressable.

