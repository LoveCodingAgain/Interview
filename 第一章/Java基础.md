## 1.1
>> Java语言特性:
  面向对象(封装、继承、多态)
  平台无关性(基于JVM运行编译.class文件,解释执行)
  语言(泛型、Lambda、生产环境使用最多的JDK8)
  丰富类库(集合、并发、网络、IO/NIO)
  JRE(Java运行环境、JVM、类库)
  JDK(Java开发工具、包括JRE、javac、javap,包括诊断工具jstat、jstack、jmap、jps、jinfo)
## 1.2
  8种基本类型如下:
  
  |  基本类型   | 类型  |  所占字节 |
  |  ----  | ----  | ---- |
  | byte   | 数值型 | 1字节 |
  | short  | 数值型 | 2字节 |
  | int  | 数值型 | 4字节 |
  | long | 数值型 | 8字节 |
  | float | 数值型 | 4字节 |
  | double | 数值型 | 8字节 |
  | boolean| 布尔型 | 1bit |
  | char| 字符型 | 2字节 |
  其对应包装类型如下:

|  基本类型   | 包装类型  |
|  ----  | ----  |
| byte   | Byte |
| short  | Short |
| int  | Integer|
| long | Long |
| float | float |
| double | Double|
| boolean| Boolean|
| char| Character |

 装箱:将基本类型转换成包装类的过程.拆箱:将包装类转换成基本类型的过程.JDK5提供的自动拆装箱
  
 区别:
 基本类型是直接用来存储值的,放在栈中方便快速使用,包装类型是类,其实例是对象,放在堆中.
 
 基本类型不是对象,因此没有方法,包装类型是类,因此有方法.
 
 基本类型直接赋值即可,包装类型需要使用new关键字创建.
 
 包装类型初始值为null,基本类型初始值不为null,是根据类型而定的.
 
 
  