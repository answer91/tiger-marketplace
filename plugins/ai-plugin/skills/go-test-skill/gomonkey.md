# gomonkey使用方法及示例

**示例**
function return

```golang
package test

import (
	"testing"

	. "github.com/agiledragon/gomonkey/v2"
	"github.com/agiledragon/gomonkey/v2/test/fake"
	. "github.com/smartystreets/goconvey/convey"
)

/*
  compare with apply_func_seq_test.go
*/
func TestApplyFuncReturn(t *testing.T) {
	Convey("TestApplyFuncReturn", t, func() {

		Convey("declares the values to be returned", func() {
			info1 := "hello cpp"

			patches := ApplyFuncReturn(fake.ReadLeaf, info1, nil)
			defer patches.Reset()

			for i := 0; i < 10; i++ {
				output, err := fake.ReadLeaf("")
				So(err, ShouldEqual, nil)
				So(output, ShouldEqual, info1)
			}

			patches.Reset() // if not reset will occur:patch has been existed
			info2 := "hello golang"
			patches.ApplyFuncReturn(fake.ReadLeaf, info2, nil)
			for i := 0; i < 10; i++ {
				output, err := fake.ReadLeaf("")
				So(err, ShouldEqual, nil)
				So(output, ShouldEqual, info2)
			}
		})
	})
}
```

**示例**
sequence return

```golang
package test

import (
	"runtime"
	"testing"

	. "github.com/agiledragon/gomonkey/v2"
	"github.com/agiledragon/gomonkey/v2/test/fake"
	. "github.com/smartystreets/goconvey/convey"
)

func TestApplyFuncSeq(t *testing.T) {
	Convey("TestApplyFuncSeq", t, func() {

		Convey("default times is 1", func() {
			info1 := "hello cpp"
			info2 := "hello golang"
			info3 := "hello gomonkey"
			outputs := []OutputCell{
				{Values: Params{info1, nil}},
				{Values: Params{info2, nil}},
				{Values: Params{info3, nil}},
			}
			patches := ApplyFuncSeq(fake.ReadLeaf, outputs)
			defer patches.Reset()

			runtime.GC()

			output, err := fake.ReadLeaf("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info1)
			output, err = fake.ReadLeaf("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info2)
			output, err = fake.ReadLeaf("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info3)
		})

		Convey("retry succ util the third times", func() {
			info1 := "hello cpp"
			outputs := []OutputCell{
				{Values: Params{"", fake.ErrActual}, Times: 2},
				{Values: Params{info1, nil}},
			}
			patches := ApplyFuncSeq(fake.ReadLeaf, outputs)
			defer patches.Reset()
			output, err := fake.ReadLeaf("")
			So(err, ShouldEqual, fake.ErrActual)
			output, err = fake.ReadLeaf("")
			So(err, ShouldEqual, fake.ErrActual)
			output, err = fake.ReadLeaf("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info1)
		})

		Convey("batch operations failed on the third time", func() {
			info1 := "hello gomonkey"
			outputs := []OutputCell{
				{Values: Params{info1, nil}, Times: 2},
				{Values: Params{"", fake.ErrActual}},
			}
			patches := ApplyFuncSeq(fake.ReadLeaf, outputs)
			defer patches.Reset()
			output, err := fake.ReadLeaf("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info1)
			output, err = fake.ReadLeaf("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info1)
			output, err = fake.ReadLeaf("")
			So(err, ShouldEqual, fake.ErrActual)
		})

	})
}
```

**示例**
function

