# 从 Hello World 到嵌入式：C、C++、数据结构与硬件基础

> 面向对象：从零开始学习编程和嵌入式开发的人  
> 学习目标：能写、能调、能解释，而不是只记住术语  
> 推荐标准：PC 端使用 C17、C++20；单片机项目以芯片工具链实际支持版本为准

代码说明：

- 标明完整头文件和 main 的 PC 示例可单独创建文件编译；
- 只展示某个函数或结构的片段，需要与相应头文件、调用代码组合；
- gpio_write、enter_critical 等名称是平台相关接口示意，必须替换为实际芯片 SDK/HAL；
- 寄存器地址均以具体芯片数据手册和厂商头文件为准，不能照抄教学地址。

快速导航：

| 部分 | 章节 | 内容 |
|---|---:|---|
| 第一部分 | 0–10 | 运行原理、环境、C Hello World 与基本语法 |
| 第二部分 | 11–24 | C 数组、字符串、指针、内存、模块和 Embedded C |
| 第三部分 | 25–39 | C++ 类、RAII、拷贝移动、模板、STL 与嵌入式 C++ |
| 第四部分 | 40–54 | 复杂度、线性结构、树、堆、图、查找和排序 |
| 第五部分 | 55–86 | 电路、MCU、外设、通信、PCB、仪器和综合实验 |

---

## 0. 学习方法与路线

这份文档按以下顺序组织：

~~~text
电脑如何运行程序
    ↓
C 语言基础与内存
    ↓
C++ 对象、资源与标准库
    ↓
数据结构与算法
    ↓
电路、数字逻辑与 MCU
    ↓
GPIO / 中断 / 定时器 / ADC / DMA / 通信总线
    ↓
PCB、仪器、联调与综合项目
~~~

每个知识点用四步掌握：

1. **看懂**：知道它解决什么问题。
2. **写出**：关闭文档，重新写最小例子。
3. **验证**：编译、运行、打断点或测波形。
4. **说清**：用“定义—原理—例子—边界”口述。

不要只复制代码。每段代码至少做三次改动：

- 改输入；
- 故意制造错误；
- 增加边界条件。

### 0.1 四个学习阶段

| 阶段 | 目标 | 通过标准 |
|---|---|---|
| 入门 | 能编译并理解基本语法 | 独立完成命令行计算器 |
| 基础 | 理解指针、内存、类和常用容器 | 独立写环形缓冲区 |
| 进阶 | 掌握复杂度、数据结构和模块化 | 能解释并实现常用结构 |
| 嵌入式 | 理解电路、MCU 外设和调试 | 完成一个可测量的小系统 |

### 0.2 遇到不会时怎么查

按以下优先级：

1. 编译器错误和警告；
2. 语言参考资料；
3. 芯片数据手册、参考手册、勘误；
4. 官方库和示例；
5. 可信书籍；
6. 博客与视频只作为辅助。

---

# 第一部分：电脑与第一个程序

## 1. 程序到底是什么

程序源代码是人能阅读的文本；CPU 最终执行的是机器指令。

典型过程：

~~~text
源文件 .c/.cpp
→ 预处理
→ 编译
→ 汇编
→ 目标文件 .o/.obj
→ 链接
→ 可执行文件 .exe/ELF
→ 操作系统加载
→ CPU 执行
~~~

在单片机中，链接后还常生成 HEX 或 BIN，再通过下载器写入 Flash。

### 1.1 CPU、内存和外设

- **CPU**：取指、译码、执行。
- **寄存器**：CPU 内部非常快的小型存储。
- **RAM**：运行时保存变量、栈和缓冲区，掉电通常丢失。
- **Flash**：保存程序和常量，掉电不丢失。
- **外设**：GPIO、定时器、ADC、UART 等硬件模块。

### 1.2 bit、byte 与十六进制

- bit：一个二进制位，取值 0 或 1；
- byte：字节，在常见 PC 和 STM32 上为 8 bit；
- 1 KiB = 1024 byte；
- 十六进制每一位表示 4 bit。

~~~text
十进制 42
= 二进制 0010 1010
= 十六进制 0x2A
~~~

嵌入式经常使用十六进制，因为寄存器位分布一眼更清楚。

### 1.3 操作系统程序与裸机程序

PC 程序：

- 操作系统负责进程、文件、终端和内存保护；
- printf 最终依赖操作系统输出。

裸机 MCU：

- 没有完整操作系统；
- 上电后从复位向量开始；
- GPIO 和串口要自己初始化；
- printf 若要使用，需要重定向到 UART、SWO 等输出。

---

## 2. 准备编译环境

Windows 可选择：

- Visual Studio Community；
- MSYS2/MinGW-w64；
- LLVM/Clang；
- WSL 中的 GCC。

嵌入式常用：

- Keil MDK；
- STM32CubeIDE；
- GCC Arm Embedded；
- 芯片厂商 IDE。

本文命令行示例使用 GCC 风格。检查：

~~~powershell
gcc --version
g++ --version
~~~

如果命令不存在，说明尚未安装或没有加入 PATH。

### 2.1 推荐的编译选项

C：

~~~powershell
gcc hello.c -std=c17 -Wall -Wextra -Wpedantic -O0 -g -o hello.exe
.\hello.exe
~~~

C++：

~~~powershell
g++ hello.cpp -std=c++20 -Wall -Wextra -Wpedantic -O0 -g -o hello_cpp.exe
.\hello_cpp.exe
~~~

含义：

- -std：选择语言标准；
- -Wall -Wextra -Wpedantic：开启常用警告；
- -O0：暂不优化，便于初学调试；
- -g：生成调试信息；
- -o：指定输出文件名。

警告不是装饰。初学阶段应尽量做到“零警告编译”。

---

## 3. C 语言 Hello World

新建 hello.c：

~~~c
#include <stdio.h>

int main(void)
{
    printf("Hello, world!\n");
    return 0;
}
~~~

### 3.1 每一行是什么意思

**#include <stdio.h>**

把标准输入输出相关声明提供给当前源文件。printf 的声明位于 stdio.h。

**int main(void)**

- main 是程序入口；
- int 表示返回整数状态码；
- void 表示不接收普通参数；
- 返回 0 通常表示成功。

不要写 void main。标准托管环境下常用形式是：

~~~c
int main(void)
~~~

或：

~~~c
int main(int argc, char *argv[])
~~~

**花括号**

定义代码块和作用域。

**分号**

结束大多数语句。

**\n**

换行转义字符，不是两个普通字符。

### 3.2 printf 做了什么

printf 是格式化输出函数：

~~~c
int age = 23;
printf("age = %d\n", age);
~~~

格式说明符必须与参数类型匹配，否则可能产生未定义行为。

常见格式：

| 类型 | 常见格式 |
|---|---|
| int | %d |
| unsigned int | %u |
| long | %ld |
| unsigned long | %lu |
| double | %f |
| char | %c |
| C 字符串 | %s |
| 指针 | %p，并转换为 void* |

对于 uint32_t 等固定宽度类型，最严谨的可移植写法使用 inttypes.h 中的格式宏。

### 3.3 第一次练习

修改程序输出：

- 姓名；
- 学校；
- 目标岗位；
- 三行内容。

再故意删除：

- 分号；
- 右括号；
- 头文件。

观察编译器如何定位错误。

---

## 4. C 的词法与基本规则

### 4.1 标识符

变量、函数和类型的名字称为标识符：

~~~c
int student_age;
float motor_speed;
~~~

规则：

- 由字母、数字、下划线等组成；
- 不能以数字开头；
- 不能使用关键字；
- 区分大小写；
- 避免以下划线加大写字母等保留形式命名。

推荐：

- 变量和函数：snake_case；
- 宏：UPPER_SNAKE_CASE；
- 类型：项目内保持一致。

### 4.2 注释

~~~c
// 单行注释

/*
 * 多行注释
 */
~~~

好注释解释“为什么”，不是重复代码：

~~~c
// 错：把 count 加 1
count++;

// 好：预留一个槽位，用 head == tail 表示空
head = (head + 1U) % CAPACITY;
~~~

### 4.3 语句、表达式和代码块

~~~c
int x = 3;          // 声明语句
x = x + 1;          // 表达式语句

if (x > 0) {        // 复合语句/代码块
    printf("%d\n", x);
}
~~~

---

## 5. 变量、常量与基本类型

### 5.1 变量

~~~c
int count = 0;
float voltage = 3.3f;
char grade = 'A';
~~~

变量包含：

- 类型；
- 名字；
- 值；
- 存储位置；
- 生命周期；
- 作用域。

先初始化再使用：

~~~c
int value = 0;
~~~

未初始化的自动局部变量具有不确定值，读取它可能造成未定义行为。

### 5.2 整数类型

C 提供：

- char；
- short；
- int；
- long；
- long long；
- 对应 unsigned 版本。

标准只保证最小范围和相对大小，不保证 int 在所有平台恰好 32 位。

协议、寄存器和文件格式优先使用 stdint.h：

~~~c
#include <stdint.h>

uint8_t command;
int16_t temperature;
uint32_t timestamp_ms;
~~~

### 5.3 signed 与 unsigned

unsigned 适合：

- 位掩码；
- 明确不为负的计数；
- 模运算；
- 寄存器。

不要因为数值不为负就无脑使用 unsigned。混合比较可能违反直觉：

~~~c
int32_t a = -1;
uint32_t b = 1U;

if (a < b) {
    // 在常见转换规则下，条件可能为假
}
~~~

原则：

- 同一计算尽量使用相同符号类别；
- 转换前确认范围；
- 编译器警告全部查看；
- 有符号溢出不能依赖。

### 5.4 浮点类型

- float：通常单精度；
- double：通常双精度；
- long double：实现相关。

~~~c
float pi_f = 3.1415926f;
double pi_d = 3.141592653589793;
~~~

浮点数不能表示所有实数：

~~~c
double x = 0.1 + 0.2;
// x 不一定精确等于 0.3
~~~

常按误差判断：

~~~c
#include <math.h>

if (fabs(x - 0.3) < 1e-9) {
    /* 足够接近 */
}
~~~

误差阈值应结合量级和业务，而不是永远使用同一个常数。

### 5.5 char 与字符串

单个字符用单引号：

~~~c
char letter = 'A';
~~~

字符串用双引号，并以空字符 '\0' 结束：

~~~c
char name[] = "Xu";
// 内存为 'X' 'u' '\0'
~~~

'A' 与 "A" 不同：

- 'A' 是一个字符常量；
- "A" 是含 'A' 和 '\0' 的字符串字面量。

### 5.6 bool

C 使用 stdbool.h：

~~~c
#include <stdbool.h>

bool is_ready = false;
is_ready = true;
~~~

条件判断中 0 为假，非 0 为真，但逻辑变量仍建议使用 bool 表达意图。

### 5.7 sizeof

~~~c
printf("%zu\n", sizeof(int));
~~~

- sizeof 是运算符，不是普通函数；
- 返回 size_t；
- sizeof(char) 永远为 1；
- 具体字节数由平台决定。

不要凭记忆写死类型大小，实际用 sizeof 验证。

### 5.8 const

~~~c
const float supply_voltage = 3.3f;
~~~

const 表示不应通过该左值修改对象，有助于表达接口约束。它不保证对象一定存于 Flash。

---

## 6. 输入与输出

### 6.1 printf

~~~c
int x = 10;
double y = 2.5;
printf("x=%d, y=%.2f\n", x, y);
~~~

%.2f 表示小数点后显示两位，不表示内部只保存两位。

### 6.2 为什么初学不推荐直接 scanf("%s")

~~~c
char name[8];
scanf("%s", name);   // 输入太长可能越界
~~~

更容易控制的方式是 fgets：

~~~c
#include <stdio.h>
#include <string.h>

int main(void)
{
    char line[64];

    printf("请输入姓名：");
    if (fgets(line, sizeof line, stdin) == NULL) {
        return 1;
    }

    line[strcspn(line, "\r\n")] = '\0';
    printf("你好，%s\n", line);
    return 0;
}
~~~

### 6.3 字符串转整数

比 atoi 更容易检查错误的方式是 strtol：

~~~c
#include <errno.h>
#include <limits.h>
#include <stdio.h>
#include <stdlib.h>

int main(void)
{
    char line[64];

    if (fgets(line, sizeof line, stdin) == NULL) {
        return 1;
    }

    char *end = NULL;
    errno = 0;
    long value = strtol(line, &end, 10);

    if (errno != 0 || end == line || value < INT_MIN || value > INT_MAX) {
        printf("输入无效\n");
        return 1;
    }

    printf("value=%ld\n", value);
    return 0;
}
~~~

真实程序还要检查 end 后是否只剩空白或换行。

---

## 7. 运算符

### 7.1 算术

~~~c
int a = 7;
int b = 3;

int sum = a + b;
int difference = a - b;
int product = a * b;
int quotient = a / b;   // 整数除法，结果为 2
int remainder = a % b;  // 余数为 1
~~~

浮点转换：

~~~c
double quotient = (double)a / b;  // 约 2.3333
~~~

### 7.2 比较与逻辑

~~~c
a == b
a != b
a < b
a <= b
a > b
a >= b
~~~

逻辑：

~~~c
condition_a && condition_b
condition_a || condition_b
!condition
~~~

&& 和 || 短路求值：

~~~c
if (pointer != NULL && *pointer > 0) {
    /* 只有指针非空才解引用 */
}
~~~

### 7.3 赋值与自增

~~~c
x += 2;
x -= 2;
x *= 2;
x /= 2;
x++;
++x;
~~~

单独一行使用时 x++ 与 ++x 最终都加一；嵌入复杂表达式时值不同，也更容易产生错误。建议避免写难以阅读的副作用表达式。

### 7.4 优先级

不要靠背完整优先级表。用括号表达意图：

~~~c
result = (a + b) * c;
~~~

经典错误：

~~~c
if (flags & MASK == 0U) {
    /* 优先级可能不是想象的那样 */
}
~~~

应写：

~~~c
if ((flags & MASK) == 0U) {
}
~~~

---

## 8. 分支与循环

### 8.1 if / else

~~~c
if (temperature > 80) {
    printf("过热\n");
} else if (temperature < 0) {
    printf("过冷\n");
} else {
    printf("正常\n");
}
~~~

即使只有一行，也建议保留花括号。

### 8.2 switch

~~~c
switch (command) {
case 0:
    stop_motor();
    break;
case 1:
    start_motor();
    break;
default:
    report_invalid_command();
    break;
}
~~~

忘记 break 会贯穿到下一分支。若故意贯穿，应明确注释或使用编译器支持的属性。

### 8.3 for

~~~c
for (int i = 0; i < 10; ++i) {
    printf("%d\n", i);
}
~~~

避免 off-by-one：

- 长度为 n 的数组，下标是 0 到 n-1；
- 循环常用 i < n，而不是 i <= n。

### 8.4 while 与 do-while

~~~c
while (condition) {
    work();
}
~~~

~~~c
do {
    work();
} while (condition);
~~~

do-while 至少执行一次。

### 8.5 break 与 continue

- break：退出最近一层循环或 switch；
- continue：跳过本轮剩余语句。

不要用多层嵌套和大量 break/continue 把控制流写得不可读。复杂逻辑可提取函数或状态机。

### 8.6 非阻塞思想

PC 入门常写 delay；嵌入式主循环不应为每件事长时间等待：

~~~c
for (;;) {
    read_buttons();
    update_state_machine();
    process_uart();
    update_outputs();
}
~~~

每一步快速返回，让多个功能轮流推进。

---

## 9. 函数、参数与作用域

### 9.1 定义函数

~~~c
int add(int a, int b)
{
    return a + b;
}
~~~

函数应做到：

- 名字表达目的；
- 输入输出清楚；
- 只完成相对单一任务；
- 处理错误；
- 避免偷偷修改大量全局状态。

