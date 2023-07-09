# bitflags
1900017702 朱越
## 项目信息
[GitHub仓库链接](https://github.com/bitflags/bitflags/tree/main)
截止我选定该crate，它在crates.io上的Most Download里排名第11位，属于较为流行的Library。

## 使用方法
官方在主页上提供的使用样例如下：
1. 在Cargo.toml中添加依赖
    ```toml
    [dependencies]
    bitflags = "2.3.3"
    ```

2. 在源文件中使用
    ```rust
    use bitflags::bitflags;

    // The `bitflags!` macro generates `struct`s that manage a set of flags.
    bitflags! {
        #[derive(Debug, Clone, Copy, PartialEq, Eq, PartialOrd, Ord, Hash)]
        struct Flags: u32 {
            const A = 0b00000001;
            const B = 0b00000010;
            const C = 0b00000100;
            const ABC = Self::A.bits() | Self::B.bits() | Self::C.bits();
        }
    }

    fn main() {
        let e1 = Flags::A | Flags::C;
        let e2 = Flags::B | Flags::C;
        assert_eq!((e1 | e2), Flags::ABC);   // union
        assert_eq!((e1 & e2), Flags::C);     // intersection
        assert_eq!((e1 - e2), Flags::A);     // set difference
        assert_eq!(!e2, Flags::A);           // set complement
    }
    ```
通过上述样例可知，bitflags在源代码中的一般使用流程是：
    1. 先使用宏定义一个对应结构体类型，其中包含所需要的位符号。
    2. 再使用bitflags所定义好的特性（常用的方法）进行便捷的位集操作。

## 前期思考
在没有bitflags这个库之前，我们也可以使用原生Rust来实现位集。那么相比于用原生Rust实现位集，bitflags有什么好处吗？
我个人的理解是：  
- 使用位集的场合很广，重复度高，且内部逻辑独立。因此封装成crate可以解耦合，提高代码可读性。
- 位集的底层数据结构往往使用的是整数类型，但位集逻辑上是基于集合的。直接使用整数来实现位集比较绕（即位运算），位集提供了一个简单直观的使用方式（即集合操作）。比如，bitflags提供了is_empty, all, from_name等方法，这些方法在集合操作上逻辑是很平凡的，却是底层数据结构整数所不具有的。

## 文件树
bitflags的文件树结构如下，其中部分子文件夹的文件以...的形式省略。
```txt
.
├── benches
│   └── parse.rs
├── Cargo.toml
├── CHANGELOG.md
├── CODE_OF_CONDUCT.md
├── CONTRIBUTING.md
├── examples
│   └── ...
├── LICENSE-APACHE
├── LICENSE-MIT
├── README.md
├── SECURITY.md
├── src
│   ├── example_generated.rs
│   ├── external
│   │   ├── arbitrary.rs
│   │   ├── bytemuck.rs
│   │   └── serde.rs
│   ├── external.rs
│   ├── internal.rs
│   ├── iter.rs
│   ├── lib.rs
│   ├── parser.rs
│   ├── public.rs
│   ├── tests
│   │   └── ...
│   ├── tests.rs
│   └── traits.rs
└── tests
    └── ...

10 directories, 104 files
```

这个仓库的文件大概由这些部分组成：
- 若干项目声明：CODE_OF_CONDUCT.md, CONTRIBUTING.md, LICENSE-APACHE, LICENSE-MIT, SECURITY.md 
- 说明文档：README.md
- 改动历史：CHANGELOG.md
- 源代码：src子文件夹
- 样例代码：examples子文件夹
- 集成测试模块：test子文件夹
- 基准测试模块：benches子文件夹
___
- 在src子文件夹下和根目录下各存在一个tests子文件夹，这二者有何区别？  
    - 在根目录的tests子文件夹下的是集成测试代码，这是Rust指定的集成测试模块存放方式。
    - 在src子文件夹下的tests子文件夹的是单元测试代码，和tests.rs同时存在，这是crate里的file+dir形式的名为test的mod。在lib.rs中以单元测试模块的方式引用这整个mod。
        ```Rust
        #[cfg(test)]
        mod tests;
        ```
    - 单元测试、集成测试和基准测试的使用方法如下：
        ```zsh
        # 全部测试
        cargo test
        # 集成测试
        cargo test --test xxx
        # 单元测试
        cargo test xx
        # 基准测试
        cargo bench
        ```

## Cargo.toml
Cargo.toml是Rust项目的配置文件
- \[package\]: 项目的基本信息
- \[dependencies\]: 项目所依赖的外部库或crate
- \[dev-dependencies\]: 开发项目需要的依赖
- \[features\]: 定义某些特性，比如可以配合条件编译使用
- \[package.metadata.docs.rs\]: 元数据相关信息（不太懂）

## lib.rs
这个crate的src子文件夹下仅有lib.rs而没有main.rs，因此是一个Library crate。lib.rs即为本crate的“根”，即编译的入口，需要从它开始理解整个crate。

### 文档部分
使用了doc attr的语法糖格式的说明文字，用于生成说明文档。
1. 可见性  
   
    > 表现得和原生struct一致，如果在声明内部结构时使用pub就是外部可见的，否则就是私有的。
2. 表示方式  
   
    > bitflags!宏需要定义一个结构体Flags，它是一个新的类型，可以用\#\[repr()\]指定表示方式或按默认方式排布。
3. 可扩展性  
   
    > 可以在其上自定义附着函数
4. 介绍了bitflags实现了如下性质  
    - 迭代器
    - 格式化和输出形式
    - 运算符和常用方法
    - 可克隆性和可拷贝性
    - 默认值
5. 边界情况  
    - 零bit的flag
    - 多bit的flag
6. 功能划分
    - public: 生成面向用户的标志类型
    - internal: 生成面向`bitflags'的标志类型
    - external: 在这里有条件地实现外部库的特征
    这里讲的并不是很具体（反正我没有直接看懂），需要到每个文件下具体分析。

### 代码部分
遇到了一些陌生但不复杂的技术：
- 使用了条件编译cfg_attr
- 使用了宏规则定义macro_rules! bitflags

## iter.rs
1. 需求
    > 对于位集，经常有一些遍历若干已定义bit的需求，这就是迭代性质。  
    > 最朴素地，我们可以使用一个预定义的数组，其中包含了这些bit的index来实现。  
    > 如果更节约一些，我们可以使用一个在这些bit上全位1的整数U作为全集，用\(1 << i\) & U来完成。  
    > bitflags一方面使用后者的思路，仅用一个整数来表示全集，另一方面并非按位顺序而是按定义顺序来进行遍历，和前者类似。
2. 细节
    > 前面的讨论假定我们定义bit的符号时都是逐位的，这样不会出现重复。  
    > 但是bitflags也要兼容多bit符号的情况，比较特殊的是出现重复的情况。
    > 按定义顺序遍历时，如果发生覆盖现象会作如下处理：  
    > 完全覆盖：不生成对应迭代器  
    > 部分覆盖：都生成
3. 实现
    > 实现上bitflags把bit定义时的符号名暴露为迭代器。本质上来说，迭代器其实就是各个已定义的bit。  
    > 在检查覆盖时使用remaining属性记录序贯的位信息，应对重复bit情况。

## internal.rs
许多特性的私有实现。
主要使用了两个宏：\_\_declare\_internal\_bitflags和\_\_impl\_internal\_bitflags。
通过这两个宏以内部私有的形式提供了多种API，比如Default, Debug, Display, FromStr等

## public.rs
集成了许多对外暴露的API。  
使用的宏包括：__declare_public_bitflags、__impl_public_bitflags_forward、__impl_public_bitflags、__impl_public_bitflags_iter、__impl_public_bitflags_ops、__impl_public_bitflags_consts。  
定义了bitflags!宏和Bitflags结构，并在其上附着了各种方法和重载了运算符。  
通过该文件体现了bitflags的主要使用方法，比如：重载了Sub运算符，实现了is_empty方法等。

## external.rs
在bitflags库的external.rs文件中，定义了一些宏和模块，用于支持对外部库的特性实现。  
这些外部库的特性可以通过crate的feature标志进行开启或关闭。  
主要是对serde（大概关于序列化和反序列化）, arbitrary（随机化）, bytemuck（字节级操作）这三个外部库的特性实现（细节我并不）。  

## 上述三者
前面我提及了三种功能的划分但没有理解具体含义，这里我试图讲一讲我的理解：  
internal部分提供了一些私有的底层API，目的是为库内部提供服务。  
external部分主要是针对外部库的依赖作一些特性适配。  
public部分则是基于底层API实现了对用户暴露的上层API，目的是为使用库的外部用户提供服务。  

这样的抽象和模块化设计带来了更好的灵活性和封装性：  
- 对于外部用户，不变的上层API可以提供高的代码稳定性  
- 对于内部管理者，私有的底层API可以在保证上层不变的情况下修改或拓展一些特性  

## traits.rs
trait有点类似接口，目的是在不同类型间共享方法。  
同时，还可以直接使用一些预定义好的特性。  
对于这样一个逻辑简单的项目，trait在其中发挥了什么作用？究竟是出于项目完整性的考虑，还是确实有优势？我在trait这一块学的不是很明白，我的猜测是：一方面可以在缺省情况下利用现有的方法（继承）；另一方面可以在需要自定义的情况下实现拓展（多态）？

## parser.rs
提供了位标记和输出文本之间的相互转换，包括到十六进制形式、到输出文本的转换等。

## 阅读体验
bitflags的总体逻辑不难，但是在Rust语言上的实现对我来说还是比较复杂的。
主要是对以下这些部分不熟悉：

- 宏的定义
- 属性的使用
- trait的使用和目的

另一方面，我之前使用C/C++比较多，在思考方式上也需要有所调整。

___



# 近期生活照
**TODO**



# 出勤次数
凭印象，我本学期应该是完全缺勤一次，部分缺勤一次（第二节才到），都是因为睡过了。