```golang
package test

import (
	"encoding/json"
	"testing"

	. "github.com/agiledragon/gomonkey/v2"
	"github.com/agiledragon/gomonkey/v2/test/fake"
	. "github.com/smartystreets/goconvey/convey"
)

var (
	outputExpect = "xxx-vethName100-yyy"
)

func TestApplyFunc(t *testing.T) {
	Convey("TestApplyFunc", t, func() {

		Convey("one func for succ", func() {
			patches := ApplyFunc(fake.Exec, func(_ string, _ ...string) (string, error) {
				return outputExpect, nil
			})
			defer patches.Reset()
			output, err := fake.Exec("", "")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, outputExpect)
		})

		Convey("one func for succ with origin", func() {
			patches := ApplyFunc(fake.Belong, func(_ string, _ []string) bool {
				return false
			})
			defer patches.Reset()
			output := fake.Belong("a", []string{"a", "b"})
			So(output, ShouldEqual, false)
			patches.Origin(func() {
				output = fake.Belong("a", []string{"a", "b"})
			})
			So(output, ShouldEqual, true)
		})

		Convey("one func for succ with origin inside", func() {
			var output bool
			var patches *Patches
			patches = ApplyFunc(fake.Belong, func(_ string, _ []string) bool {
				patches.Origin(func() {
					output = fake.Belong("a", []string{"a", "b"})
					So(output, ShouldEqual, true)
				})
				return false
			})
			defer patches.Reset()
		})

		Convey("one func for fail", func() {
			patches := ApplyFunc(fake.Exec, func(_ string, _ ...string) (string, error) {
				return "", fake.ErrActual
			})
			defer patches.Reset()
			output, err := fake.Exec("", "")
			So(err, ShouldEqual, fake.ErrActual)
			So(output, ShouldEqual, "")
		})

		Convey("two funcs", func() {
			patches := ApplyFunc(fake.Exec, func(_ string, _ ...string) (string, error) {
				return outputExpect, nil
			})
			defer patches.Reset()
			patches.ApplyFunc(fake.Belong, func(_ string, _ []string) bool {
				return true
			})
			output, err := fake.Exec("", "")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, outputExpect)
			flag := fake.Belong("", nil)
			So(flag, ShouldBeTrue)
		})

		Convey("two funcs with origin", func() {
			patches := ApplyFunc(fake.Exec, func(_ string, _ ...string) (string, error) {
				return outputExpect, nil
			})
			defer patches.Reset()
			patches.ApplyFunc(fake.Belong, func(_ string, _ []string) bool {
				return true
			})
			output, err := fake.Exec("", "")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, outputExpect)
			flag := fake.Belong("", nil)
			So(flag, ShouldBeTrue)

			var outputBool bool
			patches.Origin(func() {
				outputBool = fake.Belong("c", []string{"a", "b"})
			})
			So(outputBool, ShouldEqual, false)
		})

		Convey("input and output param", func() {
			patches := ApplyFunc(json.Unmarshal, func(data []byte, v interface{}) error {
				if data == nil {
					panic("input param is nil!")
				}
				p := v.(*map[int]int)
				*p = make(map[int]int)
				(*p)[1] = 2
				(*p)[2] = 4
				return nil
			})
			defer patches.Reset()
			var m map[int]int
			err := json.Unmarshal([]byte("123"), &m)
			So(err, ShouldEqual, nil)
			So(m[1], ShouldEqual, 2)
			So(m[2], ShouldEqual, 4)
		})

		Convey("repeat patch same func", func() {
			patches := ApplyFunc(fake.ReadLeaf, func(_ string) (string, error) {
				return "patch1", nil
			})
			output, err := fake.ReadLeaf("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, "patch1")

			patches.ApplyFunc(fake.ReadLeaf, func(_ string) (string, error) {
				return "patch2", nil
			})
			output, err = fake.ReadLeaf("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, "patch2")

			patches.Reset()
			output, err = fake.ReadLeaf("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, "Hello, World!")
		})

		Convey("declare partial args", func() {
			patches := ApplyFunc(fake.Exec, func() (string, error) {
				return outputExpect, nil
			})
			defer patches.Reset()
			output, err := fake.Exec("", "")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, outputExpect)

			patches.ApplyFunc(fake.Exec, func(_ string) (string, error) {
				return outputExpect, nil
			})
			output, err = fake.Exec("", "")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, outputExpect)

			So(func() {
				patches.ApplyFunc(fake.Exec, func(_ string, _ []string) (string, error) {
					return outputExpect, nil
				})
			}, ShouldPanic)

			patches.ApplyFunc(fake.Exec, func(_ string, _ ...string) (string, error) {
				return outputExpect, nil
			})
			output, err = fake.Exec("", "")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, outputExpect)
		})
	})
}
```

**示例**

var

```golang
package test

import (
	"testing"

	. "github.com/agiledragon/gomonkey/v2"
	"github.com/agiledragon/gomonkey/v2/test/fake"
	. "github.com/smartystreets/goconvey/convey"
)

/*
  compare with apply_func_var_seq_test.go
*/
func TestApplyFuncVarReturn(t *testing.T) {
	Convey("TestApplyFuncVarReturn", t, func() {

		Convey("declares the values to be returned", func() {
			info1 := "hello cpp"

			patches := ApplyFuncVarReturn(&fake.Marshal, []byte(info1), nil)
			defer patches.Reset()
			for i := 0; i < 10; i++ {
				bytes, err := fake.Marshal("")
				So(err, ShouldEqual, nil)
				So(string(bytes), ShouldEqual, info1)
			}

			info2 := "hello golang"
			patches.ApplyFuncVarReturn(&fake.Marshal, []byte(info2), nil)
			for i := 0; i < 10; i++ {
				bytes, err := fake.Marshal("")
				So(err, ShouldEqual, nil)
				So(string(bytes), ShouldEqual, info2)
			}
		})

	})
}
```

