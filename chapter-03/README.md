# 对所有对象通用的方法
虽然Object是一个具体的类，但它主要是为扩展而设计的。它的所有非终态方法(equals、hashCode、toString、clone和finalize)都有显式的通用约定，因为设计它们的目的就是为了重载（**overridden**）。任意一个重载了上面这些方法的类都有责任去遵循这些方法的通用约定；不遵守约定会妨碍依赖这些约定的类（如`HashMap`和`HashSet`）和其它类正常运行。

本章将告诉你何时以及如何覆盖非终态的Object方法。 本章省略了finalize方法，因为它在条款8中讨论过。而`Comparable.compareTo`虽然不是Object方法，但是本章也对它进行讨论，因为它具有类似的特征。