### 9.2 声明与定义

声明告诉编译器函数接口：

~~~c
int add(int a, int b);
~~~

定义提供函数体：

~~~c
int add(int a, int b)
{
    return a + b;
}
~~~

### 9.3 C 是值传递

~~~c
void set_to_zero(int value)
{
    value = 0;
}
~~~

调用后原变量不变，因为函数获得的是副本。

要修改调用者对象，传指针：

~~~c
#include <stdbool.h>

bool set_to_zero(int *value)
{
    if (value == NULL) {
        return false;
    }

    *value = 0;
    return true;
}
~~~

### 9.4 返回多个结果

可使用输出参数：

~~~c
bool divide(int numerator, int denominator, int *quotient, int *remainder)
{
    if (denominator == 0 || quotient == NULL || remainder == NULL) {
        return false;
    }

    *quotient = numerator / denominator;
    *remainder = numerator % denominator;
    return true;
}
~~~

也可返回 struct，通常更清晰，后文讲解。

### 9.5 作用域

~~~c
int global_value;

void example(void)
{
    int local_value = 0;

    if (local_value == 0) {
        int block_value = 1;
        /* block_value 只在此块可见 */
    }
}
~~~

全局变量方便但会增加耦合和测试难度。能通过参数传递时，优先避免随意使用全局变量。

### 9.6 static

函数内 static：

~~~c
uint32_t next_id(void)
{
    static uint32_t id = 0U;
    return ++id;
}
~~~

id 只初始化一次，生命周期贯穿程序，但作用域仍在函数内。

文件作用域 static：

~~~c
static uint32_t module_state;
static void helper(void);
~~~

表示名称只在当前源文件内部使用，有助于隐藏实现。

### 9.7 递归

函数直接或间接调用自身：

~~~c
unsigned factorial(unsigned n)
{
    if (n <= 1U) {
        return 1U;
    }
    return n * factorial(n - 1U);
}
~~~

风险：

- 每层调用消耗栈；
- 缺少终止条件会栈溢出；
- factorial 很快整数溢出；
- 资源受限 MCU 常优先迭代实现。

---

## 10. 第一阶段练习

### 10.1 温度转换

输入摄氏温度，输出华氏温度：

$$
F = C \times 9/5 + 32
$$

注意避免整数除法。

### 10.2 成绩等级

输入 0–100：

- 90–100：A；
- 80–89：B；
- 70–79：C；
- 60–69：D；
- 0–59：F；
- 超出范围报错。

### 10.3 最大值

实现：

~~~c
int max_of_three(int a, int b, int c);
~~~

### 10.4 非阻塞计时思想

给定 now_ms、start_ms、timeout_ms，判断是否超时。考虑无符号计数回绕：

~~~c
bool has_timed_out(uint32_t now_ms, uint32_t start_ms, uint32_t timeout_ms)
{
    return (uint32_t)(now_ms - start_ms) >= timeout_ms;
}
~~~

### 10.5 第一阶段口述题

1. 源代码如何变成可执行文件？
2. int 的大小是否永远为 4？
3. 为什么整数 7/3 得到 2？
4. == 与 = 有什么区别？
5. C 是值传递是什么意思？
6. static 局部变量放在哪里、活多久？
7. 为什么嵌入式循环要避免长时间阻塞？

---

# 第二部分：C 语言核心与嵌入式写法

## 11. 数组

### 11.1 一维数组

~~~c
int scores[5] = {90, 85, 78, 92, 88};
~~~

访问：

~~~c
scores[0] = 95;
printf("%d\n", scores[4]);
~~~

合法下标为 0 到 4。访问 scores[5] 越界，属于未定义行为。

### 11.2 计算元素个数

~~~c
size_t count = sizeof scores / sizeof scores[0];
~~~

这个方法只在 scores 仍是数组的作用域有效。传入函数后，数组形参会调整为指针：

~~~c
void print_array(const int values[], size_t count)
{
    for (size_t i = 0; i < count; ++i) {
        printf("%d\n", values[i]);
    }
}
~~~

函数必须额外接收长度。

### 11.3 求平均值

~~~c
bool average(const int *values, size_t count, double *result)
{
    if (values == NULL || result == NULL || count == 0U) {
        return false;
    }

    int64_t sum = 0;
    for (size_t i = 0; i < count; ++i) {
        sum += values[i];
    }

    *result = (double)sum / (double)count;
    return true;
}
~~~

为什么累加器用更宽类型？避免多个 int 相加过早溢出，但仍需根据最大输入和数量验证 int64_t 是否足够。

### 11.4 二维数组

~~~c
int matrix[2][3] = {
    {1, 2, 3},
    {4, 5, 6}
};
~~~

C 的多维数组按行连续存储。传参时必须知道后续维度：

~~~c
void print_matrix(size_t rows, int matrix[][3])
{
    for (size_t r = 0; r < rows; ++r) {
        for (size_t c = 0; c < 3U; ++c) {
            printf("%d ", matrix[r][c]);
        }
        printf("\n");
    }
}
~~~

---

## 12. C 字符串

### 12.1 本质

C 字符串是以 '\0' 结束的 char 数组：

~~~c
char word[] = {'H', 'i', '\0'};
char same[] = "Hi";
~~~

数组容量必须包含结尾空字符。

### 12.2 strlen 与 sizeof

~~~c
char text[] = "hello";

sizeof text;   // 通常为 6，包含 '\0'
strlen(text);  // 为 5，不包含 '\0'
~~~

strlen 会一直找 '\0'。如果数组没有正确终止，它会越界读取。

### 12.3 常用函数

头文件 string.h：

- strlen：长度；
- strcmp：比较；
- memcpy：复制不重叠内存；
- memmove：允许重叠；
- memset：按字节设置；
- strchr：查找字符；
- strstr：查找子串。

strcmp 返回：

- 小于 0；
- 等于 0；
- 大于 0。

不要假设只返回 -1、0、1。

### 12.4 安全拼接

使用 snprintf：

~~~c
char message[64];
int written = snprintf(message, sizeof message,
                       "id=%u,value=%d",
                       device_id, sensor_value);

if (written < 0) {
    /* 格式化失败 */
} else if ((size_t)written >= sizeof message) {
    /* 输出被截断 */
}
~~~

不要只看到 snprintf 就认为永远安全，必须检查返回值。

### 12.5 字符串字面量

~~~c
const char *message = "hello";
~~~

不要尝试修改字符串字面量：

~~~c
char *bad = "hello";
bad[0] = 'H';  // 未定义行为
~~~

需要可修改副本：

~~~c
char good[] = "hello";
good[0] = 'H';
~~~

---

## 13. 指针

### 13.1 指针保存地址

~~~c
int value = 42;
int *pointer = &value;

printf("%d\n", *pointer);
*pointer = 100;
~~~

- &value：取得地址；
- *pointer：解引用，访问所指对象。

### 13.2 空指针

~~~c
int *pointer = NULL;
~~~

空指针表示不指向有效对象。解引用 NULL 是错误。

任何接收指针的接口都要明确：

- 能否为 NULL；
- 指向多少个元素；
- 谁拥有对象；
- 对象活多久；
- 函数是否修改内容。

### 13.3 指针与数组

数组在许多表达式中转换为首元素指针：

~~~c
int values[3] = {10, 20, 30};
int *p = values;

printf("%d\n", p[1]);
printf("%d\n", *(p + 1));
~~~

p + 1 移到下一个 int，不是简单增加一个字节。

数组不是可修改的指针变量：

~~~c
values++;  // 错误
p++;       // p 是指针变量，可以
~~~

### 13.4 指针的 const

~~~c
const int *p1;               // 不能通过 p1 改值，p1 可改指向
int * const p2 = &value;     // p2 不可改指向，可改 value
const int * const p3 = &value; // 两者都受限
~~~

接口示例：

~~~c
bool checksum(const uint8_t *data, size_t length, uint8_t *result);
~~~

data 是只读输入，result 是输出。

### 13.5 指针的指针

~~~c
void reset_pointer(int **pointer)
{
    if (pointer != NULL) {
        *pointer = NULL;
    }
}
~~~

常见用途：

- 修改调用者的指针；
- 动态二维结构；
- 字符串数组；
- 链表头指针。

### 13.6 野指针与悬空指针

野指针：没有初始化：

~~~c
int *p;       // 不确定地址
~~~

悬空指针：原对象已失效：

~~~c
int *bad(void)
{
    int local = 10;
    return &local;  // 函数结束后 local 不存在
}
~~~

释放后也会悬空：

~~~c
free(p);
p = NULL;
~~~

设置 NULL 只能避免继续通过这个变量误用；其他指向同一内存的别名仍可能悬空。

### 13.7 void*

void* 是通用对象指针，可保存不同对象地址，但解引用前需要转换为具体类型。

malloc 返回 void*。C 中不需要也不建议为 malloc 返回值强制转换，因为错误的强转可能掩盖缺少头文件等问题。

---

## 14. 生命周期、存储期与内存布局

### 14.1 常见区域

~~~text
.text    程序指令
.rodata  只读数据
.data    有非零初值的全局/静态对象
.bss     零初始化的全局/静态对象
stack    函数调用、局部自动变量
heap     动态分配
~~~

实际布局由平台、编译器和链接脚本决定。

### 14.2 自动存储期

~~~c
void function(void)
{
    int local = 1;
}
~~~

local 通常随函数进入创建、退出失效。不要返回它的地址。

### 14.3 静态存储期

全局变量、文件作用域 static 和函数内 static 都贯穿程序生命周期：

~~~c
static int module_counter;
~~~

### 14.4 动态内存

~~~c
#include <stdlib.h>

size_t count = 100U;
int *values = malloc(count * sizeof *values);

if (values == NULL) {
    /* 申请失败 */
    return;
}

/* 使用 values */

free(values);
values = NULL;
~~~

必须处理：

- 乘法溢出；
- 申请失败；
- 内存泄漏；
- 重复释放；
- 释放后使用；
- 碎片化；
- 所有权。

检查乘法：

~~~c
if (count > SIZE_MAX / sizeof *values) {
    /* 大小溢出 */
}
~~~

### 14.5 MCU 中是否能用 malloc

不是绝对禁止，但要评估：

- 堆大小；
- 最坏申请时间；
- 碎片；
- 失败处理；
- 多任务安全；
- 长期运行稳定性。

常见策略：

- 静态分配；
- 固定大小内存池；
- 只在初始化阶段分配；
- 禁止实时关键路径频繁申请释放。

---

## 15. struct、enum、union 与 typedef

### 15.1 struct

~~~c
typedef struct {
    uint32_t timestamp_ms;
    int16_t temperature_centi;
    uint16_t voltage_mv;
    bool valid;
} SensorSample;
~~~

使用：

~~~c
SensorSample sample = {
    .timestamp_ms = 1000U,
    .temperature_centi = 2534,
    .voltage_mv = 3300U,
    .valid = true
};
~~~

### 15.2 通过指针访问成员

~~~c
void invalidate(SensorSample *sample)
{
    if (sample != NULL) {
        sample->valid = false;
    }
}
~~~

sample->valid 等价于 (*sample).valid。

### 15.3 对齐与填充

~~~c
typedef struct {
    uint8_t a;
    uint32_t b;
    uint8_t c;
} Example;
~~~

sizeof(Example) 可能大于 6，因为编译器为了对齐插入填充。

因此不能直接把普通 struct 原样发到串口或存入跨平台文件。应显式序列化：

~~~c
buffer[0] = (uint8_t)(value >> 8);
buffer[1] = (uint8_t)(value & 0xFFU);
~~~

### 15.4 enum

~~~c
typedef enum {
    STATE_IDLE,
    STATE_RUNNING,
    STATE_ERROR
} SystemState;
~~~

enum 比随处使用神秘数字更清晰。

### 15.5 union

union 的成员共享同一块存储：

~~~c
typedef union {
    uint32_t u32;
    uint8_t bytes[4];
} WordView;
~~~

注意：

- 字节序；
- 活跃成员和语言规则；
- 对齐；
- 不同编译器行为。

协议转换优先使用移位或 memcpy，不要依赖不清楚的类型双关。

### 15.6 typedef

typedef 为类型创建别名：

~~~c
typedef uint32_t Milliseconds;
~~~

别名提高可读性，但不会创建新的强类型。Milliseconds 与 uint32_t 对编译器通常仍是同一类型。

---

## 16. 位运算与寄存器思维

### 16.1 六类运算

~~~c
a & b     // 按位与
a | b     // 按位或
a ^ b     // 按位异或
~a        // 按位取反
a << n    // 左移
a >> n    // 右移
~~~

### 16.2 位掩码

~~~c
#define BIT(n) (1UL << (n))

uint32_t flags = 0U;

flags |= BIT(3);          // 设置 bit3
flags &= ~BIT(3);         // 清除 bit3
flags ^= BIT(3);          // 翻转 bit3
bool set = (flags & BIT(3)) != 0U;
~~~

### 16.3 设置字段

假设某字段位于 bit[6:4]：

~~~c
#define MODE_POS   4U
#define MODE_MASK  (0x7UL << MODE_POS)

register_value =
    (register_value & ~MODE_MASK) |
    (((uint32_t)mode << MODE_POS) & MODE_MASK);
~~~

先清字段，再写入经过掩码限制的新值。

### 16.4 移位风险

- 移位数不能大于等于类型宽度；
- 位运算优先使用无符号；
- 负数移位规则复杂；
- 1 << 31 可能涉及有符号问题，使用 1UL 或 UINT32_C(1)；
- 确认目标类型宽度。

### 16.5 大端与小端

值 0x12345678：

小端内存：

~~~text
低地址 → 78 56 34 12
~~~

大端内存：

~~~text
低地址 → 12 34 56 78
~~~

通信协议由协议定义字节序，不能假设与 MCU 内存顺序相同。

---

## 17. 预处理、头文件与多文件工程

### 17.1 宏

~~~c
#define BUFFER_CAPACITY 128U
~~~

带参数宏：

~~~c
#define CLAMP(x, lo, hi) \
    ((x) < (lo) ? (lo) : ((x) > (hi) ? (hi) : (x)))
~~~

即使加括号，CLAMP(i++, 0, 10) 仍可能多次求值。优先使用 static inline 函数：

~~~c
static inline int clamp_int(int x, int lo, int hi)
{
    if (x < lo) return lo;
    if (x > hi) return hi;
    return x;
}
~~~

### 17.2 条件编译

~~~c
#ifdef DEBUG
    printf("debug value=%d\n", value);
#endif
~~~

用途：

- 调试日志；
- 平台差异；
- 可选功能；
- 头文件保护。

不要用大量条件编译制造多套无法测试的程序。

### 17.3 头文件保护

~~~c
#ifndef SENSOR_H
#define SENSOR_H

#include <stdbool.h>
#include <stdint.h>

bool sensor_init(void);
bool sensor_read(int16_t *value);

#endif
~~~

### 17.4 模块划分

sensor.h：公开接口。  
sensor.c：私有实现。

~~~c
// sensor.c
#include "sensor.h"

static bool initialized;

bool sensor_init(void)
{
    initialized = true;
    return true;
}
~~~

不要在头文件直接定义普通全局变量：

~~~c
// 错误风险：每个包含者都可能得到定义
int global_counter;
~~~

应：

~~~c
// header
extern int global_counter;

// one source file
int global_counter = 0;
~~~

更好的设计通常是隐藏全局状态并提供函数接口。

---

## 18. 函数指针与回调

### 18.1 基础

~~~c
int add(int a, int b)
{
    return a + b;
}

int (*operation)(int, int) = add;
int result = operation(2, 3);
~~~

使用 typedef：

~~~c
typedef int (*BinaryOperation)(int, int);
~~~