**示例**
var sequence

```golang
package test

import (
	"testing"

	. "github.com/agiledragon/gomonkey/v2"
	"github.com/agiledragon/gomonkey/v2/test/fake"
	. "github.com/smartystreets/goconvey/convey"
)

func TestApplyFuncVarSeq(t *testing.T) {
	Convey("TestApplyFuncVarSeq", t, func() {

		Convey("default times is 1", func() {
			info1 := "hello cpp"
			info2 := "hello golang"
			info3 := "hello gomonkey"
			outputs := []OutputCell{
				{Values: Params{[]byte(info1), nil}},
				{Values: Params{[]byte(info2), nil}},
				{Values: Params{[]byte(info3), nil}},
			}
			patches := ApplyFuncVarSeq(&fake.Marshal, outputs)
			defer patches.Reset()
			bytes, err := fake.Marshal("")
			So(err, ShouldEqual, nil)
			So(string(bytes), ShouldEqual, info1)
			bytes, err = fake.Marshal("")
			So(err, ShouldEqual, nil)
			So(string(bytes), ShouldEqual, info2)
			bytes, err = fake.Marshal("")
			So(err, ShouldEqual, nil)
			So(string(bytes), ShouldEqual, info3)
		})

		Convey("retry succ util the third times", func() {
			info1 := "hello cpp"
			outputs := []OutputCell{
				{Values: Params{[]byte(""), fake.ErrActual}, Times: 2},
				{Values: Params{[]byte(info1), nil}},
			}
			patches := ApplyFuncVarSeq(&fake.Marshal, outputs)
			defer patches.Reset()
			bytes, err := fake.Marshal("")
			So(err, ShouldEqual, fake.ErrActual)
			bytes, err = fake.Marshal("")
			So(err, ShouldEqual, fake.ErrActual)
			bytes, err = fake.Marshal("")
			So(err, ShouldEqual, nil)
			So(string(bytes), ShouldEqual, info1)
		})

		Convey("batch operations failed on the third time", func() {
			info1 := "hello gomonkey"
			outputs := []OutputCell{
				{Values: Params{[]byte(info1), nil}, Times: 2},
				{Values: Params{[]byte(""), fake.ErrActual}},
			}
			patches := ApplyFuncVarSeq(&fake.Marshal, outputs)
			defer patches.Reset()
			bytes, err := fake.Marshal("")
			So(err, ShouldEqual, nil)
			So(string(bytes), ShouldEqual, info1)
			bytes, err = fake.Marshal("")
			So(err, ShouldEqual, nil)
			So(string(bytes), ShouldEqual, info1)
			bytes, err = fake.Marshal("")
			So(err, ShouldEqual, fake.ErrActual)
		})

	})
}
```

**示例**

```golang
package test

import (
	"testing"

	. "github.com/agiledragon/gomonkey/v2"
	"github.com/agiledragon/gomonkey/v2/test/fake"
	. "github.com/smartystreets/goconvey/convey"
)

func TestApplyFuncVar(t *testing.T) {
	Convey("TestApplyFuncVar", t, func() {

		Convey("for succ", func() {
			str := "hello"
			patches := ApplyFuncVar(&fake.Marshal, func(_ interface{}) ([]byte, error) {
				return []byte(str), nil
			})
			defer patches.Reset()
			bytes, err := fake.Marshal(nil)
			So(err, ShouldEqual, nil)
			So(string(bytes), ShouldEqual, str)
		})

		Convey("for fail", func() {
			patches := ApplyFuncVar(&fake.Marshal, func(_ interface{}) ([]byte, error) {
				return nil, fake.ErrActual
			})
			defer patches.Reset()
			_, err := fake.Marshal(nil)
			So(err, ShouldEqual, fake.ErrActual)
		})
	})
}
```

**示例**

