## 当遇到许多构造方法参数时，考虑构建器

静态工厂和构造函数有一个共同的限制:它们不能很好地扩展到大量可选参数。考虑一个代表包装食品上出现的营养成分标签的类的例子。这些标签有一些必要的字段，如分量大小、每瓶容量以及每份的卡路里里数，以及超过20个可选的字段——总脂肪，饱和脂肪，反式脂肪，胆固醇，钠，等等。大多数产品只有少数几个可选字段的值为非零值。

对于这样一个类来说，你应该编写哪种构造方法或是静态工厂呢？传统上，程序员们会使用重叠构造方法模式，在这种模式中，您只提供了一个只有必需参数的构造函数，然后编写一个接收单个可选参数的构造方法，再编写一个接收两个可选参数的构造方法，以此类推，最后提供一个接收所有可选参数的构造方法。如下代码示例例就说明了了这一点。出于简洁的目的，这里只给出了4个可选字段：

```java
// Telescoping constructor pattern - does not scale well!
public class NutritionFacts {
    private final int servingSize; // (mL) required
    private final int servings; // (per container) required
    private final int calories; // (per serving) optional
    private final int fat; // (g/serving) optional
    private final int sodium; // (mg/serving) optional
    private final int carbohydrate; // (g/serving) optional

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }
    public NutritionFacts(int servingSize, int servings,
        int calories) {
        this(servingSize, servings, calories, 0);
    }
    public NutritionFacts(int servingSize, int servings,
        int calories, int fat) {
        this(servingSize, servings, calories, fat, 0);
    }
    public NutritionFacts(int servingSize, int servings,
    int calories, int fat, int sodium) {
    this(servingSize, servings, calories, fat, sodium, 0);
    }                                                            
    public NutritionFacts(int servingSize, int servings,
        int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

当您想要创建一个实例时，您可以使用包含您想要设置的所有参数的最短参数列表的构造函数。

```java
NutritionFacts cocaCola = new NutritionFacts(240, 8, 100, 0, 35, 27);
```

通常，这个构造函数调用需要许多您不想设置的参数，但是您必须为它们传递一个值。在本例中，我们传递了一个值为0的fat。“只有”6个参数可能看起来不那么糟糕，但是随着参数数量的增加，很快你就数不过来了。

**简而言之，构造函数模式是有效的，但是当有许多参数时，客户端的代码很难写，而且可读性更差**。读者不知道这些值是什么意思，必须仔细地计算参数的个数来找出答案。长长的同类型参数序列会导致非常隐秘的Bug。。如果客户端不小心将两个这样的参数位置颠倒，编译器是不会报错的，但是程序在运行时将会出错\(条目51\)。

当你遇到一个构造函数中有许多可选参数时，第二个替代方法是**JavaBeans**模式，在这个模式中，你调用一个无参数的构造函数来创建对象，然后调用**setter**方法来设置每个必需的参数和每个可选的参数:

```java
// JavaBeans Pattern - allows inconsistency, mandates mutability
public class NutritionFacts {
    // Parameters initialized to default values (if any)
    private int servingSize = -1; // Required; no default value
    private int servings = -1; // Required; no default value
    private int calories = 0;
    private int fat = 0;
    private int sodium = 0;
    private int carbohydrate = 0;
    public NutritionFacts() { }
    // Setters
    public void setServingSize(int val) { servingSize = val; }
    public void setServings(int val) { servings = val; }
    public void setCalories(int val) { calories = val; }
    public void setFat(int val) { fat = val; }
    public void setSodium(int val) { sodium = val; }
    public void setCarbohydrate(int val) { carbohydrate = val; }
}
```

这种模式没有重叠构造函数模式的缺点。通过这种方式可以轻松创建实例例（就是稍微有点冗长），并且代码读起来也比较容易：

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

不幸的是，JavaBeans模式本身有严重的缺点。由于构造方法在多个调用中被拆分，所以JavaBean可能在其构建过程中处于不一致的状态。仅仅通过检查构造函数参数的有效性，该类没法实现一致性。在不一致的状态下尝试使用对象可能会导致与包含bug的代码相去甚远的错误，因此很难进行调试。与此相关的一个缺点是，JavaBeans模式排除了使类不可变的可能性\(见第17条\)，并要求程序员为确保线程安全而增加工作。

当构造完毕时，我们可以通过手工『冻结』对象并且直到冻结后才允许使用对象来消除这些缺陷，不过这种做法很少使用。此外，这么做会导致运行期错误，因为编译器无法确保程序员在使用对象前会调用对象的冻结方法。

幸运的是，还有第三种选择，它将伸缩构造函数模式的安全性与JavaBeans模式的可读性相结合。它就是构建器模式的形式。客户端调用一个构造方法（或是静态工厂），并附上它需要的参数来获得一个构建器对象，来代替直接创造所需的目标对象。然后客户端调用构建起对象上的类似setter的方法去设置每一个感兴趣的可选的参数。最后，客户端调用无参的build方法去生成目标对象，通常它是不可变的。一般来说，这个构建器类是它构建的类的静态成员类。在实践中它通常看起来就是下面的样子：

```java
// Builder Pattern
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // Required parameters
        private final int servingSize;
        private final int servings;
        // Optional parameters - initialized to default values
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }
        public Builder calories(int val)
        { calories = val; return this; }

        public Builder fat(int val)
        { fat = val; return this; }

        public Builder sodium(int val)
        { sodium = val; return this; }

        public Builder carbohydrate(int val)
        { carbohydrate = val; return this; }

        public NutritionFacts build() {
            return new NutritionFacts(this);
        }    
    }
    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

