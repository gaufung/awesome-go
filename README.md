# Awesome in Go


# 1 Bast Practices

##  Error-handing and panicking in custom package
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

## Error-handing scheme with closures

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

## Testing

- Fail() 

making test function failed

- FailNow()
making test function failed and stop execution.

- Log(args ... interface{})
log test

- Fatal(args ... interface{})
combined `FailNow` and `Log`

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