```golang
package test

import (
	"testing"

	. "github.com/agiledragon/gomonkey/v2"
	. "github.com/smartystreets/goconvey/convey"
)

var num = 10

func TestApplyGlobalVar(t *testing.T) {
	Convey("TestApplyGlobalVar", t, func() {

		Convey("change", func() {
			patches := ApplyGlobalVar(&num, 150)
			defer patches.Reset()
			So(num, ShouldEqual, 150)
		})

		Convey("recover", func() {
			So(num, ShouldEqual, 10)
		})
	})
}
```

**示例**

```golang
package test

import (
	"testing"

	. "github.com/agiledragon/gomonkey/v2"
	"github.com/agiledragon/gomonkey/v2/test/fake"
	. "github.com/smartystreets/goconvey/convey"
)

func TestApplyInterfaceReused(t *testing.T) {
	e := &fake.Etcd{}

	Convey("TestApplyInterfaceReused", t, func() {
		patches := ApplyFunc(fake.NewDb, func(_ string) fake.Db {
			return e
		})
		defer patches.Reset()
		db := fake.NewDb("mysql")

		Convey("TestApplyInterface", func() {
			info := "hello interface"
			patches.ApplyMethod(e, "Retrieve",
				func(_ *fake.Etcd, _ string) (string, error) {
					return info, nil
				})
			output, err := db.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info)
		})

		Convey("TestApplyInterfaceSeq", func() {
			info1 := "hello cpp"
			info2 := "hello golang"
			info3 := "hello gomonkey"
			outputs := []OutputCell{
				{Values: Params{info1, nil}},
				{Values: Params{info2, nil}},
				{Values: Params{info3, nil}},
			}
			patches.ApplyMethodSeq(e, "Retrieve", outputs)
			output, err := db.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info1)
			output, err = db.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info2)
			output, err = db.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info3)
		})

		Convey("the arg type can be interface", func() {
			info := "hello interface"
			patches.ApplyMethod(e, "Retrieve",
				func(_ fake.Db, _ string) (string, error) {
					return info, nil
				})
			output, err := db.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info)
		})
	})
}
```

**示例**

```golang
package test

import (
	"testing"

	. "github.com/agiledragon/gomonkey/v2"
	"github.com/agiledragon/gomonkey/v2/test/fake"
	. "github.com/smartystreets/goconvey/convey"
)

/*
	compare with apply_method_test.go, no need pass receiver
*/

func TestApplyMethodFunc(t *testing.T) {
	slice := fake.NewSlice()
	var s *fake.Slice
	Convey("TestApplyMethodFunc", t, func() {
		Convey("for succ", func() {
			err := slice.Add(1)
			So(err, ShouldEqual, nil)
			patches := ApplyMethodFunc(s, "Add", func(_ int) error {
				return nil
			})
			defer patches.Reset()
			err = slice.Add(1)
			So(err, ShouldEqual, nil)
			err = slice.Remove(1)
			So(err, ShouldEqual, nil)
			So(len(slice), ShouldEqual, 0)
		})

		Convey("for origin", func() {
			patches := ApplyMethodFunc(s, "Add", func(_ int) error {
				return nil
			})
			defer patches.Reset()

			var err error
			patches.Origin(func() {
				err = slice.Add(1)
				So(err, ShouldEqual, nil)
				err = slice.Add(1)
				So(err, ShouldEqual, fake.ErrElemExsit)
				err = slice.Remove(1)
				So(err, ShouldEqual, nil)
			})
			So(len(slice), ShouldEqual, 0)
		})

		Convey("for already exist", func() {
			err := slice.Add(2)
			So(err, ShouldEqual, nil)
			patches := ApplyMethodFunc(s, "Add", func(_ int) error {
				return fake.ErrElemExsit
			})
			defer patches.Reset()
			err = slice.Add(1)
			So(err, ShouldEqual, fake.ErrElemExsit)
			err = slice.Remove(2)
			So(err, ShouldEqual, nil)
			So(len(slice), ShouldEqual, 0)
		})

		Convey("two methods", func() {
			err := slice.Add(3)
			So(err, ShouldEqual, nil)
			defer slice.Remove(3)
			patches := ApplyMethodFunc(s, "Add", func(_ int) error {
				return fake.ErrElemExsit
			})
			defer patches.Reset()
			patches.ApplyMethodFunc(s, "Remove", func(_ int) error {
				return fake.ErrElemNotExsit
			})
			err = slice.Add(2)
			So(err, ShouldEqual, fake.ErrElemExsit)
			err = slice.Remove(1)
			So(err, ShouldEqual, fake.ErrElemNotExsit)
			So(len(slice), ShouldEqual, 1)
			So(slice[0], ShouldEqual, 3)
		})

		Convey("one func and one method", func() {
			err := slice.Add(4)
			So(err, ShouldEqual, nil)
			defer slice.Remove(4)
			patches := ApplyFunc(fake.Exec, func(_ string, _ ...string) (string, error) {
				return outputExpect, nil
			})
			defer patches.Reset()
			patches.ApplyMethodFunc(s, "Remove", func(_ int) error {
				return fake.ErrElemNotExsit
			})
			output, err := fake.Exec("", "")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, outputExpect)
			err = slice.Remove(1)
			So(err, ShouldEqual, fake.ErrElemNotExsit)
			So(len(slice), ShouldEqual, 1)
			So(slice[0], ShouldEqual, 4)
		})

		Convey("for variadic method", func() {
			slice = fake.NewSlice()
			count := slice.Append(1, 2, 3)
			So(count, ShouldEqual, 3)
			patches := ApplyMethodFunc(s, "Append", func(_ ...int) int {
				return 0
			})
			defer patches.Reset()
			count = slice.Append(4, 5, 6)
			So(count, ShouldEqual, 0)
			So(len(slice), ShouldEqual, 3)
		})
	})
}
```