### 18.2 回调

~~~c
typedef void (*EventCallback)(uint8_t event, void *context);

typedef struct {
    EventCallback callback;
    void *context;
} EventHandler;

void dispatch(const EventHandler *handler, uint8_t event)
{
    if (handler != NULL && handler->callback != NULL) {
        handler->callback(event, handler->context);
    }
}
~~~

用途：

- 驱动完成通知；
- 按键事件；
- 协议解析；
- 状态机；
- 通用硬件抽象。

context 让同一个回调函数操作不同实例。

### 18.3 回调风险

- 函数指针为空；
- context 生命周期已结束；
- 在中断里调用耗时回调；
- 回调递归触发；
- 多线程并发；
- 接口所有权不清。

---

## 19. 文件 I/O

PC 端 C 程序常使用 stdio.h：

~~~c
FILE *file = fopen("data.txt", "w");
if (file == NULL) {
    perror("fopen");
    return 1;
}

if (fprintf(file, "%d\n", 42) < 0) {
    /* 写入失败 */
}

if (fclose(file) != 0) {
    /* 关闭或刷新失败 */
}
~~~

读取：

~~~c
char line[128];
FILE *file = fopen("data.txt", "r");

if (file != NULL) {
    while (fgets(line, sizeof line, file) != NULL) {
        printf("%s", line);
    }
    fclose(file);
}
~~~

嵌入式裸机没有天然文件系统。要使用文件，通常需要：

- Flash/SD 卡；
- 文件系统库；
- 块设备驱动；
- 掉电和磨损处理。

---

## 20. 错误处理

### 20.1 返回 bool

~~~c
bool sensor_read(int16_t *value);
~~~

简单，但不能区分多种错误。

### 20.2 错误码

~~~c
typedef enum {
    STATUS_OK = 0,
    STATUS_INVALID_ARGUMENT,
    STATUS_TIMEOUT,
    STATUS_IO_ERROR
} Status;

Status sensor_read(int16_t *value);
~~~

### 20.3 输出参数

成功状态与业务值分离：

~~~c
Status divide_safe(int a, int b, int *result);
~~~

### 20.4 assert

assert 适合检查程序员不变量：

~~~c
#include <assert.h>

assert(buffer != NULL);
~~~

它不应替代对外部输入、通信错误和可恢复故障的正常处理；发布构建还可能关闭断言。

---

## 21. 未定义行为与常见错误

### 21.1 未定义行为 UB

C 标准不规定结果，可能：

- 看似正常；
- 崩溃；
- 优化后行为改变；
- 破坏其他数据。

常见 UB：

- 数组越界；
- 解引用无效指针；
- 使用已释放内存；
- 读取未初始化自动变量；
- 有符号整数溢出；
- 除以 0；
- 移位数越界；
- 格式化参数类型错误；
- 修改字符串字面量；
- 数据竞争。

### 21.2 缓冲区溢出

~~~c
void copy_bad(char *destination, const char *source)
{
    strcpy(destination, source);
}
~~~

函数不知道 destination 容量。接口应携带容量，并检查是否能完整写入。

### 21.3 sizeof 指针错误

~~~c
void clear(uint8_t data[64])
{
    memset(data, 0, sizeof data); // sizeof data 是指针大小
}
~~~

改为：

~~~c
void clear(uint8_t *data, size_t length)
{
    memset(data, 0, length);
}
~~~

### 21.4 整数溢出

~~~c
uint32_t total = count * element_size;
~~~

乘法可能在赋值前已溢出。先检查：

~~~c
if (count > UINT32_MAX / element_size) {
    /* 溢出 */
}
~~~

### 21.5 工具

PC 练习时可使用：

- 编译器警告；
- 调试器；
- AddressSanitizer；
- UndefinedBehaviorSanitizer；
- 静态分析；
- 单元测试。

GCC/Clang 示例：

~~~powershell
gcc program.c -std=c17 -Wall -Wextra -Wpedantic -fsanitize=address,undefined -g -o program.exe
~~~

不同 Windows 工具链对 sanitizer 的支持不同。

---

## 22. 编译、链接与调试

### 22.1 四阶段

预处理：

- 展开 include；
- 展开宏；
- 处理条件编译。

编译：

- 检查语法和类型；
- 生成汇编。

汇编：

- 生成目标文件。

链接：

- 合并目标文件和库；
- 解析符号；
- 安排地址；
- 生成可执行文件。

### 22.2 常见错误

**编译错误**

~~~text
expected ';'
~~~

通常是语法问题。

**链接错误**

~~~text
undefined reference to sensor_init
~~~

可能声明了但没有把定义对应的源文件加入工程。

**重复定义**

~~~text
multiple definition of global_counter
~~~

可能在头文件中错误定义全局对象。

### 22.3 Debug 与 Release

Debug：

- 低优化；
- 有调试信息；
- 行为较容易跟踪。

Release：

- 优化；
- 变量可能被消除；
- 执行顺序可能重排；
- 暴露依赖 UB 或竞态的问题。

“Debug 正常，Release 异常”常提示：

- 未初始化；
- 越界；
- volatile 缺失；
- 竞态；
- 时序依赖；
- 未定义行为。

### 22.4 断点与观察

基本调试流程：

1. 复现；
2. 缩小范围；
3. 在状态变化前设置断点；
4. 查看调用栈、局部变量、寄存器；
5. 单步验证假设；
6. 修复后做回归测试。

不要没有假设地到处打断点。

---

## 23. Embedded C：volatile、寄存器与中断

### 23.1 volatile

~~~c
volatile uint32_t * const gpio_register =
    (volatile uint32_t *)0x40020000UL;
~~~

volatile 常用于：

- 内存映射寄存器；
- 被中断异步改变的简单对象；
- 某些硬件共享状态。

volatile 不保证：

- 原子性；
- 多线程安全；
- 互斥；
- 操作顺序满足所有并发需求。

~~~c
volatile uint32_t count;
count++;  // 仍可能是读、加、写三步
~~~

### 23.2 寄存器读改写

~~~c
REGISTER |= MASK;
REGISTER &= ~MASK;
~~~

风险：

- 寄存器某些位是写 1 清零；
- 某些位只读；
- 中断或硬件同时修改；
- 读可能有副作用。

必须看参考手册，不能所有寄存器都套同一种写法。

### 23.3 中断服务函数

原则：

- 快速；
- 清标志；
- 保存最少数据；
- 不阻塞；
- 不做长日志；
- 把复杂处理交给主循环或任务。

~~~c
static volatile bool sample_ready;
static volatile uint16_t latest_sample;

void ADC_IRQHandler(void)
{
    latest_sample = adc_read_data_register();
    adc_clear_interrupt_flag();
    sample_ready = true;
}
~~~

主循环：

~~~c
for (;;) {
    if (sample_ready) {
        uint16_t sample;
        uint32_t key = enter_critical();
        sample = latest_sample;
        sample_ready = false;
        exit_critical(key);

        process_sample(sample);
    }
}
~~~

### 23.4 临界区

临界区保护不能被并发打断的短操作：

- 尽量短；
- 保存并恢复进入前中断状态；
- 不在临界区执行阻塞 I/O；
- 多核和 RTOS 环境需要相应同步机制。

### 23.5 状态机

~~~c
typedef enum {
    LED_OFF,
    LED_ON,
    LED_ERROR
} LedState;

void led_state_update(LedState *state, bool button, bool fault)
{
    if (state == NULL) {
        return;
    }

    if (fault) {
        *state = LED_ERROR;
    } else if (button) {
        *state = (*state == LED_ON) ? LED_OFF : LED_ON;
    }
}
~~~

真实按键要消抖并区分电平与“按下事件”，否则按住时会反复切换。

### 23.6 环形缓冲区

~~~c
#include <stdbool.h>
#include <stddef.h>
#include <stdint.h>

#define RING_CAPACITY 128U

typedef struct {
    uint8_t data[RING_CAPACITY];
    size_t head;
    size_t tail;
} RingBuffer;

bool ring_is_empty(const RingBuffer *ring)
{
    if (ring == NULL) {
        return true;
    }
    return ring->head == ring->tail;
}

bool ring_push(RingBuffer *ring, uint8_t value)
{
    if (ring == NULL) {
        return false;
    }

    size_t next = (ring->head + 1U) % RING_CAPACITY;
    if (next == ring->tail) {
        return false;
    }

    ring->data[ring->head] = value;
    ring->head = next;
    return true;
}

bool ring_pop(RingBuffer *ring, uint8_t *value)
{
    if (ring == NULL || value == NULL || ring_is_empty(ring)) {
        return false;
    }

    *value = ring->data[ring->tail];
    ring->tail = (ring->tail + 1U) % RING_CAPACITY;
    return true;
}
~~~

这里浪费一个槽位：

- head == tail：空；
- next == tail：满。

并发使用时还要分析：

- 单生产者/单消费者；
- 索引访问是否原子；
- 内存顺序；
- 中断和任务优先级；
- 缓冲区满的策略。

---

## 24. C 语言阶段测试

### 24.1 必须独立写出

- 数组最大值、平均值；
- 安全读取整数；
- 字符串长度和复制；
- swap 指针函数；
- struct 传感器记录；
- 位掩码设置字段；
- 函数指针回调；
- 环形缓冲区；
- 非阻塞超时；
- UART 字节流状态机。

### 24.2 口述题

1. 数组与指针为什么相关但不相同？
2. const int* 与 int* const 区别？
3. .data 与 .bss 放什么？
4. malloc 失败和碎片怎么处理？
5. struct 为什么可能有填充？
6. memcpy 与 memmove 区别？
7. static 的两种常见作用？
8. volatile 能做什么、不能做什么？
9. 为什么中断里不应 printf 大量日志？
10. Debug 正常、Release 异常先查什么？

### 24.3 通过标准

- 能解释每个答案；
- 能在 45 分钟内写出环形缓冲区；
- 编译器常用警告为零；
- 对所有数组访问给出容量；
- 所有指针接口说明 NULL 与生命周期；
- 对错误路径有处理。

---

# 第三部分：C++ 从入门到嵌入式

## 25. C++ Hello World

hello.cpp：

~~~cpp
#include <iostream>

int main()
{
    std::cout << "Hello, C++!\n";
    return 0;
}
~~~

编译：

~~~powershell
g++ hello.cpp -std=c++20 -Wall -Wextra -Wpedantic -O0 -g -o hello_cpp.exe
.\hello_cpp.exe
~~~

### 25.1 std 是什么

C++ 标准库名称大多位于 std 命名空间：

~~~cpp
std::cout
std::string
std::vector
~~~

初学不建议在全局写：

~~~cpp
using namespace std;
~~~

它会把大量名称引入当前作用域，可能冲突。可写：

~~~cpp
using std::cout;
using std::string;
~~~

或保留 std:: 前缀。

### 25.2 C 与 C++ 不是同一种语言

C++ 继承了大量 C 语法，但增加：

- 引用；
- 函数重载；
- 类和对象；
- 构造与析构；
- 模板；
- 异常；
- 标准容器与算法；
- 更严格的类型规则。

不要说“C++ 完全兼容 C”。同一段代码在两种语言中可能语义不同或不能编译。

---

## 26. C++ 类型、初始化、引用与 nullptr

### 26.1 列表初始化

~~~cpp
int count{0};
double voltage{3.3};
std::string name{"Xu"};
~~~

列表初始化能阻止部分窄化转换：

~~~cpp
int value{3.14};  // 编译错误：窄化
~~~

### 26.2 auto

~~~cpp
auto count = 10;          // int
auto voltage = 3.3;       // double
auto name = std::string{"Xu"};
~~~

auto 让编译器推导类型，不代表“没有类型”。当类型信息对读者重要或推导结果容易误解时，显式写出。

### 26.3 引用

~~~cpp
void increment(int& value)
{
    ++value;
}
~~~

引用：

- 必须绑定对象；
- 之后不能改为引用另一个对象；
- 调用语法像普通变量；
- 常用于参数和返回值。

只读引用：

~~~cpp
void print_name(const std::string& name)
{
    std::cout << name << '\n';
}
~~~

避免复制，同时承诺不修改。

### 26.4 值、引用、指针怎么选

| 需求 | 常见选择 |
|---|---|
| 小对象输入，允许复制 | 值 |
| 必须存在且要修改 | T& |
| 必须存在且只读，复制昂贵 | const T& |
| 可以没有对象 | T* 或合适的 optional |
| 表达所有权转移 | unique_ptr 或值 |

### 26.5 nullptr

~~~cpp
int *pointer = nullptr;
~~~

C++ 使用 nullptr 表达空指针，比整数 0 或宏 NULL 的重载行为更明确。

### 26.6 类型转换

C++ 推荐明确转换：

- static_cast：常规显式转换；
- dynamic_cast：多态层次安全运行时转换；
- const_cast：改变 const/volatile 限定；
- reinterpret_cast：底层重新解释，风险高。

~~~cpp
double average = static_cast<double>(sum) / count;
~~~

不要为了消除编译错误随意 cast。先判断类型设计是否正确。

---

## 27. 函数重载、默认参数与 constexpr

### 27.1 重载

~~~cpp
int add(int a, int b)
{
    return a + b;
}

double add(double a, double b)
{
    return a + b;
}
~~~

编译器根据参数类型选择。仅返回类型不同不能构成重载。

### 27.2 默认参数

~~~cpp
void set_brightness(int level, bool animate = true);
~~~

调用：

~~~cpp
set_brightness(3);
set_brightness(3, false);
~~~

默认参数通常放在公开声明处，避免声明和定义不一致。

### 27.3 inline

inline 不保证一定把函数展开。现代编译器根据优化策略决定。它还影响多翻译单元中的定义规则，常用于头文件中的短函数。

### 27.4 constexpr

~~~cpp
constexpr int square(int x)
{
    return x * x;
}

constexpr int buffer_size = square(8);
~~~

constexpr 表示函数或对象可参与常量求值；满足条件时在编译期计算，也可在运行期调用。

嵌入式价值：

- 编译期查表；
- 单位和范围计算；
- 减少运行时工作；
- 类型安全替代部分宏。

---

## 28. 类与对象

### 28.1 第一个类

~~~cpp
class Led {
public:
    explicit Led(unsigned pin)
        : pin_{pin}, is_on_{false}
    {
        gpio_config_output(pin_);
    }

    void on()
    {
        gpio_write(pin_, true);
        is_on_ = true;
    }

    void off()
    {
        gpio_write(pin_, false);
        is_on_ = false;
    }

    bool is_on() const
    {
        return is_on_;
    }

private:
    unsigned pin_;
    bool is_on_;
};
~~~

### 28.2 访问控制

- public：类外可以访问；
- private：只有类自身及友元等可以访问；
- protected：类和派生类可访问。

封装不是“所有变量都 private”这么简单，而是由类维护自己的有效状态和不变量。

### 28.3 构造函数

构造函数建立有效对象：

~~~cpp
Led status_led{13};
~~~

初始化列表：

~~~cpp
: pin_{pin}, is_on_{false}
~~~

成员按它们在类中的声明顺序初始化，不按初始化列表书写顺序。

### 28.4 explicit

单参数构造函数可能参与隐式转换：

~~~cpp
explicit Led(unsigned pin);
~~~

explicit 防止不期望的自动转换。

### 28.5 const 成员函数

~~~cpp
bool is_on() const;
~~~

表示该函数不通过 this 修改普通数据成员，可被 const 对象调用。

### 28.6 析构函数

~~~cpp
class Resource {
public:
    ~Resource()
    {
        release();
    }
};
~~~

对象生命周期结束时自动调用析构函数，适合归还资源。

---

## 29. RAII：C++ 最重要的资源思想

RAII：

> 构造函数取得资源，析构函数释放资源，让资源生命周期与对象生命周期绑定。

