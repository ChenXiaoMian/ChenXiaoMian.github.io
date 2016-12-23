---
title: PHP类包含的七种语法说明
date: 2016-2-13 23:26:39
tags: PHP
---
一个完整的PHP类包含的七种语法说明,这些语法包括属性、静态属性、方法、静态方法、类常量、构造函数、析构函数。

## 类中的七种语法说明

1. 属性
2. 静态属性
3. 方法
4. 静态方法
5. 类常量
6. 构造函数
7. 析构函数

``` bash
    <?php
      class Student {
        // 类里的属性、方法和函数的访问权限有 （函数和方法是同一个概念）
        // private 私有的 protected 受保护的 public 公有的
        // 类常量 没有访问权限修饰符
        const STUDENT = 'Tom';
        // 属性
        public $stu_name;
        // 静态属性
        public static $stu_num = 1;
        // 方法
        public function stuFunction() {
          echo 'non_static_function','<br />';
        }
        // 静态方法
        public static function static_stuFunction() {
          echo 'static_function','<br />';
        }
        // 构造函数 创建对象时自动调用
        public function __construct($stu_name) {
          $this->stu_name = $stu_name;
          echo '__construct','<br />';

        }
        // 析构函数 销毁对象时自动调用
        public function __destruct() {
          echo '__destruct','<br />';
        }
      }

      // 实例化类对象
      $object = new Student('Tom');
      // 对象调用属性
      echo $object->stu_name,'<br />';
      // 对象调用静态属性
      echo $object::$stu_num,'<br />';
      // 类调用静态属性
      echo Student::$stu_num,'<br />';
      // 使用对象分别调用方法和静态方法
      $object->stuFunction();
      $object->static_stuFunction();
      $object::stuFunction();
      $object::static_stuFunction();
      // 使用类分别调用方法和静态方法
      Student::stuFunction();
      Student::static_stuFunction();
      // 类调用类常量
      echo Student::STUDENT,'<br />';
```

总结：
对象可以调用属性和静态属性，类只能调用静态属性。
对象可以调用方法和静态方法，类可以调用方法和静态方法。
