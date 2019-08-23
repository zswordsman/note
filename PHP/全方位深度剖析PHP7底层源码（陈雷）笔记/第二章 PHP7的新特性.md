# 第二章 PHP7的新特性

1. PHP7的编译和安装
    * .configure (主要的几项）
        *  --prefix  制定安装目录
        *  --enable-fpm 编译安装php-fpm
        *  --enable-debug 开启debug
    * make && make install
        安装之前 解压之后 主要关注几个文件夹  **zend ext sapi** 
        安装之后 主要关注 **bin sbin**目录
    * zend目录下 有 bench.php、micro_bench.php 可以 基准测试 来比较php7与php5的性能差距。
2. PHP7的新特性
    * 标量 和 返回值类型声明  
```php
function arraysSum(array ...$arrays): array
{
    return array_map(function(array $array): int {
        return array_sum($array);
    }, $arrays);
}
``` 

    * null合并运算符 
    如果变量存在且值不为NULL， 它就会返回自身的值，否则返回它的第二个操作数。
```php
$username = $_GET['user'] ?? 'nobody';
// This is equivalent to:
$username = isset($_GET['user']) ? $_GET['user'] : 'nobody';
```
    * 太空船操作符（组合比较符）
```php
    // 整数
    echo 1 <=> 1; // 0
    echo 1 <=> 2; // -1
    echo 2 <=> 1; // 1
    // 浮点数
    echo 1.5 <=> 1.5; // 0
    echo 1.5 <=> 2.5; // -1
    echo 2.5 <=> 1.5; // 1
    // 字符串
    echo "a" <=> "a"; // 0
    echo "a" <=> "b"; // -1
    echo "b" <=> "a"; // 1
```
    * 通过 define() 定义常量数组
    
    * 为unserialize()提供过滤
```php
// 将所有的对象都转换为 __PHP_Incomplete_Class 对象
$data = unserialize($foo, ["allowed_classes" => false]);
// 将除 MyClass 和 MyClass2 之外的所有对象都转换为 __PHP_Incomplete_Class 对象
$data = unserialize($foo, ["allowed_classes" => ["MyClass", "MyClass2"]);
// 默认情况下所有的类都是可接受的，等同于省略第二个参数
$data = unserialize($foo, ["allowed_classes" => true]);
```
   * 整数除法函数 intdiv()
```php
var_dump(intdiv(10, 3));
//输出
int(3)
```
  * AST抽象语法树
       * PHP7 的内核中有一个重要的变化是加入了 AST。在 PHP5中，从 php 脚本到 opcodes 的执行的过程是：
            1. Lexing：词法扫描分析，将源文件转换成 token 流；
            2. Parsing：语法分析，在此阶段生成 op arrays。
       * PHP7 中在语法分析阶段不再直接生成 op arrays，而是先生成 AST，所以过程多了一步：
            1. Lexing：词法扫描分析，将源文件转换成 token 流；
            2. Parsing：语法分析，从 token 流生成抽象语法树；
            3. Compilation：从抽象语法树生成 op arrays。
    
未尽事宜 可参阅：[官方文档](https://www.php.net/manual/zh/migration70.new-features.php)