资源包括：

- 内存；
- 文件；
- 锁；
- 外设所有权；
- 临界区；
- 网络连接。

### 29.1 文件例子

~~~cpp
#include <fstream>

void write_value()
{
    std::ofstream file{"data.txt"};
    if (!file) {
        return;
    }

    file << 42 << '\n';
} // 自动关闭文件
~~~

### 29.2 临界区守卫

~~~cpp
class InterruptGuard {
public:
    InterruptGuard()
        : previous_state_{save_and_disable_interrupts()}
    {
    }

    ~InterruptGuard()
    {
        restore_interrupts(previous_state_);
    }

    InterruptGuard(const InterruptGuard&) = delete;
    InterruptGuard& operator=(const InterruptGuard&) = delete;

private:
    unsigned previous_state_;
};
~~~

使用：

~~~cpp
void update_shared_state()
{
    InterruptGuard guard;
    shared_state = 1;
}
~~~

即使函数中途 return，也会恢复进入前的中断状态。

---

## 30. 拷贝、移动与 Rule of Zero

### 30.1 拷贝

~~~cpp
std::string a{"hello"};
std::string b = a;
~~~

b 获得自己的字符串值。

### 30.2 浅拷贝风险

如果类直接管理裸资源：

~~~cpp
class Buffer {
    char *data_;
};
~~~

编译器默认拷贝只复制指针，两个对象可能指向同一块内存，最终重复释放。

### 30.3 五个特殊成员

- 析构函数；
- 拷贝构造；
- 拷贝赋值；
- 移动构造；
- 移动赋值。

如果手工管理资源，常需考虑 Rule of Five。

### 30.4 Rule of Zero

更好的方式是让标准资源类型管理资源：

~~~cpp
class Buffer {
public:
    explicit Buffer(std::size_t size)
        : data_(size)
    {
    }

private:
    std::vector<std::uint8_t> data_;
};
~~~

无需自己写析构、拷贝和移动，这叫 Rule of Zero。

### 30.5 移动

移动把可转移资源从一个对象转移给另一个对象：

~~~cpp
std::vector<int> source{1, 2, 3};
std::vector<int> destination = std::move(source);
~~~

移动后的 source 仍是有效对象，但具体内容通常不应假设，只能安全销毁或重新赋值等。

std::move 本身不移动数据，它把表达式转换为允许移动的值类别，真正动作由移动构造/赋值完成。

---

## 31. 继承、多态与组合

### 31.1 运行时多态

~~~cpp
class Sensor {
public:
    virtual ~Sensor() = default;
    virtual bool read(float& value) = 0;
};

class TemperatureSensor final : public Sensor {
public:
    bool read(float& value) override
    {
        value = read_temperature_hardware();
        return true;
    }
};
~~~

### 31.2 关键字

- virtual：启用虚函数分派；
- override：要求确实覆盖基类虚函数；
- final：禁止继续覆盖或继承；
- = 0：纯虚函数，使类成为抽象类。

### 31.3 为什么基类析构常设为 virtual

若通过基类指针删除派生对象，虚析构确保派生类析构也执行。

如果类永远不允许通过基类指针删除，应在接口设计中明确，而不是机械添加或删除 virtual。

### 31.4 多态代价

典型代价：

- 每个多态对象可能有虚表指针；
- 间接调用；
- 某些优化受限；
- 代码尺寸。

多数情况下代价不大，但资源受限 MCU 应基于测量和需求决定。

### 31.5 优先组合

“有一个”通常用组合，“是一个”才考虑继承：

~~~cpp
class Robot {
private:
    Motor left_motor_;
    Motor right_motor_;
};
~~~

不要为了展示面向对象建立深继承树。

---

## 32. 模板

### 32.1 函数模板

~~~cpp
template <typename T>
T maximum(T a, T b)
{
    return (a < b) ? b : a;
}
~~~

调用：

~~~cpp
auto a = maximum(2, 3);
auto b = maximum(2.5, 1.0);
~~~

模板在编译期为使用到的类型实例化。

### 32.2 类模板

~~~cpp
template <typename T, std::size_t Capacity>
class FixedBuffer {
public:
    bool push(const T& value)
    {
        if (size_ >= Capacity) {
            return false;
        }
        data_[size_++] = value;
        return true;
    }

    std::size_t size() const
    {
        return size_;
    }

private:
    std::array<T, Capacity> data_{};
    std::size_t size_{0};
};
~~~

嵌入式中模板可实现：

- 固定容量容器；
- 类型安全寄存器封装；
- 编译期配置；
- 零运行时开销抽象。

代价：

- 多种实例可能增加代码尺寸；
- 错误信息复杂；
- 编译时间增加。

---

## 33. 标准容器

### 33.1 std::array

固定大小、连续存储：

~~~cpp
std::array<int, 4> values{1, 2, 3, 4};
~~~

优点：

- 知道长度；
- 支持迭代器和算法；
- 不动态分配。

非常适合固定容量嵌入式数据。

### 33.2 std::vector

动态连续数组：

~~~cpp
std::vector<int> values;
values.reserve(100);
values.push_back(1);
~~~

特点：

- 随机访问 O(1)；
- 尾部追加摊销 O(1)；
- 中间插入/删除 O(n)；
- 扩容会重新分配，迭代器和指针可能失效。

实时路径使用前：

- reserve；
- 明确最大容量；
- 评估动态内存；
- 避免不可预测扩容。

### 33.3 std::string

管理字符串内存：

~~~cpp
std::string message{"hello"};
message += " world";
~~~

比手动 char 数组方便，但可能动态分配。资源受限环境可使用固定字符串方案或 char 数组。

### 33.4 deque、list、map、unordered_map

| 容器 | 主要特点 |
|---|---|
| deque | 两端高效插入删除，内存不一定连续 |
| list | 双向链表，节点分配多，缓存局部性差 |
| map | 有序键值，典型平衡树，O(log n) |
| unordered_map | 哈希表，平均 O(1)，最坏 O(n) |

不要看到理论复杂度就选容器。还要看：

- 元素数量；
- 内存；
- 分配次数；
- 缓存；
- 确定性；
- 是否需要排序。

### 33.5 std::span

C++20 span 是不拥有数据的连续视图：

~~~cpp
void process(std::span<const std::uint8_t> data)
{
    for (auto byte : data) {
        consume(byte);
    }
}
~~~

它同时携带指针和长度，但调用者必须保证底层数据在使用期间仍有效。

---

## 34. 迭代器、算法与 Lambda

### 34.1 范围 for

~~~cpp
for (const auto& value : values) {
    std::cout << value << '\n';
}
~~~

使用 const auto& 避免复制大对象。

### 34.2 标准算法

~~~cpp
#include <algorithm>
#include <vector>

std::vector<int> values{4, 1, 3, 2};
std::sort(values.begin(), values.end());

auto it = std::find(values.begin(), values.end(), 3);
if (it != values.end()) {
    /* 找到 */
}
~~~

优先使用经过验证的标准算法，而不是每次重写。

### 34.3 Lambda

~~~cpp
int threshold = 10;

auto above_threshold = [threshold](int value) {
    return value > threshold;
};
~~~

捕获：

- [threshold]：按值；
- [&threshold]：按引用；
- [=]：默认按值；
- [&]：默认按引用。

按引用捕获的对象必须在 lambda 使用期间仍有效。

算法例子：

~~~cpp
auto count = std::count_if(values.begin(), values.end(),
                           [threshold](int value) {
                               return value > threshold;
                           });
~~~

---

## 35. 智能指针

### 35.1 unique_ptr

独占所有权：

~~~cpp
auto sensor = std::make_unique<TemperatureSensor>();
~~~

不能复制，可以移动。对象随 unique_ptr 自动销毁。

### 35.2 shared_ptr

共享所有权，通过引用计数管理：

~~~cpp
auto object = std::make_shared<Object>();
~~~

成本和风险：

- 控制块；
- 引用计数；
- 可能原子操作；
- 循环引用；
- 所有权容易变模糊。

不是“现代 C++ 就全部用 shared_ptr”。

### 35.3 weak_ptr

观察 shared_ptr 管理的对象而不增加强引用，可打破某些循环引用。

### 35.4 嵌入式是否使用

如果系统不使用动态内存：

- 静态对象；
- 栈对象；
- 引用；
- 不拥有指针；
- 固定池。

无需为了“现代”而引入智能指针。智能指针解决的是所有权，不是所有指针场景。

---

## 36. 异常与错误处理

~~~cpp
try {
    risky_operation();
} catch (const std::exception& error) {
    std::cerr << error.what() << '\n';
}
~~~

异常优点：

- 错误沿调用栈传播；
- RAII 自动清理；
- 正常路径与错误路径分离。

嵌入式考虑：

- 工具链支持；
- 代码尺寸；
- 最坏执行时间；
- 团队规范；
- 异常是否被禁用。

禁用异常时可使用：

- 状态码；
- expected 风格结果类型；
- optional 表示“有或无值”；
- assert 检查内部不变量。

不能在一个工程中一半假设异常开启、一半关闭而没有明确规则。

---

## 37. C 与 C++ 互操作

C 头文件供 C++ 调用时：

~~~c
#ifdef __cplusplus
extern "C" {
#endif

void driver_init(void);

#ifdef __cplusplus
}
#endif
~~~

extern "C" 控制链接名称，不会把 C 函数自动变成 C++ 类，也不改变调用接口的类型安全要求。

C++ 可在底层调用 C 驱动，上层用类封装：

~~~cpp
class Uart {
public:
    void init()
    {
        uart_driver_init();
    }
};
~~~

---

## 38. 嵌入式 C++ 的原则

### 38.1 可以放心考虑的特性

- class 封装；
- constexpr；
- enum class；
- 模板；
- std::array；
- RAII；
- 引用；
- 函数重载；
- 零开销抽象。

### 38.2 需要评估的特性

- 动态内存；
- 异常；
- RTTI；
- 深层虚函数体系；
- iostream；
- 大型 STL 容器；
- 静态对象初始化顺序；
- 模板实例化导致代码膨胀。

不是全面禁止，而是测量：

- Flash；
- RAM；
- 栈；
- 执行时间；
- 启动时间；
- 最坏情况；
- 工具链支持。

### 38.3 enum class

~~~cpp
enum class Mode : std::uint8_t {
    automatic,
    manual,
    error
};
~~~

比普通 enum 更强类型，不会轻易与整数或其他枚举混用。

### 38.4 固定容量优先

~~~cpp
template <typename T, std::size_t N>
using FixedArray = std::array<T, N>;
~~~

资源上限明确的系统更容易分析。

### 38.5 不在构造阶段做不可控硬件工作

全局对象构造发生顺序和硬件初始化阶段要谨慎。常见做法：

- 构造只建立软件状态；
- 显式 init 完成可能失败的硬件初始化；
- 返回错误状态；
- 控制全局初始化顺序。

---

## 39. C++ 阶段测试

### 39.1 必须独立写出

- C++ Hello World；
- const 引用参数；
- 一个带构造函数的 Led 类；
- RAII 临界区守卫；
- std::array 与 std::vector 使用；
- std::sort + lambda；
- 抽象 Sensor 接口和两个实现；
- 固定容量模板缓冲区；
- unique_ptr 所有权转移；
- C 驱动的 C++ 封装。

### 39.2 口述题

1. 引用与指针怎么选？
2. explicit 解决什么问题？
3. 初始化列表为什么重要？
4. RAII 是什么？
5. Rule of Zero 比 Rule of Five 好在哪里？
6. std::move 自己会移动吗？
7. vector 扩容使哪些东西失效？
8. 虚析构为什么重要？
9. map 与 unordered_map 差异？
10. 为什么嵌入式不必禁止全部 C++ 特性？

### 39.3 通过标准

- 不把 C++ 只理解为 C 加 class；
- 能讲清对象生命周期和所有权；
- 知道容器复杂度和内存行为；
- 不滥用 new、shared_ptr 和继承；
- 能根据 MCU 资源选择语言特性。

---

# 第四部分：数据结构与算法

## 40. 数据结构解决什么问题

数据结构决定数据如何组织，算法决定如何处理。

同一需求可能有不同选择：

- 按下标快速读取：数组；
- 先进先出：队列；
- 后进先出：栈；
- 按键快速查值：哈希表；
- 按优先级取任务：堆；
- 表示网络或路径：图。

选择不能只看理论复杂度，还要看：

- 数据规模；
- 最大容量；
- 内存是否连续；
- 是否允许动态分配；
- 最坏时间；
- 缓存局部性；
- 实现和验证成本。

---

## 41. 时间复杂度与空间复杂度

### 41.1 Big O

Big O 描述输入规模 n 增大时，工作量增长的数量级。

常见顺序：

~~~text
O(1)
O(log n)
O(n)
O(n log n)
O(n²)
O(2ⁿ)
O(n!)
~~~

### 41.2 例子

O(1)：

~~~c
int first = values[0];
~~~

O(n)：

~~~c
for (size_t i = 0; i < n; ++i) {
    sum += values[i];
}
~~~

O(n²)：

~~~c
for (size_t i = 0; i < n; ++i) {
    for (size_t j = 0; j < n; ++j) {
        compare(values[i], values[j]);
    }
}
~~~

### 41.3 最好、平均、最坏、摊销

- 最好情况：最幸运输入；
- 平均情况：假设某种输入分布；
- 最坏情况：保证上限；
- 摊销：一串操作平均成本。

vector 尾部 push_back 通常是摊销 O(1)：多数追加 O(1)，偶尔扩容复制 O(n)。

实时系统更重视最坏情况，而不仅是平均速度。

### 41.4 空间复杂度

算法额外使用多少内存：

- 原地算法可能 O(1) 额外空间；
- 归并排序通常 O(n) 额外空间；
- 递归深度会消耗栈。

在 MCU 上，栈只有几 KB 时，空间可能比时间更关键。

---

## 42. 数组与动态数组

### 42.1 数组

连续存储：

~~~text
[0][1][2][3][4]
~~~

特点：

- 按下标 O(1)；
- 缓存局部性好；
- 中间插入删除 O(n)；
- 固定数组容量编译期确定。

### 42.2 动态数组

C++ std::vector：

- size：当前元素数；
- capacity：已分配容量；
- 扩容时可能搬家；
- reserve 预留容量；
- resize 改变元素数量。

~~~cpp
std::vector<int> values;
values.reserve(100);
~~~

### 42.3 什么时候选数组

- 数据量有明确上限；
- 高频顺序遍历；
- 需要下标；
- 不想动态分配；
- DMA 需要连续缓冲区。

嵌入式通常优先固定数组和环形数组。

---

## 43. 单链表

每个节点保存数据和下一个节点地址：

~~~text
[data|next] → [data|next] → [data|null]
~~~

### 43.1 定义

~~~c
typedef struct ListNode {
    int value;
    struct ListNode *next;
} ListNode;
~~~

### 43.2 头插

~~~c
bool list_push_front(ListNode **head, int value)
{
    if (head == NULL) {
        return false;
    }

    ListNode *node = malloc(sizeof *node);
    if (node == NULL) {
        return false;
    }

    node->value = value;
    node->next = *head;
    *head = node;
    return true;
}
~~~

### 43.3 查找

~~~c
ListNode *list_find(ListNode *head, int target)
{
    for (ListNode *node = head; node != NULL; node = node->next) {
        if (node->value == target) {
            return node;
        }
    }
    return NULL;
}
~~~

### 43.4 释放

~~~c
void list_clear(ListNode **head)
{
    if (head == NULL) {
        return;
    }

    ListNode *node = *head;
    while (node != NULL) {
        ListNode *next = node->next;
        free(node);
        node = next;
    }
    *head = NULL;
}
~~~

### 43.5 复杂度