**示例**

```golang
package test

import (
	"testing"

	. "github.com/agiledragon/gomonkey/v2"
	"github.com/agiledragon/gomonkey/v2/test/fake"
	. "github.com/smartystreets/goconvey/convey"
)

/*
   compare with apply_method_seq_test.go
*/

func TestApplyMethodReturn(t *testing.T) {
	e := &fake.Etcd{}
	Convey("TestApplyMethodReturn", t, func() {
		Convey("declares the values to be returned", func() {
			info1 := "hello cpp"
			patches := ApplyMethodReturn(e, "Retrieve", info1, nil)
			defer patches.Reset()
			for i := 0; i < 10; i++ {
				output1, err1 := e.Retrieve("")
				So(err1, ShouldEqual, nil)
				So(output1, ShouldEqual, info1)
			}

			patches.Reset() // if not reset will occur:patch has been existed
			info2 := "hello golang"
			patches.ApplyMethodReturn(e, "Retrieve", info2, nil)
			for i := 0; i < 10; i++ {
				output2, err2 := e.Retrieve("")
				So(err2, ShouldEqual, nil)
				So(output2, ShouldEqual, info2)
			}
		})
	})
}
```

**示例**

```golang
package test

import (
	"testing"

	. "github.com/agiledragon/gomonkey/v2"
	"github.com/agiledragon/gomonkey/v2/test/fake"
	. "github.com/smartystreets/goconvey/convey"
)

func TestApplyMethodSeq(t *testing.T) {
	e := &fake.Etcd{}
	Convey("TestApplyMethodSeq", t, func() {

		Convey("default times is 1", func() {
			info1 := "hello cpp"
			info2 := "hello golang"
			info3 := "hello gomonkey"
			outputs := []OutputCell{
				{Values: Params{info1, nil}},
				{Values: Params{info2, nil}},
				{Values: Params{info3, nil}},
			}
			patches := ApplyMethodSeq(e, "Retrieve", outputs)
			defer patches.Reset()
			output, err := e.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info1)
			output, err = e.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info2)
			output, err = e.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info3)
		})

		Convey("retry succ util the third times", func() {
			info1 := "hello cpp"
			outputs := []OutputCell{
				{Values: Params{"", fake.ErrActual}, Times: 2},
				{Values: Params{info1, nil}},
			}
			patches := ApplyMethodSeq(e, "Retrieve", outputs)
			defer patches.Reset()
			output, err := e.Retrieve("")
			So(err, ShouldEqual, fake.ErrActual)
			output, err = e.Retrieve("")
			So(err, ShouldEqual, fake.ErrActual)
			output, err = e.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info1)
		})

		Convey("batch operations failed on the third time", func() {
			info1 := "hello gomonkey"
			outputs := []OutputCell{
				{Values: Params{info1, nil}, Times: 2},
				{Values: Params{"", fake.ErrActual}},
			}
			patches := ApplyMethodSeq(e, "Retrieve", outputs)
			defer patches.Reset()
			output, err := e.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info1)
			output, err = e.Retrieve("")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, info1)
			output, err = e.Retrieve("")
			So(err, ShouldEqual, fake.ErrActual)
		})

	})
}
```

**示例**

