# 4. CLASSES AND INTERFACES

## 13. Minimize the accessibility of classes and members

**Encapsulation**:

- A well designed module hides all of its implementation details.
- Separates its API from its implementation.
- Decouples modules that comprise a system, allowing them to be isolated while:
    - developed (can be developed in parallel)
    - tested (individual modules may prove succesful even if the system does not)
    - optimized and modified (no harm to other modules)
    - understood (dont need other modules to be understood)
    - used

**Make each class or member as inaccesible as possible**

If a package-private top level class is used by only one class make it a private nested class of the class that uses it. ([Item 22](#22-favor-static-member-classes-over-nonstatic))

It is acceptable to make a private member of a public class package-private in order to test it.

**Instance fields should never be public** ([Item 14](#14-in-public-classes-use-accessor-methods-not-public-fields)) Class will not be thread-safe.

Static fields can be public if contain primitive values or references to immutable objects. A final field containing a reference to a mutable object has all the disadvantages of a non final field.

Nonzero-length array is always mutable.

```java

	//Potential security hole!
	public static final Thing[] VALUES = {...}
```

Solution:

```java

	private static final Thing[] PRIVATE_VALUES ={...}
	public static final List<Thing> VALUES = Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```

Or:

```java

	private static final Thing[] PRIVATE_VALUES ={...}
	public static final Thing[] values(){
		return PRIVATE_VALUES.clone;
	}
```

## 14. In public classes, use accessor methods, not public fields

Degenerate classes should not be public

```java

	class Point {
		public double x;
		public double y;
	}
```

- They don't benefit from _encapsulation_ ([Item 13](#13-minimize-the-accessibility-of-classes-and-members))

- Can't change representation without changing the API.

- Can't enforce invariants.

- Can't take auxiliary actions when a field is accessed.

Replace them with _accessor methods_ (getters) and _mutators_ (setters).

```java

	class Point {
		private double x;
		private double y;

		public Point(double x, double y){
			this.x = x;
			this.y = y;
		}

		public double getX() { return x; }
		public double getY() { return y; }

		public void setX(double x) { this.x = x; }
		public void setY(double y) { this.y = y; }
	}
```

If a class is accessed **outside its package**, provide **accesor methods**.

If a class is **package-private or is a private nested class**, its **ok to expose** its data fields.

In **public classes** it is a questionable option to **expose immutable fields**.

## 15. Minimize Mutability

All the information of the instance is provided when it is created.
They are easier to design, implement and use. And they are less prone to errors and more secure

- Don't provide any methods that modify the object's state (no mutators)
- Ensure that the class can't be extended
- Make all fields final
- Make all fields private
- Ensure exclusive access to any mutable component

```java

	public final class Complex {
		private final double re;
		private final double im;

		public Complex (double re, double im) {
			this.re = re;
			this.im = im;
		}

		// Accessors with no corresponding mutators
		public double realPart() { return re;}
		public double imaginaryPart() { return im;}

		public Complex add(Complex c){
			return new Complex(re + c.re, im + c.im);
		}

		public Complex subtract(Complex c){
			return new Complex(re - c.re, im - c.im);
		}

		...

		@Override public boolean equals (Object o){...}
	}

```

The arithmetic operation **create and return a new instance**. (Functional approach)

Immutable objects are simple. They only have one state for their lifetime.

Immutable objects are thread-safe. Synchronization is not required. They can be shared freely and can reuse existing instances.

```java

	public static final Complex ZERO = new Complex(0,0)
	public static final Complex ONE = new Complex(1,0)
	public static final Complex I = new Complex(0,1)
```

Using static factories can create constants of frequently requested instances and serve them in future requests.

Internals of the immutable objects can also be shared.

They make great building blocks for other objects.

The disadvantage is that a separate object is required for distinct values. In some cases it could create a performance problem.

**How to deny subclassing in immutable objects**

1. Making it final

2. Make all of its constructors private or package-private and add a public static factory

```java

	public class Complex {
		private final double re;
		private final double im;

		private Complex (double re, double im){
			this.re = re;
			this.im = im;
		}

		public static Complex valueOf(double re, double im){
			return new Complex(re,im);
		}

		...
	}
```

This technique allows flexibility of multiple implementations, it's possible to tune the performance and permit to create more factories with names that clarify its function.

**Summary**

Classes should be immutable unless there are good reasons to make them mutable.

If a class can not be immutable, limit its mutability as much as possible.

Make every field final unless there is a good reason not to do it.

Some of the rules can be relaxed to improve performance (caching, lazy initialization...).

## 16. Favor composition over inheritance

Inheritance in this case is when a class extends another (_implementation inheritance_) Not interface inheritance.

**Inheritance violates encapsulation**

Fragility causes

1. A subclass depends on the implementation details of its superclass. If the superclass change the subclass may break.

2. The superclass can aquire new methods in new releases that might not be added in the subclass.

**Composition**

Instead of extending, give your new class a private field that references an instance of the existing class.

Each instance method in the new class (_forwarding class_)invokes the corresponding method (_forwarding methods_) on the contained instance of the existing class and returns the results.

**Wrapper (Decorator Pattern)**

```java

	// Wrapper class - uses composition in place of inheritance
	public class InstrumentedSet<E> extends ForwardingSet<E> {
		private int addCount = 0;
		//It extends a class(inheritance),but it is a forwarding class that is actually a compositon of the Set
		(specifically a forwarding class), not the Set itself.
		public InstrumentedSet (Set<E> s){
			super(s)
		}

		@Override
		public boolean add(E e){
			addCount++;
			return super.add(e);
		}

		@Override
		public boolean addAll (Collection< ? extends E> c){
			addCount += c.size();
			return super.addAll(c);
		}

		public int getAddCount() {
			return addCount;
		}
	}
```

```java

	// Reusable forwarding class
	public class ForwardingSet<E> implements Set<E> {
		private final Set<E> s; // Here is the composition. It uses the Set but not extends it.
		public ForwardingSet(Set<E> s) { this.s = s ; }

		// It implemets the Set, using the interface, and create the forwarding methods.
		public void clear() {s.clear();}
		public boolean contains(Object o) { return s.contains(o)}
		...
		public boolean add(E e) { return s.add(e)}
		public boolean addAll (Collection< ? extends E> c){return s.addAll(c)}
		...
	}

```

## 17. Design and document for inheritance or else prohibit it.

The class must document its _self-use_ of overridable methods.

Methods and constructors should document which _overridable_ methods or constructors (nonfinal, and public or protected ) invokes. The description begins with the phrase "This implementation."

To document a class so that it can be safely subclassed, you must describe implementations details.

To allow programmers to write efficient subclasses without undue pain, a class may have to provide hooks into its internal working in the form of judiciously chosen protected methods.

**Test the class for subclassing**. The only way to test a class designed for inheritance is to write subclasses.

Constructors must not invoke overridable methods. For _Serializable_ and _Cloneable_ implementations neither _clone_ nor _readObject_ may invoke overridable methods.

Prohibit subclassing in classes that are not designed and documented to be safely subclassed. 2 options:

- Declare the class final
- Make all constructors private or package-private and add public static factories in place of the constructors.
  ([Item 15](#15-minimize-mutability))

Consider using [Item 16](#16-favor-composition-over-inheritance) if what you want is to increase the functionality of your class instead of subclassing.

## 18. Prefer interfaces to abstract classes

Java permits only single Inheritance, this restriction on abstract classes severely contrains their use as type functions.

Inteface is generally the best way to define a type that permits multiple implementations.

Existing classes can be easily retro-fitted to implement a new interface.

Interfaces are ideal for defining mixins (a type that a class can implement in addition to its primary type to declare that it provides some optional bahaviour)

Interfaces allow the construction of nonhierarchical type frameworks.

Interfaces enable safe, powerful functionality enhancements (Wrapper class. [Item 16](#16-favor-composition-over-inheritance))

Combine the virtues of interfaces and abstract classes, by providing an abstract **skeletal implementation** class to go with each **nontrivial interface** that you export.

```java

	//Concrete implementation built atop skeletal implementation
	static List<Integer> intArrayAsList(final int[] a) {
		if (a == null) throw new NullPointerException();


		// From the documentation
		//This class provides a skeletal implementation of the List interface to minimize the effort required to implement this interface backed by a "random access" data store (such as an array)
		//To implement an unmodifiable list, the programmer needs only to extend this class and provide implementations for the get(int) and size() methods.
		//To implement a modifiable list, the programmer must additionally override the set(int, E)

		return new AbstractList<Integer>(){
			public Integer get (int i){
				return a[i]; // Autoboxing (Item 5)
			}

			@Override
			public Integer set(int i, Integer val){
				int oldVal = a[i];
				a[i] = val;		// Auto-unboxing
				return oldVal;	// Autoboxing
			}

			public int size(){
				return a.length;
			}
		}
	}
```

Skeletal implementations are designed for inheritance so follow [Item 17](#17-design-and-document-for-inheritance-or-else-prohibit-it) guidelines.

_simple implementation_ is like a skeletal implementation in that it implements the simplest possible working implementation.

Cons: It is far easier to evolve an abstract class than an interface. Once an interface is released and widely implemented, it is almost impossible to change.

## 19. Use interfaces only to define types

When a class implements an interface, the interface serves as a _type_ that can be used to refer to instances of the class.

Any other use, like the _constant interface_ should be avoided.

```java

	// Constant interface antipattern
	public interface PhysicalConstants {
		static final double AVOGRADOS_NUMBER = 6.02214199e23;
		static final double BOLTZAN_CONSTANT = 1.3806503e-23;
		static final double ELECTRON_MASS = 9.10938188e-31;
	}
```

Better use an enum type ([Item 31](#31-use-instance-fields-instead-of-ordinals)), or a noninstantiable _utility class_ ([Item 4](#4-enforce-noninstantiability-with-a-private-constructor))

```java

	//Constant utility class
	package com.effectivejava.science

	public class PhysicalConstants{
		private PhysicalConstants(){} // Prevents instantiation

		public static final double AVOGRADOS_NUMBER = 6.02214199e23;
		public static final double BOLTZAN_CONSTANT = 1.3806503e-23;
		public static final double ELECTRON_MASS = 9.10938188e-31;
	}
```

To avoid the need of qualifying use _static import_.

```java

	//Use of static import to avoid qualifying constants
	import static com.effectivejava.science.PhysicalConstants.*

	public class Test {
		double atoms(double mols){
			return AVOGRADOS_NUMBER * mols;
		}
		...
		// Many more uses of PhysicalConstants justify the static import
	}
```

## 20. Prefer class hierarchies to tagged classes

Tagged classes are verbose, error-prone and inefficient.

They have lot of boilerplate, bad readability, increase memory footprint, and more shortcommings.

```java

	// Tagged Class
	class Figure{
		enum Shaple {RECTANGLE, CIRCLE};

		final Shape shape;

		// Rectangle fields
		double length;
		double width;

		//Circle field
		double radius;

		// Circle Constructor
		Figure (double radius) {
			shape = Shape.CIRCLE;
			this.radius=radius;
		}

		// Rectangle Constructor
		Figure (double length, double width) {
			shape = Shape.RECTANGLE;
			this.length=length;
			this.width=width;
		}

		double area(){
			switch(shape){
				case RECTANGLE:
					return length*width;
				case CIRCLE
					return Math.PI * (radius * radius);
				defalult
					throw new AssertionError();
			}
		}
	}
```

A tagged class is just a palid imitation of a class hierarchy.

- The code is simple and clear.
- The specific implementations are in its own class
- All fields are final
- The compiler ensures that each class's constructor initializes its data fields.
- Extendability and flexibility (Square extends Rectangle)

```java

	abstract class Figure{
		abstract double area();
	}
	class Circle extends Figure{
		final double radius;

		Circle(double radius) { this.radius=radius;}

		double area(){return Math.PI * (radius * radius);}
	}
	class Rectangle extends Figure{
		final double length;
		final double width;

		Rectangle (double length, double width) {
			this.length=length;
			this.width=width;
		}

		double area(){return length*width;}
	}

	class Square extends Rectangle {
		Square(double side){
			super(side,side);
		}
	}
```

## 21. Use function objects to represent strategies

Strategies are facilities that allow programs to store and transmit the ability to invoke a particular function. Similar to _function pointers_, _delegates_ or _lambda expression_.

It is possible to define a object whose method perform operations on other objects.

**Concrete strategy**

```java

	class StringLengthComparator{
		public int compare(String s1, String s2){
			return s1.length() - s2.length();
		}
	}
```

Concrete strategies are typically _stateless_ threfore they should be singletons.

To be able to pass different strategies, clients should invoke methods from a _strategy interface_ instead of a concrete class.

**Comparator interface.** _Generic_([Item 26](#26-favor-generic-types))

```java

	public interface Comparator<T>{
		public int compare(T t1, T t2);
	}
```

```java

	class StringLengthComparator implements Comparator<String>{
		private StringLengthComparator(){} // Private constructor
		public static final StringLengthComparator INSTANCE = new StringLengthComparator(); // Singleton instance
		public int compare(String s1, String s2){
			return s1.length() - s2.length();
		}
	}
```

Using **anonymous classes**

```java

	Arrays.sort(stringArray, new Comparator<String>(){
		public int compare(String s1, String s2){
			return s1.length() - s2.length();
		}
	})
```

An anonymous class will create a new instance each time the call is executed. Consider a private static final field and reusing it.

Concrete strategy class don't need to be public, because the strategy interface serve as a type.
A host class can export the a public static field or factory, whose type is the interface and the concrete strategy class is a private nested class.

```java

	// Exporting a concrete strategy
	class Host{
		private static class StringLengthComparator implements Comparator<String>, Serializable {
			public int compare(String s1, String s2){
				return s1.length() - s2.length();
			}
		}

		//Returned comparator is serializable
		public static final Comparator<String> STRING_LEGTH_COMPARATOR = new StringLengthComparator();

		...
	}
```

## 22. Favor static member classes over nonstatic

4 types of nested classes.

1. static
2. nonstatic
3. anonymous
4. local

**Static**, a member class that does not require access to an enclosing instance must be _static_.

Storing references cost time, space and can cost not wanted behaviors of the garbage collector([Item 6](#6-eliminate-obsolete-object-references))

Common use of static member class is a public helper in conjuctions with its outer class. A nested class enum _Operation_ in _Calculator_ class. `Calculator.Operation.PLUS`;

**Nonstatic** member class instances are required to have an enclosing instance.

**Anonymous** classes are used to create _function objects_ on the fly. ([Item 21](#21-use-function-objects-to-represent-strategies))

**Local** class from the official docs: Use it if you need to create more than one instance of a class, access its constructor, or introduce a new, named type (because, for example, you need to invoke additional methods later).

Anonymous class, from the official docs: Use it if you need to declare fields or additional methods.