| 操作 | 单链表 |
|---|---:|
| 头插 | O(1) |
| 已知节点后插入 | O(1) |
| 按下标访问 | O(n) |
| 查找 | O(n) |
| 删除已知前驱后的节点 | O(1) |

### 43.6 MCU 上的缺点

- 每节点有指针开销；
- 动态分配碎片；
- 节点分散，缓存局部性差；
- 容易内存泄漏和断链；
- 最坏分配时间不稳定。

替代：

- 固定节点池；
- 数组索引代替指针；
- intrusive list；
- 固定容量容器。

### 43.7 双向链表

节点含 prev 和 next：

- 已知节点可 O(1) 删除；
- 能双向遍历；
- 指针和维护成本更高。

---

## 44. 栈

栈是 LIFO：后进先出。

~~~text
push 3
push 5
pop → 5
~~~

应用：

- 函数调用栈；
- 表达式求值；
- 括号匹配；
- DFS；
- 撤销操作。

### 44.1 数组实现

~~~c
#define STACK_CAPACITY 32U

typedef struct {
    int data[STACK_CAPACITY];
    size_t size;
} IntStack;

bool stack_push(IntStack *stack, int value)
{
    if (stack == NULL || stack->size >= STACK_CAPACITY) {
        return false;
    }
    stack->data[stack->size++] = value;
    return true;
}

bool stack_pop(IntStack *stack, int *value)
{
    if (stack == NULL || value == NULL || stack->size == 0U) {
        return false;
    }
    *value = stack->data[--stack->size];
    return true;
}
~~~

push/pop 均为 O(1)。

### 44.2 函数调用栈

通常保存：

- 返回地址；
- 保存的寄存器；
- 局部变量；
- 参数的一部分；
- 对齐空间。

栈溢出原因：

- 大局部数组；
- 深递归；
- 中断嵌套；
- 任务栈太小；
- 调用层次过深。

---

## 45. 队列与环形队列

队列是 FIFO：先进先出。

应用：

- UART 接收；
- 任务消息；
- 传感器样本；
- 事件处理。

### 45.1 环形数组

~~~text
尾部到数组末尾后回到开头
~~~

优点：

- 固定容量；
- 无需搬移；
- 入队出队 O(1)；
- 适合中断与主循环解耦。

### 45.2 两种判满设计

**浪费一个槽位**

- head == tail：空；
- next(head) == tail：满；
- 有效容量 N-1。

**额外保存 size**

- size == 0：空；
- size == N：满；
- 所有 N 个槽可用；
- 并发更新多一个共享变量。

### 45.3 满时策略

- 拒绝新数据；
- 覆盖最旧数据；
- 阻塞等待；
- 启用流控；
- 报警并统计。

策略由业务决定。例如控制命令不应静默覆盖，连续波形可能允许保留最新数据。

---

## 46. 哈希表

哈希表把 key 通过哈希函数映射到桶：

~~~text
key → hash(key) → bucket index
~~~

### 46.1 冲突

不同 key 可能进入同一桶：

- 拉链法：桶内链表或其他容器；
- 开放寻址：在数组中继续探测。

### 46.2 复杂度

- 平均查找/插入：O(1)；
- 最坏：O(n)；
- 取决于哈希质量、负载因子和冲突策略。

### 46.3 开放寻址概念

线性探测：

~~~text
index, index+1, index+2 ... 直到空位
~~~

删除不能简单清空，否则会破坏探测链，通常需要“墓碑”状态。

### 46.4 嵌入式场景

小规模固定命令表常用排序数组或 switch，可能比通用哈希表更简单、可预测。

选择哈希表前问：

- key 数量多少？
- 是否动态变化？
- 能否接受最坏 O(n)？
- 内存预算？
- 哈希攻击是否相关？

---

## 47. 树与二叉搜索树

### 47.1 基本术语

- 根；
- 父节点；
- 子节点；
- 叶节点；
- 深度；
- 高度；
- 子树。

二叉树每个节点最多两个子节点。

### 47.2 遍历

前序：

~~~text
根 → 左 → 右
~~~

中序：

~~~text
左 → 根 → 右
~~~

后序：

~~~text
左 → 右 → 根
~~~

层序：按层使用队列。

### 47.3 二叉搜索树 BST

常见不变量：

- 左子树键小于节点；
- 右子树键大于节点；
- 重复键策略需明确。

查找：

~~~c
typedef struct TreeNode {
    int key;
    struct TreeNode *left;
    struct TreeNode *right;
} TreeNode;

TreeNode *bst_find(TreeNode *root, int key)
{
    while (root != NULL) {
        if (key == root->key) {
            return root;
        }
        root = (key < root->key) ? root->left : root->right;
    }
    return NULL;
}
~~~

复杂度：

- 平均与树高相关，理想平衡为 O(log n)；
- 极端退化成链表，O(n)。

### 47.4 平衡树

AVL、红黑树等通过旋转控制高度。标准 std::map 常用平衡树实现，但标准不强制具体结构，只规定复杂度等要求。

---

## 48. 堆与优先队列

这里的“堆”是数据结构，不是 malloc 使用的 heap 内存区域。

最大堆：

- 每个父节点大于等于子节点；
- 根是最大值；
- 通常用数组存储完全二叉树。

索引从 0：

~~~text
left(i)  = 2i + 1
right(i) = 2i + 2
parent(i)= (i - 1) / 2，要求 i > 0
~~~

复杂度：

| 操作 | 复杂度 |
|---|---:|
| 查看最大/最小 | O(1) |
| 插入 | O(log n) |
| 删除根 | O(log n) |
| 建堆 | O(n) |

应用：

- 优先任务；
- 定时事件；
- Top K；
- Dijkstra 等算法。

C++：

~~~cpp
std::priority_queue<int> queue;
queue.push(3);
queue.push(10);
int largest = queue.top();
~~~

实时系统中要明确相同优先级顺序、容量和动态分配行为。

---

## 49. 图

图由顶点和边组成：

- 有向/无向；
- 加权/无权；
- 稀疏/稠密。

### 49.1 表示

邻接矩阵：

- 空间 O(V²)；
- 判断两点是否有边 O(1)；
- 适合小而稠密。

邻接表：

- 空间 O(V+E)；
- 遍历某点邻居高效；
- 适合稀疏图。

### 49.2 BFS

广度优先搜索使用队列：

- 按距离层级扩展；
- 无权图最短路径；
- 复杂度 O(V+E)。

### 49.3 DFS

深度优先使用递归或显式栈：

- 连通性；
- 拓扑相关；
- 路径搜索；
- 复杂度 O(V+E)。

MCU 上深图递归 DFS 可能栈溢出，可使用固定容量显式栈。

### 49.4 Dijkstra

适用于边权非负的单源最短路径，常配最小堆。

不能用于有负权边的图。算法正确性依赖前提，不能只背名字。

---

## 50. 查找

### 50.1 线性查找

~~~c
int find_linear(const int *values, size_t count, int target)
{
    for (size_t i = 0; i < count; ++i) {
        if (values[i] == target) {
            return (int)i;
        }
    }
    return -1;
}
~~~

复杂度 O(n)，无需排序。

### 50.2 二分查找

前提：数组已按同一比较规则排序。

~~~c
bool binary_search_int(const int *values, size_t count, int target)
{
    if (values == NULL) {
        return false;
    }

    size_t left = 0U;
    size_t right = count;  // 半开区间 [left, right)

    while (left < right) {
        size_t middle = left + (right - left) / 2U;

        if (values[middle] < target) {
            left = middle + 1U;
        } else {
            right = middle;
        }
    }

    return left < count && values[left] == target;
}
~~~

复杂度 O(log n)。

常见错误：

- 未排序；
- 左右边界不一致；
- middle 计算溢出；
- 查重复元素时不知道要第一个还是任意一个。

---

## 51. 排序

### 51.1 冒泡排序

相邻比较交换：

- 时间 O(n²)；
- 可稳定；
- 易理解；
- 实际大数据通常不选。

~~~c
void bubble_sort(int *values, size_t count)
{
    if (values == NULL) {
        return;
    }

    for (size_t end = count; end > 1U; --end) {
        bool swapped = false;

        for (size_t i = 1U; i < end; ++i) {
            if (values[i - 1U] > values[i]) {
                int temporary = values[i - 1U];
                values[i - 1U] = values[i];
                values[i] = temporary;
                swapped = true;
            }
        }

        if (!swapped) {
            break;
        }
    }
}
~~~

### 51.2 插入排序

- O(n²)；
- 稳定；
- 原地；
- 小数组或基本有序时表现好；
- 实现简单、常数低。

### 51.3 选择排序

- O(n²)；
- 交换次数少；
- 通常不稳定；
- 无论原数组是否有序都要大量比较。

### 51.4 快速排序

- 平均 O(n log n)；
- 最坏 O(n²)；
- 通常原地；
- 通常不稳定；
- 递归和枢轴选择影响明显。

### 51.5 归并排序

- O(n log n)；
- 稳定；
- 通常需要 O(n) 辅助空间；
- 适合链表和外部排序。

### 51.6 堆排序

- O(n log n) 最坏保证；
- 原地；
- 不稳定；
- 缓存局部性通常不如快速排序。

### 51.7 计数排序

- O(n+k)；
- 适合键范围 k 不大的整数；
- 需要额外计数空间；
- 不是比较排序。

### 51.8 排序对比

| 算法 | 平均 | 最坏 | 额外空间 | 稳定 |
|---|---:|---:|---:|:---:|
| 冒泡 | O(n²) | O(n²) | O(1) | 可 |
| 插入 | O(n²) | O(n²) | O(1) | 是 |
| 选择 | O(n²) | O(n²) | O(1) | 否 |
| 快排 | O(n log n) | O(n²) | 栈空间 | 否 |
| 归并 | O(n log n) | O(n log n) | O(n) | 是 |
| 堆排 | O(n log n) | O(n log n) | O(1) | 否 |

“稳定”表示相等键元素排序后相对顺序保持。

实际 C++ 优先 std::sort、std::stable_sort 等标准算法。

---

## 52. 递归、分治、动态规划与贪心

### 52.1 递归

必须有：

- 基准条件；
- 问题规模缩小；
- 栈深度可控。

### 52.2 分治

把问题分成子问题、分别解决、合并结果。归并排序是典型分治。

### 52.3 动态规划

适合：

- 重叠子问题；
- 最优子结构。

核心步骤：

1. 定义状态；
2. 写转移；
3. 定义初始条件；
4. 确定计算顺序；
5. 恢复答案（如需要）。

斐波那契：

~~~c
uint64_t fibonacci(unsigned n)
{
    if (n < 2U) {
        return n;
    }

    uint64_t previous = 0U;
    uint64_t current = 1U;

    for (unsigned i = 2U; i <= n; ++i) {
        uint64_t next = previous + current;
        previous = current;
        current = next;
    }
    return current;
}
~~~

还要检查 uint64_t 溢出，不能认为算法能无限计算。

### 52.4 贪心

每一步选择当前看起来最优的方案。只有满足特定性质时才能得到全局最优，需要证明或已知结论。

---

## 53. 嵌入式中的结构选择

| 需求 | 首选思路 |
|---|---|
| UART 连续接收 | 固定环形缓冲区 |
| 有限状态控制 | enum + 状态机 |
| 固定传感器列表 | std::array / C 数组 |
| 动态但上限明确 | 固定池 + 索引 |
| 定时事件按最近到期 | 最小堆或有序表 |
| 小命令表 | switch / 排序数组 |
| RTOS 任务通信 | RTOS 队列 |
| 图像大缓冲 | 静态或专用内存区 |

嵌入式原则：

- 容量上限明确；
- 失败行为明确；
- 最坏时间可分析；
- 不隐藏动态分配；
- 所有越界可检测；
- 中断和任务并发关系明确。

---

## 54. 数据结构阶段测试

### 54.1 实现题

1. 固定容量栈；
2. 环形队列；
3. 单链表插入、查找、删除、释放；
4. 二分查找；
5. 插入排序；
6. 二叉树前中后序；
7. 最小堆 push/pop；
8. 图的 BFS；
9. 哈希表开放寻址；
10. 用数组节点池代替 malloc 链表。

### 54.2 口述题

1. O(1) 是否一定比 O(n) 快？
2. 为什么链表理论插入 O(1)，实际不一定比 vector 好？
3. 环形队列如何判空判满？
4. 哈希表为什么最坏 O(n)？
5. BST 为什么会退化？
6. heap 内存和堆数据结构有什么区别？
7. BFS 与 DFS 各用什么辅助结构？
8. 二分查找需要什么前提？
9. 排序稳定是什么意思？
10. MCU 为什么偏爱固定容量结构？

### 54.3 通过标准

- 能独立实现数组、链表、栈、队列；
- 能说出核心操作复杂度；
- 能根据场景选择，而不是只背表；
- 所有固定结构处理满/空；
- 所有动态结构处理分配失败和释放；
- 能解释最坏情况。

---

# 第五部分：硬件与 MCU 基础

## 55. 安全先于知识

本教程默认低压直流实验。

禁止在没有专业训练和正确设备时直接测量：

- 市电；
- 高压电源；
- 大容量电容；
- 非隔离开关电源一次侧；
- 高能电池组；
- 未知接地点。

普通台式示波器探头地夹通常与保护地相连。夹错位置可能造成短路、设备损坏或触电。

低压实验也要注意：

- 限流；
- 防反接；
- 电池短路；
- 元件温升；
- 静电；
- 电机堵转电流；
- 焊接通风和烫伤。

---

## 56. 电压、电流、电阻与功率

### 56.1 电压

电压是两点间的电势差，单位伏特 V。说“这个点是 3.3 V”必须隐含相对某个参考地。

### 56.2 电流

电流是电荷流动速率，单位安培 A。

测电流时万用表通常串联进回路；误把电流档并联到电源两端可能造成短路。

### 56.3 电阻

电阻限制电流，单位欧姆 Ω。

欧姆定律：

$$
V = I R
$$

变形：

$$
I = V/R
$$

### 56.4 功率

$$
P = VI = I^2R = V^2/R
$$

元件不仅要阻值正确，还要功率额定值足够。

### 56.5 串联与并联

串联：

$$
R_{total}=R_1+R_2+\cdots
$$

并联：

$$
\frac{1}{R_{total}}=
\frac{1}{R_1}+\frac{1}{R_2}+\cdots
$$

### 56.6 基尔霍夫定律

KCL（电流定律）：

> 流入一个节点的电流总和等于流出的电流总和。

KVL（电压定律）：

> 沿一个闭合回路，各段电压升降的代数和为 0。

它们是分析分压、偏置、运放和电源回路的基础。欧姆定律描述元件，KCL/KVL 描述网络关系。

### 56.7 LED 限流例子

假设：

- 电源 3.3 V；
- LED 正向压降约 2.0 V；
- 目标电流 5 mA。

$$
R = (3.3-2.0)/0.005 = 260\ \Omega
$$

可选择接近的标准值 270 Ω，再核对实际亮度、GPIO 电流能力和 LED 数据手册。

LED 不是固定 2.0 V 电阻负载，正向压降随颜色、电流和温度变化。

---

## 57. 基础元件

### 57.1 电阻

用途：

- 限流；
- 分压；
- 上拉/下拉；
- 偏置；
- 终端匹配；
- 电流采样。

分压：

~~~text
Vin ─ R1 ─ Vout ─ R2 ─ GND
~~~

理想空载：

$$
V_{out}=V_{in}\frac{R_2}{R_1+R_2}
$$

接上负载后，负载与 R2 并联，输出会改变。

### 57.2 电容

电容储存电荷：

$$
Q=CV
$$

能量：

$$
E=\frac{1}{2}CV^2
$$

常见用途：

