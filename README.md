# Awesome in Go
Best Practice in Go


# Error-handing and panicking in custom package
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

