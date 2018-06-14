## 当遇到许多构造方法参数时，考虑构建器

静态工厂和构造函数有一个共同的限制:它们不能很好地扩展到大量可选参数。考虑一个代表包装食品上出现的营养成分标签的类的例子。这些标签有一些需要的领域，包括分量，每个容器的分量，以及每份的热量，以及超过20个可选的领域——总脂肪，饱和脂肪，反式脂肪，胆固醇，钠，等等。大多数产品只有少数几个可选字段的值为非零值。

对于这样一个类，你应该怎样去写它的构造方法或者静态工厂方法呢？传统上，程序员使用伸缩构造函数模式，在这种模式中，您只提供了一个只有必需参数的构造函数，另一个带有一个可选参数，第三个带有两个可选参数，等等，最后在带有所有可选参数的构造函数中。下面是它在实践中的模样。为了方便起见，只显示4个可选字段：

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

通常，这个构造函数调用需要许多您不想设置的参数，但是您必须为它们传递一个值。在本例中，我们传递了一个值为0的fat。“只有”6个参数可能看起来不那么糟糕，但是随着参数数量的增加，它很快就会失去控制。

**简而言之，可伸缩的构造函数模式是有效的，但是当有许多参数时，客户端的代码很难写，而且可读性更差**。读者不知道这些值是什么意思，必须仔细地计算参数的个数来找出答案。长序列的相同类型的参数可以导致细微的错误。如果客户端不小心将两个这样的参数位置颠倒，编译器是不会报错的，但是程序在运行时将会出错\(条目51\)。

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

这种模式没有伸缩构造函数模式的缺点。如果有点冗长，创建实例很容易，并且容易读取结果代码:

```java
NutritionFacts cocaCola = new NutritionFacts();
cocaCola.setServingSize(240);
cocaCola.setServings(8);
cocaCola.setCalories(100);
cocaCola.setSodium(35);
cocaCola.setCarbohydrate(27);
```

不幸的是，**JavaBeans**模式本身有严重的缺点。由于构造方法在多个调用中被拆分，所以**JavaBean** 可能在其构建过程中处于不一致的状态。仅仅通过检查构造函数参数的有效性，该类没法实现一致性。在不一致的状态下尝试使用对象可能会导致与包含bug的代码相去甚远的错误，因此很难进行调试。与此相关的一个缺点是，**JavaBeans**模式排除了使类不可变的可能性\(见第17条\)，并要求程序员为确保线程安全而增加工作。

当对象的构造完成时，我们可以手动冻结这个对象，来减少这些缺点。并且直到下次被冻结才能使用。但是这种变形是笨拙，在实践中很少使用。而且，它会在运行时造成错误，因为编译器不能确保程序员使用对象之前，调用的是冻结的方法。

幸运的是，还有第三种选择，它将伸缩构造函数模式的安全性与**JavaBeans**模式的可读性相结合。它就是构建器模式的形式。客户端调用一个构造方法（或是静态工厂），并附上它需要的参数来获得一个构建器对象，来代替直接创造所需的目标对象。然后客户端调用构建起对象上的类似**setter**的方法去设置每一个感兴趣的可选的参数。最后，客户端调用无参的**build**方法去生成目标对象，通常它是不可变的。一般来说，这个构建器类是它构建的类的静态成员类。在实践中它通常看起来就是下面的样子：

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

**NutritionFacts**类是不可变的，所有参数默认值都在一个位置。构建器的**setter** 方法返回构建器本身，以便可以链接调用，从而生成一个连贯的API。客户端代码是这样的:

```java
NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8)
.calories(100).sodium(35).carbohydrate(27).build();
```

该客户端代码易于编写，更重要的是易于阅读。构建器模式模拟了在**Python**和**Scala**中找到的命名可选参数。

为简便起见，省略了有效性检查。要检查构建器的构造函数和方法中的参数有效性，为了尽快检查出无效参数。检查**build**方法调用的构造函数中涉及多个参数的不变量。要确保这些不变量没被篡改，请在复制构造器参数\(第50项\)之后对对象字段进行检查。如果检查失败，会抛出一个**IllegalArgumentException**\(第72项\)，它的详细消息会指示出哪些参数无效\(第75项\)。

构建器模式非常适合类的层次结构。使用构建器的并行层次结构，每个构建器嵌套在相应的类中。抽象类有抽象的构建器;具体类有具体的构建器。例如，将一个抽象类当做代表了不同种类披萨的层次结构的根类:

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

请注意,**Pizza.Builder**是具有递归类型参数的泛型类型\(条目30\)。它与抽象的self方法一起使用，允许方法链的形式在子类中正常工作，而不需要强制类型转换。在面对Java缺少自我类型这一事实下，像这种仿造自我类型的工作方法是常用的。

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

