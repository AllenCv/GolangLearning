# 乘法口诀表

### 代码实现一个乘法口诀表

```go
package main

import "fmt"

func mul(from,to int) {
	for i := from; i <= to; i++ {
		for j := from; j <= i; j++ {
			fmt.Printf("%d x %d = %2d ", j, i, j*i)
		}
	fmt.Println()
	}
}

func main() {
	mul(1,10)
}

```

#### 输出：

> 1 x 1 =  1 
> 1 x 2 =  2 2 x 2 =  4 
> 1 x 3 =  3 2 x 3 =  6 3 x 3 =  9 
> 1 x 4 =  4 2 x 4 =  8 3 x 4 = 12 4 x 4 = 16 
> 1 x 5 =  5 2 x 5 = 10 3 x 5 = 15 4 x 5 = 20 5 x 5 = 25 
> 1 x 6 =  6 2 x 6 = 12 3 x 6 = 18 4 x 6 = 24 5 x 6 = 30 6 x 6 = 36 
> 1 x 7 =  7 2 x 7 = 14 3 x 7 = 21 4 x 7 = 28 5 x 7 = 35 6 x 7 = 42 7 x 7 = 49 
> 1 x 8 =  8 2 x 8 = 16 3 x 8 = 24 4 x 8 = 32 5 x 8 = 40 6 x 8 = 48 7 x 8 = 56 8 x 8 = 64 
> 1 x 9 =  9 2 x 9 = 18 3 x 9 = 27 4 x 9 = 36 5 x 9 = 45 6 x 9 = 54 7 x 9 = 63 8 x 9 = 72 9 x 9 = 81 



## 优化的问题

这个乘法口诀表虽然已经实现了，但是打印出来的表格仅仅只是用%2d来设置了结果的宽度，一旦结果超过三位数，这个表格将不堪入目，为了实现整洁美观的结果，一直在对传入的两个参数、结果使用占位符宽度进行调整，结果依然无法对齐，达不到期待的结果。请教老师该使用什么方法进行处理，给我分析了问题和解决问题的方法：

- 在乘法口诀表中，要传入两个参数，在这两个参数中，第二个参数明显要比第一个参数大，那么第二个参数的2次方就是公式结果的最大位数。
  - 如 mul(1,10)，那么10 * 10 = 100，这个结果为三位数，它就是最大位数。
  - 所以公式的字符串应该为：%2d * %2d = %3d。
  - 通过字符串来组合这个模板，要实现动态指定位数，就得拿到第二个参数的位数跟结果的位数。
  - 最后打印的代码应该是：fmt.Printf(“%2d * %2d = %3d”, i, j, i * j )



## 解决

1. 要动态指定这个第二个和第三个参数位数跟结果的位数，就对输入的的数字进行判断长度。

   - 通过Google `golang 如何获取整型的长度`，查找到了资料 https://segmentfault.com/q/1010000011754972

   > 文章中说贴了解决方案是先将数字转为字符串，在计算出字符串的长度。
   >
   > 使用了内置方法 `strconv.Itoa(int)`，再使用`len()`获得长度。
   >
   > 但是使用该方法代码会啰嗦，效率不高，因此使用math.Log10()方法分别对参数处理，然后拼接输出模板。

2. 分别对三个参数进行对齐操作，将三个模板拼接。

   ```go
   // 获取参数的位数
   func getDigit(number float64) int {
   	return int(math.Log10(number)) + 1
   }
   
   // 第二个参数的位数
   argDigit := getDigit(float64(number))
   // 第三个参数的位数，这里的math.Pow方法
   maxDigit := getDigit(math.Pow(float64(number), 2))
   
   arg1 := "%" + fmt.Sprintf("%d", argDigit) + "d"
   arg2 := "%-" + fmt.Sprintf("%d", argDigit) + "d"
   result := "%-" + fmt.Sprintf("%d", maxDigit) + "d"
   format := arg1 + " + " + arg2 + " = " + result  
   }
   ```

   > “%”：占位符，默认为右对齐。
   >
   > "%-2d"：表示该占位符的宽度为2，负号的作用是对齐方式为左对齐。
   >
   > `func Log10(x float64) float64`
   >
   > ​	Log10返回x的十进制对数。 特殊情况与Log相同。
   >
   > `func Pow(x, y float64) float64`
   >
   > ​	Pow返回x ** y，也就是 x 的 y 次方，如: math.pow(10,2) ，结果就是100
   >
   > 这里使用math.Log10()的方法，获得第二个参数的对数

3. 最后根据需求添加对用户输入的操作符进行做加法，乘法运算。

4. 整体代码如下

   ```go
   package main
   
   import (
   	"fmt"
   	"math"
   )
   
   func normative(oper string,number int) string {
   	// 监测 to 的位数
   	argDigit := getDigit(float64(number))
   
   	// math.Pow() 参数的 2 次方
   	maxDigit := getDigit(math.Pow(float64(number), 2))
   
   	// 参数和结果格式模板
   	// 左侧参数向右对齐
   	arg1 := "%" + fmt.Sprintf("%d", argDigit) + "d"
   
   	// 右侧参数向左对齐
   	arg2 := "%-" + fmt.Sprintf("%d", argDigit) + "d"
   	// 结果向左对齐
   	result := "%-" + fmt.Sprintf("%d", maxDigit) + "d"
   
   	// 公式模板
   	if oper == "+" {
   		format := arg1 + " + " + arg2 + " = " + result
   		return format
   	} else if oper == "*" {
   		format := arg1 + " * " + arg2 + " = " + result
   		return format
   	} else {
   		return "没有这个模板"
   	}
   }
   
   func getDigit(number float64) int {
   	return int(math.Log10(number)) + 1
   }
   
   func calc(operator string, from, to int) {
   	result := normative(operator,to)
   	if operator == "+" {
   		for i := from; i <= to; i++ {
   			for j := from; j <= i; j++ {
   				fmt.Printf(result, j, i, i + j)
   			}
   			fmt.Println()
   		}
   	} else if operator == "*" {
   		for i := from; i <= to; i++ {
   			for j := from; j <= i; j++ {
   				fmt.Printf(result, j, i, i*j)
   			}
   			fmt.Println()
   		}
   	} else {
   		fmt.Printf("请输入正确的操作符.")
   	}
   }
   
   func main() {
   	calc("*", 1, 100)
     calc("+", 1, 100)
   }
   
   ```

   





