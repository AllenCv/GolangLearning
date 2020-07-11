# 猜数字游戏

让用户猜程序给定的随机数1-100
让用户在终端输入数字进行比较
每猜一次提示用户与答案相比大了还是小了.
记录用户一共猜了多少次
用户输入exit会退出程序
用户猜中随机数也退出程序

## 一、伪代码

初始化一个随机数

初始化计数器

开始循环

计数器自增

让用户输入一个数字
对数字做必要的处理

如果是exit就退出

如果相等就猜对退出

如果大于小于就继续循环

## 二、 代码实现

math/rand包提供了生成随机数的方法，通过`rand.Intn()`方法生成的是一个伪随机数，随机数都需要一个随机种子来生成随机数，这里默认的seed为1，因为伪随机数的数字是固定的，所以只要执行的代码一样，生成的结果就会一样。

1. 为了实现真正的随机性，那就必须保证seed每次都不同，世界上拥有唯一性，不可逆的就是时间，所以这里使设置时间戳为随机种子`rand.Seed(time.Now().UnixNano())`

   > - 通俗的讲，时间戳是一份能够表示一份数据在一个特定时间点已经存在的完整的可验证的数据。它的提出主要是为用户提供一份电子证据，以证明用户的某些数据的产生时间。
   > - 时间戳服务的本质是将用户的数据和当前准确时间绑定，在此基础上用时间戳系统的数字证书进行签名，凭借时间戳系统在法律上的权威授权地位，产生可用于法律证据的时间戳，用来证明用户数据的产生时间，达到“不可否认”或“抗抵赖”的目标

2. 让用户输入一个数字，这个就得进行交互，但是怎样来读取用户的输入呢？Go语言提供`os.Stdin`方法可以得到用户输入。

3. 得到用户输入后，程序需要将输入读取，`bufio`包实现对数据读写的缓冲，它封装了`io.Reader`和`io.Writer`对象。**使用这个包可以大幅提高文件读写的效率**。

4. 使用`bufio`包读取文件，使用`bufio.NewReader()`初始化为一个 Reader对象，存在缓冲区中，

   ```go
   func NewReader(rd io.Reader) *Reader {
         // 默认缓存大小：defaultBufSize=4096
         return NewReaderSize(rd, defaultBufSize)
   }
   ```

   > 简单的说就是，把文件读取进缓冲（内存）之后再读取的时候就可以避免文件系统的io  从而提高速度。同样的，在进行写操作时，先把文件写入缓冲（内存），然后由缓冲写入文件系统。直接把  内容->文件 和 内容->缓冲->文件相比，缓冲区的设计是为了存储多次的写入，最后在把缓冲区的内容写入文件，这样就减少了不断对文件的读取写入，提高效率。

5. 用户输入被写入了缓冲区，需要将内容读取出来，使用`ReadString`对输入进行读取。

   ```go
   func (b *Reader) ReadString(delim byte) (line string, err error) {
          bytes, err := b.ReadBytes(delim)
          return string(bytes), err
   }
   ```

   >  ReadString 从输入中读取，直到遇到第一个界定符（delim）为止，返回一个指向缓存中字节的 line

   ```go
   text, err := reader.ReadString('\n')
   if err != nil{
     fmt.Println(err)
     break
   }
   
   // 用户输入 exit 退出游戏，reader.ReadString读取为：exit\n ，要将\n替换掉。
   text = strings.Replace(text, "\n", "", -1)
   if text == "text" {
     fmt.Println("退出游戏")
     break
   }
   ```

   > 对`text`进行判断，如果输入的是`exit`就中止循环，如果输入的数字就对数字进行判断是否大于、小于或者相等。
   >
   > **判断逻辑：**
   >
   > 1. 首先要定义中断条件，优先将中断条件写在最前面，尽早中断。
   >
   >  	2. 尽早中断可以有效避免if嵌套，if嵌套多了会使代码逻辑混乱且不易于阅读，所以尽早中断可以实现整洁/易读的代码。
   >  	3. 如果判断逻辑仅仅只是if...else，遵循尽早中断条件，将中断条件写在前面，那么else可以不用写，直接返回正确结果。

6. 最终代码：

   ```go
   package main
   
   import (
   	"bufio"
   	"fmt"
   	"math/rand"
   	"os"
   	"strconv"
   	"strings"
   	"time"
   )
   
   func main() {
   	rand.Seed(time.Now().UnixNano())
   	randomNumber := rand.Intn(100) + 1
   	count := 0
   	reader := bufio.NewReader(os.Stdin)
   	fmt.Printf("请输入一个数字开始游戏\n> ")
   	for {
   		text, err := reader.ReadString('\n')
   		if err != nil {
   			fmt.Println(err)
   			break
   		}
   		text = strings.Replace(text, "\n", "",-1)
       
   		if text == "exit" {
   			fmt.Println("退出程序")
   			break
   		}
   
   		number, err := strconv.Atoi(text)
   		if err != nil{
   			fmt.Println("输入错误.")
   			continue
   		}
   
   		count ++
   
   
   		if number == randomNumber {
   			fmt.Printf("你输入的数字是: %d 在第 %d 次答对了。\n", number, count)
   			break
   		}
   		if number < randomNumber {
   			fmt.Printf("你输入的数字是: %d 但小于生成的数字,请重新输入.\n", number)
   			continue
   		}
   		fmt.Printf("你输入的数字是: %d 但大于生成的数字,请重新输入.\n", number)

   	}
   
   }
   ```
   
   

