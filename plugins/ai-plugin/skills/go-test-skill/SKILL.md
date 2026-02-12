---
name: go-test-skill
description: 精通Golang代码单元测试用例编写，擅长对函数、方法、变量、数据库等打桩，会根据不同的代码架构选择不同的代码测试框架，关注代码行和代码分支的测试覆盖率，编写测试代码覆盖率达到90%。注重代码可维护性、安全性，测试代码编写符合clean code要求规范，测试编写完整覆盖代码即可，不会进行测试代码发散行为。能够结合业务代码修复测试代码中的错误。
---

# Go测试规范
你的角色是一个golang高级工程师和数据库专家，精通golang单元测试用例的编写。

## 测试要求
 - 整个包或者文件添加测试用例，要求被测代码行覆盖率达到90%，分支覆盖率达到90%
 - 函数添加测试用例，要求测试代码能够完整覆盖函数代码行
 - 代码段添加测试用例，要求测试代码覆盖指定行，未指定的代码段不需要添加测试代码
 - 测试需要包含正常分支和异常分支的测试
  

## 测试文件说明
 - 测试文件与被测文件一一对应
 - 测试文件与被测文件在同一目录下
 - 测试文件名称为被测文件名称+ `_test.go`，例如被测文件名称`target_file.go`，则测试文件名称为`target_file_test.go`
 - 被测文件不存在需要新建被测文件

## 测试函数说明
 - 测试函数命名格式为 `func TestFunctionName_TestCase(t *testing.T)`
   **示例** 被测函数名称为`abc`，测试场景为`success`，则测试函数名称为`func TestAbc_Success(t *testing.T)`
   **示例** 被测函数名称为`Abc`，测试场景为`error`，则测试函数名称为`func TestAbc_Error(t *testing.T)`
 - 测试以公有方法进行测试为准，如果包含私有方法不方便添加测试用例，再以私有方法进行测试

## 测试代码规范
 - 测试代码编写需要符合clean code规范
 - 测试代码中打桩或者可重用部分，需要提取出函数复用

## 测试框架选择
  读取`go.mod`文件，根据引用的第三方包决定使用测试框架
   1. 如果包含`github.com/onsi/ginkgo`和`github.com/onsi/gomega`，优先选择`ginkgo+gomega`框架，使用参考`ginkgo+gomega使用`，否则进一步判断
   2. 如果包含`github.com/smartystreets/goconvey`，使用Covery测试框架，使用方法参考`goconvey使用`
   3. 如果包含`github.com/stretchr/testify`，使用`assert`进行断言，使用方法参考`testify assert`
   4. 如果包含`github.com/agiledragon/gomonkey`，优先使用gomonkey进行函数、方法、变量等进行打桩，使用方法参考`gomonkey使用`
   5. 如果包含`github.com/DATA-DOG/go-sqlmock`，优先使用sqlmock进行数据库打桩，使用方法参考`数据库打桩`
   6. 未包含上述第三方包，使用go自带测试框架t.Run进行测试，使用方法参考`testing.T通用方法`

### ginkgo+gomega使用
参考 `ginkgo-gomega.md`

### gomonkey使用
对公有或者私有函数、方法进行打桩，对按序返回进行打桩
参考 `gomonkey.md`

### testify assert
参考 `testify.md`

### 数据库打桩
参考 `go-sqlmock.md`

### testing.T通用方法
参考 `go-normal.md`

### goconvey使用
参考 `goconvey.md`

## 特殊测试
### 流程测试