- 去耦；
- 滤波；
- 储能；
- 定时；
- AC 耦合；
- 稳压环路。

RC 时间常数：

$$
\tau=RC
$$

充放电经过约 5τ 接近最终值，但“接近”程度取决于要求。

电容不是理想元件，还存在：

- ESR；
- ESL；
- 漏电；
- 容值公差；
- 直流偏压导致陶瓷电容有效容值下降；
- 极性电容反接风险。

### 57.3 电感

$$
V=L\frac{di}{dt}
$$

电感阻止电流快速变化，储能：

$$
E=\frac{1}{2}LI^2
$$

用于：

- 开关电源；
- 滤波；
- EMI 抑制；
- 储能。

电感、电机、继电器断电时会产生反向电压，需要续流或钳位路径。

### 57.4 二极管

理想上单向导通；实际有正向压降、反向漏电、恢复时间和额定值。

用途：

- 整流；
- 防反接；
- 续流；
- 钳位；
- TVS 浪涌保护；
- 发光。

肖特基二极管通常正向压降低、恢复快，但反向漏电可能更大。

### 57.5 BJT

BJT 是电流控制型晶体管，可作开关或放大。

作为开关需考虑：

- 基极限流；
- 饱和压降；
- 集电极电流；
- 功耗；
- 关断速度。

### 57.6 MOSFET

MOSFET 栅极主要表现为电容负载，由栅源电压控制导通程度。

选 N 沟道 MOSFET 低边开关要看：

- 3.3 V 栅压下的 Rds(on)；
- 最大漏极电流；
- 功耗与散热；
- 栅极电荷；
- 漏源耐压；
- 封装；
- 负载瞬态。

Vgs(th) 只是刚开始有很小电流的阈值，不代表已经充分导通。

典型：

~~~text
电源正 ─ 负载 ─ Drain
Source ─ GND
GPIO ─ 栅极电阻 ─ Gate
Gate ─ 下拉电阻 ─ GND
~~~

### 57.7 运算放大器基础

理想运放常用近似：

- 输入电流为 0；
- 负反馈线性工作时两输入端电压接近；
- 开环增益很大。

实际要看：

- 输入共模范围；
- 输出摆幅；
- 供电；
- 带宽和转换速率；
- 输入失调；
- 噪声；
- 稳定性。

“轨到轨”也不表示在所有负载下能精确到达电源轨。

---

## 58. 模拟信号与数字信号

### 58.1 模拟

电压或电流在范围内连续变化，例如：

- 麦克风；
- 光敏分压；
- 温度传感器模拟输出；
- 电源纹波。

### 58.2 数字

数字系统把电压区间解释为逻辑 0 或 1。

重要参数：

- VIL(max)：保证识别为低的最高输入电压；
- VIH(min)：保证识别为高的最低输入电压；
- VOL(max)：输出低时的最高电压；
- VOH(min)：输出高时的最低电压；
- 绝对最大额定值。

绝对最大额定值不是正常工作范围。超过可能损坏，低于它也不代表功能一定正常。

### 58.3 3.3 V 与 5 V 兼容

不能只看“高低电平”：

- 5 V 输出可能损坏 3.3 V 输入；
- 3.3 V 高电平是否满足 5 V 输入 VIH 要查手册；
- 某些 MCU 引脚 5 V tolerant，但有模式和引脚限制；
- 上电顺序可能通过保护二极管反向供电。

方案：

- 电阻分压；
- 单向/双向电平转换；
- 开漏上拉到合适电压；
- 专用缓冲器。

### 58.4 逻辑门

| A | B | AND | OR | XOR |
|:---:|:---:|:---:|:---:|:---:|
| 0 | 0 | 0 | 0 | 0 |
| 0 | 1 | 0 | 1 | 1 |
| 1 | 0 | 0 | 1 | 1 |
| 1 | 1 | 1 | 1 | 0 |

NOT 把 0/1 反相。

位运算与逻辑运算不同：

~~~c
uint8_t bitwise = a & b;   // 每一位分别 AND
bool logical = a && b;     // 判断整体真假，结果为 0 或 1
~~~

### 58.5 组合逻辑与时序逻辑

组合逻辑：

- 输出只由当前输入决定；
- 例如加法器、译码器、多路选择器。

时序逻辑：

- 输出还与过去状态有关；
- 需要锁存器、触发器、寄存器和时钟；
- 例如计数器、状态机、串行移位。

### 58.6 建立时间、保持时间与亚稳态

触发器在时钟边沿附近要求输入稳定：

- 建立时间：边沿前；
- 保持时间：边沿后。

异步输入直接进入同步数字逻辑可能引发亚稳态。常见处理：

- 单 bit 慢速状态用两级同步器；
- 脉冲需保证宽度或使用握手/脉冲同步；
- 多 bit 总线用 FIFO、握手或专用 CDC 结构；
- 不能认为每一位各加两级同步就能得到一致多位数据。

MCU 的外部异步信号也需关注输入同步、滤波和中断边沿。

---

## 59. 上拉、下拉、推挽、开漏与消抖

### 59.1 浮空输入

没有明确电平的高阻输入会受噪声影响随机变化。

上拉：

~~~text
VCC ─ 电阻 ─ 输入节点 ─ 开关 ─ GND
~~~

开关松开读高，按下读低。

### 59.2 推挽输出

推挽能主动驱动高和低：

- 边沿快；
- 驱动能力较好；
- 不允许两个相反推挽输出直接相连。

### 59.3 开漏输出

开漏通常只能主动拉低，释放后靠上拉为高。

优点：

- 多设备共享；
- 线与；
- 适合电平转换；
- I2C 使用。

代价：

- 上升沿由 R 与总线电容决定；
- 上拉过大上升慢；
- 上拉过小低电平电流大。

### 59.4 按键抖动

机械触点在按下/释放瞬间会反复弹跳。

解决：

- RC + 施密特触发；
- 固定周期采样；
- 连续稳定若干次；
- 边沿后等待稳定；
- 状态机。

非阻塞软件消抖：

~~~c
typedef struct {
    bool stable_state;
    bool last_raw;
    uint32_t changed_at_ms;
} Debouncer;

bool debounce_update(Debouncer *d, bool raw, uint32_t now_ms)
{
    if (d == NULL) {
        return false;
    }

    if (raw != d->last_raw) {
        d->last_raw = raw;
        d->changed_at_ms = now_ms;
    }

    if (raw != d->stable_state &&
        (uint32_t)(now_ms - d->changed_at_ms) >= 20U) {
        d->stable_state = raw;
        return true;
    }

    return false;
}
~~~

返回 true 表示稳定状态发生变化。

使用前要把 stable_state、last_raw 和 changed_at_ms 初始化为当前真实输入与当前时间，不能使用未初始化结构。

---

## 60. 电源、去耦、地与保护

### 60.1 LDO

线性稳压器：

- 简单；
- 噪声可较低；
- 输入输出压差变成热。

近似功耗：

$$
P_{loss}=(V_{in}-V_{out})I
$$

例如 12 V 降 3.3 V、100 mA：

$$
P_{loss}=0.87 W
$$

小封装可能过热。

### 60.2 Buck

降压开关电源：

- 效率高；
- 适合大压差和大电流；
- 设计更复杂；
- 有开关纹波和 EMI；
- 电感、二极管/MOSFET、反馈布局关键。

### 60.3 去耦

MCU 每组电源脚附近放小陶瓷电容，提供高频瞬态电流，缩短回路。

板级还需要较大储能电容应对较慢负载变化。

不能只背“0.1 μF”：

- 看官方硬件指南；
- 关注每个 VDD；
- 放置距离和回流；
- VDDA 模拟电源；
- 电容有效容值；
- 负载瞬态。

### 60.4 地不是理想零电位

走线和铜皮有阻抗，大电流回流会产生压降和噪声。

原则：

- 完整地平面；
- 高频回路面积小；
- 大电流路径远离敏感 ADC；
- 去耦回路短；
- 不随意切割高速信号参考平面；
- 先理解回流，再谈“单点接地”。

### 60.5 保护

可能需要：

- 保险丝/PTC；
- 反接保护；
- TVS；
- ESD 器件；
- 过流；
- 过压；
- 欠压；
- 温度；
- 电机续流；
- 接口串联电阻。

保护器件的额定值、钳位电压和布局必须与系统能量匹配。

---

## 61. MCU 架构

### 61.1 MCU 内部

~~~text
CPU Core
├─ 寄存器
├─ ALU/FPU
├─ 中断控制
└─ 调试模块

片上系统
├─ Flash
├─ SRAM
├─ 时钟/复位
├─ GPIO
├─ Timer/PWM
├─ ADC/DAC
├─ DMA
├─ UART/I2C/SPI/CAN
└─ 看门狗/RTC
~~~

### 61.2 内存映射 I/O

外设寄存器映射到地址空间，CPU 用普通加载/存储指令访问。

~~~c
#define GPIO_ODR (*(volatile uint32_t *)0x40020014UL)
GPIO_ODR |= (1UL << 5);
~~~

这只是教学示意。真实工程使用芯片头文件定义，必须核对地址和寄存器语义。

### 61.3 总线

CPU、Flash、SRAM、DMA 和外设通过片上总线连接。

需要理解：

- 不同总线时钟；
- 外设挂在哪条总线；
- DMA 能否访问某段内存；
- 总线争用；
- 桥接和预分频。

### 61.4 Cortex-M

常见：

- Cortex-M0/M0+：低成本、低功耗；
- Cortex-M3：通用；
- Cortex-M4/M4F：DSP 扩展，可带 FPU；
- Cortex-M7：更高性能、缓存更复杂；
- Cortex-M33：支持较新安全架构等。

具体功能看芯片，不要只靠内核名称判断全部外设和资源。

---

## 62. 时钟、复位、启动与看门狗

### 62.1 时钟来源

- 内部 RC；
- 外部晶振；
- 低速 32.768 kHz；
- PLL 倍频；
- 分频器。

时钟树决定：

- CPU 主频；
- 总线频率；
- 外设频率；
- USB 等专用时钟；
- 功耗。

### 62.2 为什么先开外设时钟

很多 MCU 通过时钟门控节能。外设未开时钟，寄存器写入可能无效或行为不符合预期。

### 62.3 定时器倍频陷阱

一些 STM32 中，当 APB 预分频不为 1 时，定时器输入时钟可能是 PCLK 的 2 倍。必须查对应参考手册。

### 62.4 复位

来源：

- 上电复位；
- 外部复位脚；
- 软件复位；
- 看门狗；
- 欠压；
- 调试器。

复位原因寄存器应在启动时读取、记录，再按手册清除。

### 62.5 从复位到 main

1. 读取初始栈指针；
2. 进入 Reset_Handler；
3. 初始化时钟/底层；
4. 复制 .data；
5. 清零 .bss；
6. 初始化运行时；
7. 调用 main；
8. 启动主循环或 RTOS。

### 62.6 Boot

启动模式可能从：

- 用户 Flash；
- 系统 Bootloader；
- SRAM。

BOOT 引脚和选项字节决定具体行为。系统 Bootloader 可支持 UART/USB 等下载，取决于芯片。

### 62.7 看门狗

程序必须在规定时间内“喂狗”，否则复位。

正确使用：

- 只有系统关键任务都健康才喂；
- 记录复位原因；
- 超时时间覆盖正常最坏延迟；
- 不在一个永远运行的高优先级中无条件喂。

---

## 63. GPIO

常见模式：

- 输入；
- 推挽输出；
- 开漏输出；
- 模拟；
- 复用功能；
- 上拉/下拉；
- 输出速度/驱动强度。

### 63.1 输出 LED

流程：

1. 开 GPIO 时钟；
2. 配置模式；
3. 配置输出类型、速度、上下拉；
4. 设置初始电平；
5. 写置位/复位寄存器。

很多 STM32 提供原子置位/复位寄存器，比对 ODR 读改写更安全。

### 63.2 输入

按键：

- 明确有效高/低；
- 上拉/下拉；
- 消抖；
- 边沿事件；
- 长按/短按。

### 63.3 GPIO 驱动能力

数据手册规定：

- 单引脚电流；
- 端口总电流；
- 芯片总电流；
- 电压降；
- 速度和负载电容。

GPIO 不能直接驱动电机、大功率 LED、继电器等负载。

---

## 64. 中断与 NVIC

中断让 CPU 暂停当前流程处理事件，再恢复。

### 64.1 中断现场

Cortex-M 异常进入时硬件自动保存部分寄存器，软件和编译器再保存必要现场。

中断有：

- 向量号；
- 使能；
- 挂起；
- 优先级；
- 抢占；
- 中断标志。

### 64.2 优先级

通常数值越小优先级越高，但：

- 实现的优先级位数有限；
- 分组影响抢占和子优先级；
- RTOS 对可调用 API 的中断优先级有限制。

### 64.3 中断延迟

受：

- 当前是否关中断；
- 更高优先级 ISR；
- 指令完成；
- 总线和存储；
- 临界区；
- 缓存（高端内核）；
- RTOS 配置。

### 64.4 EXTI

外部中断可检测 GPIO 边沿：

- 上升沿；
- 下降沿；
- 双边沿。

机械按键即使用 EXTI 仍需消抖。

### 64.5 ISR 原则

~~~text
读数据 → 清标志 → 保存最小状态 → 通知主流程 → 返回
~~~

避免：

- while 永久等待；
- delay；
- malloc；
- 大量 printf；
- 复杂浮点算法；
- 调用非 ISR 安全的 RTOS API。

---

## 65. 定时器与 PWM

### 65.1 定时器计数

常见计数器由输入时钟经过 PSC 分频，再计到 ARR。

边沿对齐向上计数：

$$
f_{update}=\frac{f_{TIM}}{(PSC+1)(ARR+1)}
$$

### 65.2 PWM

$$
Duty\approx\frac{CCR}{ARR+1}
$$

例：TIM 时钟 72 MHz：

- PSC=71；
- ARR=999；
- CCR=250。

得到：

- 计数频率 1 MHz；
- PWM 1 kHz；
- 占空比约 25%。

### 65.3 PWM 应用

- LED 调光；
- 电机驱动；
- 舵机脉冲；
- 蜂鸣器；
- DAC 近似输出（需滤波）；
- 通信时序。

舵机通常以高电平脉宽编码位置，不能只说“占空比控制角度”而忽略周期和脉宽。

### 65.4 输入捕获

记录外部边沿到来时的计数值：

- 测频率；
- 测脉宽；
- HC-SR04 Echo；
- PWM 输入；
- 转速。

必须处理：

- 计数器溢出；
- 边沿切换；
- 超时；
- 输入滤波；
- 多次捕获。

### 65.5 输出比较

计数器达到比较值时：

- 翻转引脚；
- 产生中断；
- 触发 DMA；
- 调度精确事件。

### 65.6 编码器模式

定时器可解码正交 A/B 相，获得方向和计数。还需处理：

- 溢出；
- 零位；
- 速度计算；
- 毛刺；
- 机械分辨率和倍频。

---

## 66. ADC、DAC 与比较器

### 66.1 ADC

N 位 ADC 有 $2^N$ 个量化码。

理想单极性近似：

$$
Code\approx\frac{V_{in}}{V_{ref}}(2^N-1)
$$

12 位、3.3 V、Code=2048：

$$
V_{in}\approx2048/4095\times3.3\approx1.65V
$$

### 66.2 分辨率不等于精度

误差：

- 参考电压；
- 偏置/增益；
- 非线性；
- 量化；
- 电源噪声；
- PCB；
- 传感器；
- 源阻抗；
- 采样时间；
- 温漂。

### 66.3 采样保持

ADC 内部采样电容需要通过信号源阻抗充电。源阻抗高而采样时间短，会达不到目标精度。

解决：

