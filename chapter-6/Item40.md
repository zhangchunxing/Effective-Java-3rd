# 始终如一地使用Override注解

Java库包含几种注释类型。对于典型的程序员来说，其中最重要的是`@Override`。这个注解只能用于方法声明，它表示被注解的方法声明覆盖了超类中的声明。如果你始终使用这个注解，它将保护你免受大量恶意bug的攻击。考虑下面程序，其中类`Bigram`表示一个双字母组，或有序的字母对：

```java
// Can you spot the bug?
public class Bigram {
    private final char first;
    private final char second;
    
    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }
    
    public boolean equals(Bigram b) {
    	return b.first == first && b.second == second;
    }
    
    public int hashCode() {
    	return 31 * first + second;
    }
    
    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
        	for (char ch = 'a'; ch <= 'z'; ch++)
        		s.add(new Bigram(ch, ch));
        System.out.println(s.size());
    }
}
```