NutritionFacts类是不可变的，所有参数默认值都在一个位置。构建器的setter 方法返回构建器本身，这样调用就可以链接起来，形成一种流式API 。客户端代码是这样的:

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8) .calories(100).sodium(35).carbohydrate(27).build();
```

该客户端代码易于编写，更重要的是易于阅读。构建器模式模拟了在Python和Scala中找到的命名可选参数。

为简便起见，省略了有效性检查。要检查构建器的构造函数和方法中的参数有效性，为了尽快检查出无效参数。检查build方法调用的构造函数中涉及多个参数的不变量。要确保这些不变量没被篡改，请在复制构造器参数\(第50项\)之后对对象字段进行检查。如果检查失败，会抛出一个IllegalArgumentException（第72项），它的详细消息会指示出哪些参数无效\(第75项\)。

构建器模式非常适合类的层次结构。使用并行的构建器层次结构，每个构建器嵌套在相应的类中。抽象类有抽象的构建器;具体类有具体的构建器。例如，将一个抽象类当做代表了不同种类披萨的层次结构的根类:

```java
// Builder pattern for class hierarchies
public abstract class Pizza {
    public enum Topping { HAM, MUSHROOM, ONION, PEPPER, SAUSAGE }
    final Set<Topping> toppings;

    abstract static class Builder<T extends Builder<T>> {
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);

        public T addTopping(Topping topping) {
            toppings.add(Objects.requireNonNull(topping));
            return self();
        }

        abstract Pizza build();
        // Subclasses must override this method to return "this"
        protected abstract T self();
    }

    Pizza(Builder<?> builder) {
        toppings = builder.toppings.clone(); // See Item 50
    }
}
```

注意到Pizza.Builder是个泛型类型，它有一个递归的类型参数（条款30）。通过该参数以及抽象的self方法可以让方法在子类中恰当地链接起来，而无需进行类型转换。这种对于Java缺乏自我类型问题的解决方案叫做模拟的自我类型。

这里有两个具体的披萨子类，一个是标准的纽约风格披萨，另一个是奶酪馅饼式披萨。前者需要一个尺寸大小参数，而后者可以让你指定酱汁是在里面还是在外面。

```java
public class NyPizza extends Pizza {
    public enum Size { SMALL, MEDIUM, LARGE }
    private final Size size;

    public static class Builder extends Pizza.Builder<Builder> {
        private final Size size;
        
        public Builder(Size size) {
            this.size = Objects.requireNonNull(size);
        }

        @Override 
        public NyPizza build() {
            return new NyPizza(this);
        }

        @Override 
        protected Builder self() { return this; }
    }

    private NyPizza(Builder builder) {
        super(builder);
        size = builder.size;
    }
}

public class Calzone extends Pizza {
    private final boolean sauceInside;

    public static class Builder extends Pizza.Builder<Builder> {
        private boolean sauceInside = false; // Default
        
        public Builder sauceInside() {
            sauceInside = true;
            return this;
        }

        @Override
        public Calzone build() {
            return new Calzone(this);
        }

        @Override
        protected Builder self() { return this; }
    }

    private Calzone(Builder builder) {
        super(builder);
        sauceInside = builder.sauceInside;
    }
}
```

注意每个子类的builder的build\(\)方法都被声明成返回一个具类： NyPizza.Builder的build方法返回NyPizza类，而Calzone.Builder的build方法返回Calzone类，这种子类方法返回父类中声明的返回类型的子类型的技术，称为返回类型的协变。它允许客户端使用这些构建器，而不需要强制转换。

这些“层次化的构建器“的客户端代码本质上等价于简单的NutritionFacts的构建器的代码。如下面展示的客户端例子代码，假如要在枚举常量集合上发生静态导入也是很简洁的。

```java
NyPizza pizza = new NyPizza.Builder(SMALL).addTopping(SAUSAGE).addTopping(ONION).build();
Calzone calzone = new Calzone.Builder().addTopping(HAM).sauceInside().build();
```

与构造函数相比，构建器的第二个优点是，构建器可以有多个可变的参数，因为每个参数都在自己的方法中指定。

或者，构建器可以把多次调用所需要的参数聚合到某一个方法里的一个单一字段上，正如前面 addTopping方法所展示的那样。

构建器模式非常灵活。一个构建器可以重复使用建立多个对象。构建器的参数可以在调用构建方法时进行调整，以改变创建的对象。构建器可以在对象创建时自动填充一些字段，例如在创建对象时增加的序列号。

构建器模式也有缺点。为了能创建一个对象，你必须先创建它的构建器。虽然创建这个构建器的成本在实践中不太可能被注意到，但是在对性能要求很高的场景下这可能是个问题。此外，构建器模式比可伸缩构造器模式更加冗长，因此，只有当有足够多的参数时，使用它才有价值，比如四个或更多的参数时。但是，如果您从构造函数或静态工厂开始，并在类发展到参数数量失控时切换到构建函数，那么过时的构造函数或静态工厂将会像令人痛心的拇指一样突出。总之，在设计类时，构建器模式是一个很好的选择，该类的构造函数或静态工厂将具有多个参数，特别是如果许多参数是可选的或类型相同的。 与伸缩性的构造函数相比，使用构建器模式的客户端代码更容易读写，而且构建器比JavaBeans要安全得多。 