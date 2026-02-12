# 通用单元测试方法及案例

**示例**

```golang
package calculator

import (
    "fmt"
    "testing"
)

func TestCalculator(t *testing.T) {
    t.Run("TestAddition", func(t *testing.T) {
        tests := []struct {
            name string
            a, b int
            want int
        }{
            {"Positive numbers", 2, 3, 5},
            {"Negative numbers", -2, -3, -5},
            {"Mixed numbers", -2, 5, 3},
            {"Zero addition", 0, 10, 10},
        }

        for _, tt := range tests {
            t.Run(fmt.Sprintf("Add: %s", tt.name), func(t *testing.T) {
                got := Add(tt.a, tt.b)
                if got != tt.want {
                    t.Errorf("Add(%d, %d) = %d; want %d", tt.a, tt.b, got, tt.want)
                }
            })
        }
    })
}
```