- 延长采样时间；
- 降低源阻抗；
- 运放缓冲；
- 按官方公式设计。

### 66.4 采样定理

理论上采样率大于最高频率 2 倍才能避免基本混叠条件，但工程中需要：

- 抗混叠模拟滤波；
- 更高采样余量；
- 考虑滤波器过渡带；
- 关注幅相精度。

### 66.5 滤波

- 均值；
- 滑动平均；
- 中值；
- 一阶低通；
- 数字滤波器；
- 硬件 RC。

滤波会带来延迟，不能只追求“越平越好”。

### 66.6 DAC

DAC 把数字码转换为模拟电压或电流。

若芯片没有 DAC，可用：

- PWM + 低通滤波；
- 外部 I2C/SPI DAC。

PWM 滤波输出存在纹波、带宽和负载问题，不等同于高质量 DAC。

### 66.7 比较器

比较两个模拟电压并输出数字结果：

- 阈值检测；
- 过流保护；
- 零交叉；
- 迟滞。

加入正反馈形成迟滞可避免阈值附近噪声反复翻转。

---

## 67. DMA

DMA 在外设和内存之间搬运数据，降低 CPU 逐项搬运负担。

典型：

~~~text
定时器触发 ADC
→ ADC 转换
→ DMA 循环写数组
→ 半传输/全传输中断
→ CPU 处理对应半区
~~~

### 67.1 配置要素

- 源地址；
- 目标地址；
- 传输方向；
- 数据宽度；
- 长度；
- 地址是否递增；
- 普通/循环；
- 优先级；
- 触发请求；
- 完成/错误中断。

### 67.2 双缓冲

CPU 处理 A 时 DMA 写 B，之后交换。

必须保证：

- CPU 处理速度赶上数据产生；
- 缓冲区所有权清楚；
- 缓存一致性（有 D-Cache 的 MCU）；
- 半满/全满事件不丢；
- 错误和溢出有统计。

### 67.3 DMA 不是自动完成全部工作

DMA 不会自动：

- 解析协议；
- 检查业务校验；
- 解决并发；
- 防止缓冲区被覆盖；
- 保证缓存一致；
- 恢复外设错误。

---

## 68. UART / USART

UART 是异步串行通信，双方约定：

- 波特率；
- 数据位；
- 校验；
- 停止位。

常见 8N1：

~~~text
空闲高 → 起始位0 → 8数据位 → 无校验 → 1停止位1
~~~

### 68.1 连接

~~~text
设备A TX → 设备B RX
设备A RX ← 设备B TX
GND      ↔ GND
~~~

TX/RX 交叉，必须共地。

### 68.2 波特率与吞吐

115200、8N1 每个有效字节约 10 bit：

$$
115200/10=11520\ bytes/s
$$

实际还要扣帧头、校验、空闲和软件处理。

### 68.3 TTL、RS-232、RS-485

- MCU UART 引脚通常是 3.3 V/5 V TTL/CMOS 电平；
- RS-232 使用不同电压和极性，需要收发器；
- RS-485 常为差分、适合较长线和多节点，需要收发器与终端设计。

不能把 MCU TX 直接接传统 RS-232 接口。

### 68.4 接收方式

| 方式 | 特点 |
|---|---|
| 轮询 | 简单，可能占 CPU 和丢数据 |
| 字节中断 | 响应及时，高速时中断多 |
| DMA 定长 | 高效，适合固定帧 |
| DMA + 空闲线 | 适合变长流，需正确管理缓冲 |

### 68.5 UART 没有天然帧边界

自定义帧：

~~~text
AA 55 | LEN | CMD | PAYLOAD | CRC
~~~

解析器必须处理：

- 半包；
- 多包粘连；
- 丢字节；
- 长度越界；
- 校验失败；
- 超时；
- 重新同步。

### 68.6 常见错误

- 波特率不一致；
- 时钟配置错；
- TX/RX 未交叉；
- 未共地；
- 电平标准错；
- 溢出标志未处理；
- 接收缓冲太小；
- printf 阻塞；
- 字节序和类型宽度错误。

逻辑分析仪适合看帧和波特率，示波器适合看电平、边沿与噪声。

---

## 69. I2C

I2C 是同步、地址式、半双工总线：

- SCL：时钟；
- SDA：数据；
- 开漏；
- 外部上拉；
- 多个设备共享。

### 69.1 时序

- 空闲：SCL/SDA 高；
- START：SCL 高时 SDA 高到低；
- STOP：SCL 高时 SDA 低到高；
- 每 8 bit 后第 9 位 ACK/NACK。

### 69.2 地址

常用 7 位地址。传输时再附加读写位。

最大常见混乱：

- 数据手册给 7 位地址 0x68；
- 某些库要求已经左移后的值；
- 不同 API 约定不同。

必须阅读 API 文档，不能盲目左移。

### 69.3 读寄存器

~~~text
START
地址+W → ACK
寄存器地址 → ACK
Repeated START
地址+R → ACK
读取数据
主机最后 NACK
STOP
~~~

### 69.4 上拉计算思想

总线上升时间由上拉电阻和总线电容决定：

- R 太大：上升慢；
- R 太小：低电平电流大；
- 速率越高、总线电容越大，对 R 要求越严格。

按 I2C 规范和器件灌电流能力计算，不只凭经验放 4.7 kΩ。

### 69.5 多主机与仲裁

开漏让主机在发送高时观察总线：

- 想释放为高却读到低，说明其他主机发送低；
- 失去仲裁的主机停止发送；
- 总线不会因两个主机相反强驱而短路。

### 69.6 Clock Stretching

从机可拉低 SCL 延长时钟，要求主机控制器和软件支持。不是所有实现都正确支持。

### 69.7 总线卡死

从机可能在未完成事务时一直拉低 SDA。

恢复思路：

- 超时；
- 检查 SDA/SCL；
- 临时用 GPIO 给 SCL 若干脉冲；
- 产生 STOP；
- 复位外设或从机；
- 记录故障。

具体流程看器件要求。

---

## 70. SPI

常见信号：

- SCLK；
- MOSI；
- MISO；
- CS/SS。

特点：

- 同步；
- 常见全双工；
- 推挽，速度可高；
- 没有统一地址；
- 每个从机常需片选；
- 没有 I2C 式通用 ACK。

### 70.1 CPOL 与 CPHA

- CPOL：空闲时钟电平；
- CPHA：在哪个边沿采样。

组合形成 Mode 0–3。主从必须一致。

### 70.2 全双工本质

SPI 每个时钟同时移出和移入一位。即使只想读，也常要发送 dummy 字节来产生时钟。

### 70.3 片选

不同设备对 CS 有要求：

- 每个命令前后拉高/拉低；
- 连续多字节期间保持低；
- CS 建立/保持时间；
- 事务间隔。

### 70.4 常见故障

- 模式错；
- MSB/LSB 顺序错；
- CS 时序错；
- 时钟过快；
- MISO 多设备冲突；
- 电平不兼容；
- 长线振铃；
- DMA 长度错。

---

## 71. UART、I2C、SPI 对比

| 项目 | UART | I2C | SPI |
|---|---|---|---|
| 时钟线 | 无 | 有 | 有 |
| 数据线 | TX/RX | SDA | MOSI/MISO |
| 典型双工 | 全双工 | 半双工 | 全双工 |
| 设备寻址 | 协议自定义 | 统一地址 | 片选 |
| 电气 | 常推挽 | 开漏+上拉 | 常推挽 |
| ACK | 协议自定义 | 有 | 无统一 ACK |
| 典型用途 | 模块/调试/远距离转换 | 板内多传感器 | 高速 ADC/Flash/显示 |

选择取决于：

- 速度；
- 引脚；
- 距离；
- 设备数量；
- 错误检测；
- 现有器件接口。

---

## 72. CAN

CAN 是面向消息标识符的差分总线，常用于汽车和工业。

### 72.1 物理层

- CAN_H/CAN_L 差分；
- MCU 需要 CAN 控制器和外部收发器；
- 总线两端通常各 120 Ω 终端；
- 终端不是每个节点都放；
- 支线长度受速率影响。

### 72.2 仲裁

显性位覆盖隐性位。多个节点同时发送时：

- 标识符数值较小通常优先级更高；
- 失去仲裁的节点停止发送；
- 高优先级消息不被破坏。

### 72.3 标识符不是设备地址

标识符描述消息含义和优先级，多个节点可关注同一消息。

### 72.4 错误处理

CAN 有：

- 位监测；
- CRC；
- ACK；
- 格式检查；
- 错误计数；
- error active/passive；
- bus off。

上层仍需设计：

- 超时；
- 新鲜度；
- 计数器；
- 节点状态；
- 安全降级。

### 72.5 CAN FD

CAN FD 支持更大数据段和数据阶段更高比特率，但控制器、收发器和网络设计要兼容。

---

## 73. USB、以太网与无线概览

### 73.1 USB

USB 区分：

- Host；
- Device；
- OTG；
- Endpoint；
- Enumeration；
- Class，例如 CDC、HID、MSC。

USB 不是“更快的 UART”。它有主机调度、描述符、端点和协议栈。

USB D+/D- 是差分高速信号：

- 阻抗；
- 等长；
- ESD；
- 连接器；
- 参考平面；
- 布局都重要。

### 73.2 以太网

分层理解：

~~~text
应用：HTTP/MQTT
传输：TCP/UDP
网络：IP
链路：Ethernet MAC
物理：PHY + 变压器/接口
~~~

MAC 与 PHY 不同：

- MAC 处理帧；
- PHY 处理物理信号；
- 常通过 RMII/MII 连接。

### 73.3 Wi-Fi

- STA：接入路由器；
- AP：自己提供热点；
- DHCP：获取 IP；
- DNS：域名解析；
- TCP/UDP：传输。

无线稳定性还受：

- 天线；
- 地和净空；
- 供电峰值；
- 信道干扰；
- 路由器；
- 重连策略。

### 73.4 经典蓝牙与 BLE

经典蓝牙常见 SPP 等模型；BLE 使用：

- GAP：发现、广播、连接；
- GATT：Service、Characteristic；
- Notification/Indication；
- 连接间隔和低功耗。

Web Bluetooth 常面向 BLE GATT，不等于浏览器能直接访问任意经典蓝牙串口。

---

## 74. 裸机调度与 RTOS 桥梁

### 74.1 前后台系统

~~~text
中断：快速采集/通知
主循环：处理状态机、协议和输出
~~~

### 74.2 时间片式协作调度

~~~c
for (;;) {
    uint32_t now = millis();

    if ((uint32_t)(now - last_sensor) >= 10U) {
        last_sensor = now;
        sensor_task();
    }

    if ((uint32_t)(now - last_display) >= 100U) {
        last_display = now;
        display_task();
    }

    uart_task();
}
~~~

所有 task 都必须快速返回。

### 74.3 什么时候考虑 RTOS

- 多个并发活动；
- 不同优先级和周期；
- 阻塞式等待希望释放 CPU；
- 需要队列、信号量、互斥；
- 系统复杂度已超过简单状态机。

RTOS 不会自动修复糟糕架构，还会增加：

- 任务栈；
- 并发；
- 优先级；
- 死锁；
- 调度调试。

---

## 75. 传感器与执行器

### 75.1 传感器链路

~~~text
物理量
→ 敏感元件
→ 模拟前端/数字处理
→ ADC/数字接口
→ 标定
→ 滤波
→ 业务值
~~~

必须区分：

- 分辨率；
- 精度；
- 量程；
- 灵敏度；
- 零偏；
- 漂移；
- 重复性；
- 带宽；
- 采样率。

### 75.2 标定

单点：修正偏置。  
两点：修正偏置与增益。  
多点：拟合非线性。

标定需要参考标准、温度条件和可追溯记录，不能只把一次读数设为“正确值”。

### 75.3 常见传感器

- 温度：热敏电阻、数字温度；
- 光照：光敏电阻、光电二极管、数字光强；
- IMU：加速度、陀螺仪；
- 距离：超声波、ToF、红外；
- 压力；
- 电流；
- 编码器。

### 75.4 执行器

- LED；
- 蜂鸣器；
- 继电器；
- 直流电机；
- 步进电机；
- 舵机；
- 电磁阀。

MCU 通常只输出控制信号，需要：

- MOSFET/驱动芯片；
- 独立电源；
- 续流；
- 过流；
- 反馈；
- 急停。

### 75.5 电机基础

直流电机：

- PWM 控速；
- H 桥换向；
- 启动/堵转电流大；
- 反电动势；
- 编码器闭环。

步进电机：

- 脉冲控制步数；
- 加减速；
- 可能失步；
- 驱动器限流；
- 微步不等于绝对精度同等提高。

舵机：

- 内部通常有位置闭环；
- 外部给位置脉宽；
- 主控未必能读实际角度；
- 供电和堵转电流重要。

---

## 76. 如何阅读数据手册

### 76.1 文档类型

| 文档 | 主要内容 |
|---|---|
| Datasheet | 引脚、资源、电气参数、封装、时序 |
| Reference Manual | 外设原理与寄存器 |
| Programming Manual | 内核和指令相关内容 |
| User Manual | 开发板、软件或工具使用 |
| Application Note | 某种设计和应用方法 |
| Errata | 已知芯片问题和规避方式 |

### 76.2 读 MCU 数据手册的顺序

1. 完整料号；
2. 特性摘要；
3. 引脚和封装；
4. 内存；
5. 时钟；
6. 外设映射；
7. 工作条件；
8. 绝对最大值；
9. DC 电气；
10. AC 时序；
11. 典型应用；
12. 勘误。

### 76.3 绝对最大值与推荐工作条件

绝对最大值：

> 超过可能永久损坏，不保证接近上限能正常运行。

推荐工作条件：

> 厂商保证功能和参数的正常范围。

设计必须留裕量，不把正常工作点贴着绝对最大值。

### 76.4 典型值不是保证值

Typical 只代表典型样品/条件。量产设计应重点看 min/max 和条件。

### 76.5 读寄存器

确认：

- 地址偏移；
- 复位值；
- 位宽；
- 读写权限；
- 写 1 清零；
- 保留位；
- 修改前置条件；
- 标志清除顺序；
- 勘误。

不要把保留位随意写 1。

### 76.6 时序图

读时序图先找：

- 参考边沿；
- 建立时间；
- 保持时间；
- 最大频率；
- 有效窗口；
- 测试负载和电压条件。

---

## 77. 如何阅读原理图

原理图表达电气连接，不一定表达物理位置。

### 77.1 先分功能块

~~~text
电源入口
→ 稳压与保护
→ MCU 最小系统
→ 下载调试
→ 传感器接口
→ 通信接口
→ 执行器驱动
~~~

### 77.2 网络名

同名网络在电气上连接，即使图上没有画长线。

常见：

- 3V3；
- 5V；
- GND；
- SWDIO；
- SWCLK；
- UART_TX；
- I2C_SCL。

### 77.3 连接点与交叉

线路交叉不一定相连，要看连接点标记和 EDA 规则。

### 77.4 检查清单

- 电源脚是否全部连接；
- 去耦是否完整；
- 模拟电源；
- 复位；
- BOOT；
- 晶振；
- SWD；
- 上拉/下拉；
- 接口电压；
- 未使用引脚；
- 器件方向；
- 测试点；
- 保护。

---

## 78. PCB 设计基础

### 78.1 流程

~~~text
需求
→ 原理图
→ ERC
→ 封装核对
→ 板框/层叠
→ 布局
→ 布线
→ 地与电源
→ DRC
→ Gerber/BOM/CPL
→ 制造装配
→ 上电调试
~~~

### 78.2 封装核对

重点：

- 引脚编号；
- Pin 1；
- 焊盘尺寸；
- 间距；
- 丝印方向；
- 连接器正反面；
- 3D 模型仅辅助，不能替代尺寸检查。

