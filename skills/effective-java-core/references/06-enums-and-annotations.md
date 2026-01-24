# 6. ENUMS AND ANNOTATIONS

## 30. Use enums instead of _int_ constants

Enums are classes that export one instance for each enumeration constant via a public static final field.
Clients can not create instances or extend them.
They are a generalization of singletons([Item 3](#3-enforce-the-singleton-property-with-a-private-constructor-or-an-enum-type))
They are compile-time type safe.

**Enums can have data associated**

```java

	public enum Planet{
		MERCURY(3.334e+23,2.234e6)
		VENUS(4.234e+23,6.636e6)
		EARTH(5.865e+23,6.256e6)
		...

		private final double mass;
		private final double radius;
		private final double surfaceGravity;

		private static final double G = 6.67300E-11;

		Planet(double mass, double radius){
			this.mass = mass;
			this.radius = radius;
			surfaceGravity = G * mass/(radius * radius);
		}

		public double mass() {return mass;}
		public double radius() {return radius;}
		public double surfaceGravity() {return surfaceGravity;}

		public double surfaceWeight(double mass){
			return mass * surfaceGravity;
		}
	}
```

Enums are immutable so their fields should be final([Item 15](#15-minimize-mutability))
Make fields private ([Item 14](#14-in-public-classes-use-accessor-methods-not-public-fields))

Enums should be a member class inside a top-level class if it is not generally used.

**Enum type with constant-specific method implementations**

```java

	public enum Operation{
		PLUS { double apply(double x, double y){return x + y;}},
		MINUS { double apply(double x, double y){return x - y;}},
		TIMES { double apply(double x, double y){return x * y;}},
		DIVIDE { double apply(double x, double y){return x / y;}};

 		// The abstract method force us not to forget to implement the method.
		abstract double apply(double x, double y);

	}
```

**Strategy enum pattern**
Use it, if multiple enum constants share common behaviors.

```java

	enum PayrollDay{
		MONDAY(PayType.WEEKDAY),
		TUESDAY(PayType.WEEKDAY),
		...
		SATURDAY(PayType.WEEKEND),
		SUNDAY(PayType.WEEKEND);

		private final PayType payType;

		PayrollDay(PayType payType) {this.payType = payType;}

		double pay(double hoursWorked, double payRate){
			return payType.pay(hoursWorked, payRate);
		}
		//The strategy  enum type
		private enum PayType{
			WEEKDAY{
				double overtimePay(double hours, double payRate) { return ...}
			};
			WEEKEND{
				double overtimePay(double hours, double payRate) { return ...}
			};
			private static final int HOURS_PER_SHIFT = 8;

			abstract double overtimePay(double hours, double payRate);

			double pay(double hoursWorked, double payRate){
				double basePay = hoursWorked * payRate;
				return basePay + overtimePay(hoursWorked, payRate);
			}
		}
	}
```

## 31. Use instance fields instead of ordinals

Never derive a value of an enum to its ordinal

```java

	public enum Ensemble{
		SOLO, DUET, TRIO...;
		public int numberOfMusicians() {return ordinal() + 1}
	}
```

Better approach

```java

	public enum Ensemble{
		SOLO(1), DUET(2), TRIO(3)...TRIPLE_QUARTET(12);
		private final int numberOfMusicians;
		Ensemble(int size) {this.numberOfMusicians = size;}
		public int numberOfMusicians() {return numberOfMusicians;}
	}
```

## 32. Use EnumSet instead of bit fields

If the elements of an enumarated are used primarily in sets, use EnumSet.

```java

	public class Text{
		public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

		//Any Set could be passed. Best EnumSet
		public void applyStyles(Set<Style> styles){ ... }
	}

	//Use
	text.applyStyles(EnumSet.of(Style.BOLD, Style. ITALIC));
```

It is a good practice to accept the interface `Set` instead of the implementation `	EnumSet`.

## 33. Use EnumMap instead of ordinal indexing

Use EnumMap to associate data with an enum

```java

	Map<Herb.Type, Set<Herb>> herbsByType = new EnumMap<Herb.Type, Set<Herb>>(Herb.Type.class);

	for (Herb.Type t : Herb.Type.values())
		herbsByType.put(t, new HashSet<Herb>())

	for (Herb h : garden)
		herbsByType.get(h.type).add(h);

	System.out.println(herbsByType);

```

In case you need a multidimensional relationship use `EnumMap<..., EnumMap<...>>`

## 34. Emulate extensible enums with interfaces

Enums types can not extend another enum types.

_Opcodes_ as a use case of enums extensibility.

```java

	public interface Operation{
		double apply(double x, double y);
	}
	public enum BasicOperation implements Operation{
		PLUS("+"){
			public double apply(double x, double y) {return x + y}
		},
		MINUS("-"){...},TIMES("*"){...},DIVIDE("/"){...};

		private final String symbol;
		BasicOperation(String symbol){
			this.symbol = symbol;
		}
		@Override
		public String toString(){ return symbol; }
	}
```

_BasicOperation_ is not extensible, but the interface type _Operation_ is, and it is the one used to represent operations in APIs.

**Emulated extension type**

```java

	public enum ExtendedOperation implements Operation{
		EXP("^"){
			public double apply(double x, double y) {return Math.pow(x,y)}
		}
		REMAINDER("%"){
			public double apply(double x, double y) {return x % y}
		}

		private final String symbol;
		ExtendedOperation(String symbol){
			this.symbol = symbol;
		}
		@Override
		public String toString(){ return symbol; }
	}
```

## 35. Prefer annotations to naming patterns

Sample of the _@Test_ annotation

**Marker**

```java

	//Marker annotation type declaration
	import java.lang.annotation.*;

	//Indicates that the annotated method is a test method.
	//Use only on parameterless static methods.
	@Retention(RetentionPolicy.RUNTIME)
	@Target(ElementType.METHOD)
	public @interface Test {
	}
```

`@Retention` and `@Target` are _meta-annotations_

`@Retention(RetentionPolicy.RUNTIME)`: indicates that the Test annotation should be retained at runtime.(It makes them visible to the test tool)

`@Target(ElementType.METHOD)`indicates that is legal only on method declarations. Not in class, fields or other programs declarations

**Retention RetentionPolicies**

| Enum    | Description                              |
| :------ | :--------------------------------------- |
| CLASS   | Retain only at compile time, not runtime |
| RUNTIME | Retain at compile and also runtime       |
| SOURCE  | Discard by the compiler                  |

**Target ElementTypes**

| Enum            | Valid on...                                        |
| :-------------- | :------------------------------------------------- |
| ANNOTATION_TYPE | Annotation type declaration                        |
| CONSTRUCTOR     | constructors                                       |
| FIELD           | the field (includes also enum constants)           |
| LOCAL_VARIABLE  | local variables                                    |
| METHOD          | methods                                            |
| PACKAGE         | packages                                           |
| PARAMETER       | parameter declaration                              |
| TYPE            | class, interface, annotation and enums declaration |
| TYPE_PARAMETER  | type parameter declarations                        |
| TYPE_USE        | the use of a specific type                         |

**Use**

```java

	public class Sample{
		@Test public static void m1(){}		// Test pass
		public static void m2(){} 			// Not test applied
		@Test public static void m3(){ 		// Test fail
			throw new RuntimeException("Boom")
		}
		@Test public void m4(){}			// Invalid use. Non static method.
	}
```

**Process annotations**

```java

	import java.lang.reflect.*

	public class RunTests{
		public static void main(String[] args) throws Exception {
			int tests = 0;
			int passed  = 0;
			Class testClass = Class.forName(args[0]);
			for (Method m : testClass.getDeclaredMethods()){
				if (m.isAnnotationPresent(Test.class)){
					tests++;
					try{
						m.invoke(null);
						passed++;
					} catch(InvocationTargetException wrappedExc){
						Throwable exc = wrappedExc.getCause();
						System.out.println(m + " failed: " + exc);
					} catch (Exception exc){
						System.out.println("INVALID @Test: " + m);
					}
				}
			}
			System.out.printf("Passed: %d, Failed: %d%n", passed, tests - passed);
		}
	}
```

## 36. Consistently use the _Override_ annotation

Use the _Override_ annotation on every method declaration that you believe to override a super class declaration.

```java

	public class Bigram {
		private final class first;
		private final class second;
		public Bigram(char first, char second){
			this.first = first;
			this.second = second;
		}
		public boolean equals(Bigram b){ //ERROR.
			return b.first == first && b.second == second;
		}
		...
	}
```

We are overloading equals instead of overriding it.  
The correct sign to override the super method is:

```java

	public boolean equals(Object o){}
```

With the use of Override the compiler would alert us about our mistake.

## 37. Use marker interfaces to define types

Marker interface in Java is interfaces with no field or methods or in simple word empty interface in java is called marker interface.

A _marker interface_ is an interface that contains no method declarations, but "marks" a class that implements the interface as having some property.

When your class implements `java.io.Serializable` interface it becomes Serializable in Java and gives compiler an indication that use Java Serialization mechanism to serialize this object.

- Marker interfaces define a type that is implemented by instances of the marked class; marker annotations do not. (Catch errors in compile time).
- They can be targeted more precisely than marker annotations.
- It's possible to add more information to an annotation type after it is already in use.