```golang
package test

import (
	"testing"

	. "github.com/agiledragon/gomonkey/v2"
	"github.com/agiledragon/gomonkey/v2/test/fake"
	. "github.com/smartystreets/goconvey/convey"
)

func TestApplyMethod(t *testing.T) {
	slice := fake.NewSlice()
	var s *fake.Slice
	Convey("TestApplyMethod", t, func() {

		Convey("for succ", func() {
			err := slice.Add(1)
			So(err, ShouldEqual, nil)
			patches := ApplyMethod(s, "Add", func(_ *fake.Slice, _ int) error {
				return nil
			})
			defer patches.Reset()
			err = slice.Add(1)
			So(err, ShouldEqual, nil)
			err = slice.Remove(1)
			So(err, ShouldEqual, nil)
			So(len(slice), ShouldEqual, 0)
		})

		Convey("for already exist", func() {
			err := slice.Add(2)
			So(err, ShouldEqual, nil)
			patches := ApplyMethod(s, "Add", func(_ *fake.Slice, _ int) error {
				return fake.ErrElemExsit
			})
			defer patches.Reset()
			err = slice.Add(1)
			So(err, ShouldEqual, fake.ErrElemExsit)
			err = slice.Remove(2)
			So(err, ShouldEqual, nil)
			So(len(slice), ShouldEqual, 0)
		})

		Convey("two methods", func() {
			err := slice.Add(3)
			So(err, ShouldEqual, nil)
			defer slice.Remove(3)
			patches := ApplyMethod(s, "Add", func(_ *fake.Slice, _ int) error {
				return fake.ErrElemExsit
			})
			defer patches.Reset()
			patches.ApplyMethod(s, "Remove", func(_ *fake.Slice, _ int) error {
				return fake.ErrElemNotExsit
			})
			err = slice.Add(2)
			So(err, ShouldEqual, fake.ErrElemExsit)
			err = slice.Remove(1)
			So(err, ShouldEqual, fake.ErrElemNotExsit)
			So(len(slice), ShouldEqual, 1)
			So(slice[0], ShouldEqual, 3)
		})

		Convey("one func and one method", func() {
			err := slice.Add(4)
			So(err, ShouldEqual, nil)
			defer slice.Remove(4)
			patches := ApplyFunc(fake.Exec, func(_ string, _ ...string) (string, error) {
				return outputExpect, nil
			})
			defer patches.Reset()
			patches.ApplyMethod(s, "Remove", func(_ *fake.Slice, _ int) error {
				return fake.ErrElemNotExsit
			})
			output, err := fake.Exec("", "")
			So(err, ShouldEqual, nil)
			So(output, ShouldEqual, outputExpect)
			err = slice.Remove(1)
			So(err, ShouldEqual, fake.ErrElemNotExsit)
			So(len(slice), ShouldEqual, 1)
			So(slice[0], ShouldEqual, 4)
		})
	})
}
```

**示例**

```golang
package test

import (
	"testing"

	"github.com/agiledragon/gomonkey/v2/test/fake"

	. "github.com/smartystreets/goconvey/convey"

	. "github.com/agiledragon/gomonkey/v2"
)

func TestApplyPrivateMethod(t *testing.T) {
	Convey("TestApplyPrivateMethod", t, func() {
		Convey("patch private pointer method in the different package", func() {
			f := new(fake.PrivateMethodStruct)
			var s *fake.PrivateMethodStruct
			patches := ApplyPrivateMethod(s, "ok", func(_ *fake.PrivateMethodStruct) bool {
				return false
			})
			defer patches.Reset()
			result := f.Happy()
			So(result, ShouldEqual, "unhappy")
		})

		Convey("patch private value method in the different package", func() {
			s := fake.PrivateMethodStruct{}
			patches := ApplyPrivateMethod(s, "haveEaten", func(_ fake.PrivateMethodStruct) bool {
				return true
			})
			defer patches.Reset()
			result := s.AreYouHungry()
			So(result, ShouldEqual, "I am full")
		})

		Convey("repeat patch same method", func() {
			var s *fake.PrivateMethodStruct
			patches := ApplyPrivateMethod(s, "ok", func(_ *fake.PrivateMethodStruct) bool {
				return false
			})
			result := s.Happy()
			So(result, ShouldEqual, "unhappy")

			patches.ApplyPrivateMethod(s, "ok", func(_ *fake.PrivateMethodStruct) bool {
				return true
			})
			result = s.Happy()
			So(result, ShouldEqual, "happy")

			patches.Reset()
			result = s.Happy()
			So(result, ShouldEqual, "unhappy")
		})
	})

}
```