封装错可能让整板无法装配。

### 78.3 布局优先于布线

先放：

- 连接器；
- 电源；
- MCU；
- 去耦；
- 晶振；
- 高速接口；
- 功率器件；
- 模拟前端。

按功能和电流路径组织，不能为了“看起来整齐”牺牲回路。

### 78.4 去耦布局

~~~text
VDD 引脚 → 电容 → GND 回流
~~~

路径要短、环路面积小。电容“在原理图上有”不代表 PCB 去耦有效。

### 78.5 晶振

- 靠近 MCU；
- 两根线短而对称；
- 远离高速和大电流开关；
- 负载电容按晶体、寄生和厂家建议；
- 不随意在下方穿高速信号。

### 78.6 线宽与过孔

按：

- 电流；
- 铜厚；
- 允许温升；
- 压降；
- 制造能力；
- 阻抗需求。

高电流不能使用细长线和大量小过孔而不计算。

### 78.7 地平面与回流

信号电流需要回流路径。高速边沿更倾向沿信号下方参考平面返回。

跨越地平面分割会迫使回流绕路，增大环路和 EMI。

### 78.8 模拟与数字

不要简单把地切成两半。更重要的是：

- 分区布局；
- 控制回流；
- 模拟输入远离开关节点；
- 参考和地干净；
- ADC 驱动满足要求；
- 按芯片官方指南连接模拟地/电源。

### 78.9 差分线

USB、以太网等差分信号关注：

- 差分阻抗；
- 成对；
- 间距；
- 参考平面；
- 过孔；
- 长度匹配；
- 连接器和 ESD 布局。

长度匹配不是唯一目标，阻抗和连续回流同样重要。

### 78.10 天线

无线模块：

- 天线净空；
- 不铺铜区域；
- 远离金属和高噪声；
- 放板边；
- 按模块指南；
- 外壳也会影响。

### 78.11 生产文件

- Gerber：各制造图层；
- Drill：钻孔；
- BOM：物料；
- CPL/Pick and Place：贴片坐标；
- 装配图；
- 测试说明。

出板前使用独立 Gerber 查看器检查，不只相信 EDA 画布。

---

## 79. 测量与调试工具

### 79.1 万用表

测：

- 电压；
- 电阻；
- 通断；
- 电流；
- 二极管压降。

注意：

- 测电阻/通断前断电；
- 电流档需串联；
- 表笔插孔正确；
- 量程正确；
- 电容放电；
- 通断蜂鸣不代表高频下阻抗为零。

### 79.2 可调电源

首次上电：

- 设置正确电压；
- 设小电流限流；
- 观察电流；
- 异常立即断电；
- 逐步开放负载。

限流不是完整保护，但能显著降低首次上电损坏风险。

### 79.3 示波器

观察模拟电压随时间：

- 幅值；
- 周期；
- 脉宽；
- 上升/下降时间；
- 纹波；
- 噪声；
- 振铃；
- 时序关系。

核心设置：

- 探头倍率；
- 垂直档位；
- 时基；
- 采样率；
- 带宽限制；
- 触发源/边沿/电平；
- AC/DC 耦合。

短地弹簧比长地夹更适合高速边沿，减少测量环路产生的假振铃。

### 79.4 逻辑分析仪

适合：

- UART；
- I2C；
- SPI；
- 多通道数字时序；
- 长时间捕获；
- 协议解码。

限制：

- 只按阈值判断高低；
- 看不到真实模拟电平质量；
- 解码正常不代表电气裕量足够。

### 79.5 调试器

SWD/JTAG 可：

- 下载；
- 断点；
- 单步；
- 查看寄存器和内存；
- Watch；
- 调用栈；
- 复位控制；
- SWO/Trace（视芯片和工具）。

断点会改变实时系统时序。电机或通信运行中暂停 CPU 可能产生危险或让外设继续运行。

### 79.6 焊接与返修

基础：

- 正确温度；
- 助焊剂；
- 清洁烙铁头；
- 控制加热时间；
- 检查虚焊、连锡、方向；
- ESD 防护；
- 通风。

---

## 80. 新 PCB 首次上电

### 80.1 上电前

1. 对照原理图和 PCB；
2. 检查器件方向；
3. 放大检查焊点；
4. 测电源对地阻值；
5. 检查短路；
6. 不先接昂贵负载；
7. 设置限流电源。

### 80.2 分阶段

1. 只验证输入电源；
2. 测各稳压输出；
3. 查 MCU 复位脚；
4. SWD 连接并读芯片；
5. 下载最小 GPIO 程序；
6. 验证时钟；
7. 打开调试 UART；
8. 一次接一个外设；
9. 最后接功率负载。

### 80.3 最小程序

最初只做：

- 初始化时钟；
- LED 慢闪；
- UART 打印固定字符串；
- 看门狗暂按计划配置。

不要第一版固件一次启动所有模块，否则故障范围太大。

---

## 81. 分层排错法

### 81.1 七层

~~~text
1. 电源
2. 时钟/复位
3. 引脚与电平
4. 外设波形
5. 驱动寄存器
6. 协议与数据
7. 应用状态机
~~~

### 81.2 证据链

~~~text
现象
→ 提出假设
→ 设计测量
→ 获得证据
→ 排除/确认
→ 修复
→ 回归验证
~~~

### 81.3 完全没反应

检查：

- 输入电压/电流；
- 3.3 V；
- 复位；
- Boot；
- SWD；
- 芯片方向；
- 晶振；
- 固件是否下载；
- GPIO 是否接错。

### 81.4 偶发复位

可能：

- 电源压降；
- 看门狗；
- HardFault；
- EMI；
- 栈溢出；
- 越界；
- 温度；
- 连接器接触；
- 电机启动电流。

要读取复位原因、保留日志并测电源波形。

### 81.5 通信偶发错误

检查：

- 物理电平；
- 终端/上拉；
- 线长；
- 共地；
- 波特率/时钟；
- 缓冲区；
- 溢出；
- DMA 边界；
- 竞态；
- 校验；
- EMI。

### 81.6 HardFault

Cortex-M 可查看：

- 堆栈中的 PC/LR；
- CFSR；
- HFSR；
- BFAR；
- MMFAR；
- map 文件；
- 反汇编。

常见根因：

- 空指针；
- 越界；
- 栈溢出；
- 非法地址；
- 对齐；
- 错误函数指针；
- 已释放对象。

---

## 82. 从零开始的 12 个实验

### 实验 1：PC Hello World

目标：

- 安装编译器；
- 编译 C/C++；
- 故意制造错误；
- 使用调试器。

### 实验 2：命令行计算器

内容：

- 输入；
- 运算符；
- switch；
- 除零；
- 错误处理。

### 实验 3：数组与统计

- 读取一组数；
- 最大/最小/平均；
- 排序；
- 二分查找；
- 写测试。

### 实验 4：数据结构库

- 固定栈；
- 环形队列；
- 链表；
- 单元测试；
- sanitizer。

### 实验 5：GPIO 与按键

- LED；
- 上拉按键；
- 消抖；
- 短按/长按；
- 非阻塞主循环。

### 实验 6：定时器 PWM

- 计算 PSC/ARR/CCR；
- 控制 LED；
- 示波器测频率；
- 比较计算与实测。

### 实验 7：ADC

- 电位器或光敏分压；
- ADC 采样；
- 转电压；
- 滑动平均；
- 串口输出。

### 实验 8：UART 协议

- 环形缓冲；
- 帧状态机；
- CRC/校验；
- 半包、粘包；
- 超时和错误统计。

### 实验 9：I2C 传感器

- WHO_AM_I；
- 配置寄存器；
- 连续读取；
- 逻辑分析仪波形；
- 断线和总线恢复。

### 实验 10：SPI 显示/Flash

- Mode；
- CS；
- 多字节读写；
- DMA；
- 示波器/逻辑分析。

### 实验 11：C++ 硬件封装

- C 驱动；
- C++ 类；
- RAII；
- std::array；
- 无动态分配；
- 测 Flash/RAM。

### 实验 12：综合系统

~~~text
传感器 → 采样/DMA → 过滤 → 状态机
→ PWM/显示 → UART 命令 → 错误与看门狗
~~~

要求：

- 原理图；
- 接口表；
- 状态图；
- 测试用例；
- 波形；
- 错误日志；
- Git 提交；
- 项目说明。

---

## 83. 综合闭卷题

### 83.1 C

1. Hello World 从源文件到运行经历什么？
2. int 为什么不能假设固定 32 位？
3. 数组与指针区别？
4. C 字符串为什么需要 '\0'？
5. const int* 与 int* const？
6. static、extern、volatile？
7. 栈、堆、.data、.bss？
8. struct 对齐为什么影响通信？
9. 宏参数多次求值是什么问题？
10. 未定义行为举五例。

### 83.2 C++

11. 引用与指针如何选择？
12. 构造、析构和初始化列表？
13. RAII？
14. Rule of Zero？
15. 拷贝与移动？
16. 虚函数和虚析构？
17. 模板的收益和代码膨胀？
18. vector size/capacity 和失效规则？
19. unique_ptr/shared_ptr/weak_ptr？
20. 哪些 C++ 特性在 MCU 上需评估？

### 83.3 数据结构

21. Big O 说明什么、不说明什么？
22. 数组和链表如何选？
23. 栈和队列的区别？
24. 环形队列如何判满？
25. 哈希冲突怎么处理？
26. BST 为什么最坏 O(n)？
27. 最小堆适合什么？
28. BFS/DFS？
29. 二分查找前提？
30. 常见排序对比？

### 83.4 硬件

31. 欧姆定律与功率？
32. LED 限流怎么算？
33. 去耦电容为什么靠近芯片？
34. 推挽与开漏？
35. 3.3 V/5 V 如何判断兼容？
36. MCU 上电到 main？
37. GPIO、中断、定时器？
38. PWM 频率怎么算？
39. ADC 分辨率与精度？
40. DMA 解决什么？
41. UART 8N1？
42. I2C START/ACK/上拉？
43. SPI CPOL/CPHA？
44. CAN 仲裁与终端？
45. 示波器和逻辑分析仪区别？
46. 首次上电流程？
47. HardFault 怎么查？
48. 数据手册和参考手册区别？
49. PCB 回流路径是什么？
50. 电机为什么不能由 GPIO 直接驱动？

### 83.5 回答结构

每题：

1. 一句定义；
2. 原理；
3. 一个计算/代码/波形；
4. 一个常见错误；
5. 一个项目场景。

如果 50 题中有 40 题能脱稿讲清，说明基础框架已建立。

---

## 84. 常用公式速查

### 电路

$$
V=IR
$$

$$
P=VI=I^2R=V^2/R
$$

$$
V_{out}=V_{in}\frac{R_2}{R_1+R_2}
$$

$$
\tau=RC
$$

### 定时器与 PWM

$$
f=\frac{f_{TIM}}{(PSC+1)(ARR+1)}
$$

$$
Duty\approx\frac{CCR}{ARR+1}
$$

### ADC

$$
Code\approx\frac{V_{in}}{V_{ref}}(2^N-1)
$$

### UART

8N1 理论有效字节率：

$$
Bytes/s\approx Baud/10
$$

### 超声波

$$
distance=\frac{time\times sound\ speed}{2}
$$

### 复杂度

~~~text
数组下标 O(1)
线性查找 O(n)
二分查找 O(log n)
链表查找 O(n)
哈希平均 O(1)，最坏 O(n)
平衡 BST O(log n)
堆 push/pop O(log n)
BFS/DFS O(V+E)
高效比较排序典型 O(n log n)
~~~

---

## 85. 推荐资料

### C / C++

- [C language reference（cppreference）](https://en.cppreference.com/w/c/language.html)
- [C++ language reference（cppreference）](https://en.cppreference.com/w/cpp/language.html)
- [C++ standard library](https://en.cppreference.com/w/cpp/standard_library.html)
- [C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines)
- [SEI CERT C Coding Standard](https://wiki.sei.cmu.edu/confluence/display/c/SEI+CERT+C+Coding+Standard)
- [GCC Documentation](https://gcc.gnu.org/onlinedocs/)
- [GDB Documentation](https://sourceware.org/gdb/documentation/)

### 数据结构与算法

- [Algorithms, 4th Edition 官方站点](https://algs4.cs.princeton.edu/home/)
- [C++ algorithms library](https://en.cppreference.com/w/cpp/algorithm.html)
- [C++ containers library](https://en.cppreference.com/w/cpp/container.html)

### MCU 与硬件

- [Arm Cortex-M4 Generic User Guide](https://developer.arm.com/documentation/dui0553/latest/)
- [CMSIS Core](https://arm-software.github.io/CMSIS_6/latest/Core/index.html)
- [STM32F407/417 官方文档页](https://www.st.com/en/microcontrollers-microprocessors/stm32f407-417/documentation.html)
- [STM32F4 RM0090](https://www.st.com/resource/en/reference_manual/rm0090-stm32f405415-stm32f407417-stm32f427437-and-stm32f429439-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)
- [STM32F1 RM0008](https://www.st.com/resource/en/reference_manual/cd00171190-stm32f101xx-stm32f102xx-stm32f103xx-stm32f105xx-and-stm32f107xx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)
- [STM32 Timer Cookbook AN4776](https://www.st.com/resource/en/application_note/an4776-general-purpose-timer-cookbook-for-stm32-microcontrollers-stmicroelectronics.pdf)
- [I2C Specification UM10204（NXP）](https://www.nxp.com/docs/en/user-guide/UM10204.pdf)
- [FreeRTOS Kernel Documentation](https://www.freertos.org/Documentation/02-Kernel/01-About-the-FreeRTOS-kernel/01-FreeRTOS-kernel)
- [Tektronix Oscilloscope Primer](https://www.tek.com/en/documents/primer/xyzs-oscilloscopes-primer)
- [Saleae Logic Analyzer Learning Portal](https://support.saleae.com/tutorials/learning-portal)

学习资料的用途：

- cppreference 查语法和库；
- 编译器文档查实现；
- 数据手册查电气和引脚；
- 参考手册查寄存器；
- 应用笔记查设计方法；
- 勘误查已知问题。

---

## 86. 最终掌握清单

### 编程

- [ ] 能独立创建、编译和调试 C/C++ 程序；
- [ ] 理解类型、转换、控制流和函数；
- [ ] 理解数组、字符串、指针与生命周期；
- [ ] 会模块化、错误处理和单元测试；
- [ ] 能解释未定义行为；
- [ ] 理解类、RAII、拷贝、移动和多态；
- [ ] 会选择标准容器和算法；
- [ ] 知道 MCU 上哪些 C++ 特性需要评估。

### 数据结构

- [ ] 会实现数组、链表、栈、队列；
- [ ] 理解哈希、树、堆、图；
- [ ] 会线性和二分查找；
- [ ] 理解主要排序；
- [ ] 会分析时间和空间复杂度；
- [ ] 能按嵌入式约束选择固定容量结构。

### 硬件

- [ ] 会用欧姆定律和功率公式；
- [ ] 认识常用阻容、二极管、MOSFET；
- [ ] 理解数字电平、上拉、推挽和开漏；
- [ ] 理解电源、去耦、回流和保护；
- [ ] 能说清 MCU 启动、时钟、GPIO 和中断；
- [ ] 会计算定时器、PWM 和 ADC；
- [ ] 理解 DMA；
- [ ] 能比较 UART、I2C、SPI、CAN；
- [ ] 会读数据手册和原理图；
- [ ] 会使用万用表、示波器、逻辑分析仪和调试器；
- [ ] 能按分层方法完成新板上电和故障定位。

### 真正完成的标准

> 不看文档，重新实现核心代码；不用背稿，画出系统框图；不用猜测，用编译器、调试器和仪器给出证据。
