# 5. GENERICS

## 23. Don't use raw types in new code

Generic classes and interfaces are the ones who have one or more _type parameter_ as _generic_, i.e. `List<E>`

Each generic type defines a set of _parametrized types_ `List<String>`

_Raw types_ is the generic type definition without type parameters. `List`

```java

	private final List stamps = ...

	stamps.add(new Coin(...)); //Erroneous insertion. Does not throw any error

	Stamp s = (Stamp) stamps.get(i); // Throws ClassCastException when getting the Coin
```

```java

	private final Collection<Stamp> stamps = ...

	stamps.add(new Coin()); // Compile time error. Coin can not be add to Collection<Stamp>

	Stamp s = stamps.get(i); // No need casting
```

Use of raw types lose safety and expressiveness of generics.

Type safety is kept in a parametrized type like `List<Object>` but not in raw types (`List`).

There are subtyping rules for generics. For example `List<String>` is a subtype of `List` but not of `List<Object>` ([Item 25](#25-prefer-lists-to-arrays))

**Unbounded Wildcard Types `Set<?>`**
Used when a generic type is needed but we don't know or care about the actual type.

Never add elements (other than null) into a `Collection<?>`

2 exceptions (because generic type information is erased at runtime):

- Use raw types in class literals `List.class`,`String[].class` are legal, `List<String>.class`, `List<?>.class` are not.
- Use of instanceof

```java

	if (o instanceof Set){
		Set<?> = (Set<?>) o;
	}
```

| **Term**                | **Example**                        | **Item** |
| ----------------------- | ---------------------------------- | -------- |
| Parametrized type       | `List<String>`                     | 23       |
| Actual type parameter   | `String`                           | 23       |
| Generic type            | `List<E>`                          | 23, 26   |
| Formal type parameter   | `E`                                | 23       |
| Unbounded wildcard type | `List<?>`                          | 23       |
| Raw type                | `List`                             | 23       |
| Bounded type parameter  | `<E extends Number>`               | 26       |
| Recursive type bound    | `<T extends Comparable<T>>`        | 27       |
| Bounded wildcard type   | `List<? extends Number>`           | 28       |
| Generic method          | `static <E> List<E> asList(E[] a)` | 27       |
| Type token              | `String.class`                     | 29       |

## 24. Eliminate unchecked warnings

Eliminate every unchecked warning that you can, if you can´t use _Suppress-Warnings_ annotation on the smallest scope possible.

```java

	Set<Lark> exaltation = new HashSet(); Warning, unchecked conversion found.
	Set<Lark> exaltation = new HashSet<Lark>(); Good
```

## 25. Prefer lists to arrays

Arrays are _covariant_: if `Sub` is a subtype of `Super`, `Sub[]` is a subtype of `Super[]`  
Generics are _invariant_: for any two types `Type1` and `Type2`, `List<Type1>` in neither sub or super type of `List<Type2>`

```java

	// Fails at runtime
	Object[] objectArray = new Long[1];
	objectArray[0] ="I don't fit in" // Throws ArrayStoreException

	// Won't compile
	List<Object> ol = new ArrayList<Long>();//Incompatible types
	ol.add("I don't fit in")
```

Arrays are _reified_: Arrays know and enforce their element types at runtime.
Generics are _erasure_: Enforce their type constrains only at compile time and discard (or _erase_) their element type information at runtime.

Therefore it is illegal to create an array of a generic type, a parameterized type or a type parameter.

`new List<E>[]`, `new List<String>[]`, `new E[]` will result in _generic array creation_ errors.

## 26. Favor generic types

Making [Item 6](#6-eliminate-obsolete-object-references) to use generics.

```java

	public class Stack{
		private E[] elements;
		private int size = 0;
		private static final int DEFAULT_INITIAL_CAPACITY = 16;

		public Stack(){
			elements = new E [DEFAULT_INITIAL_CAPACITY];//Error: Can't create an array of a non-reifiable type.
		}

		public void push(E e){
			ensureCapacity();
			elements[size++] = e;
		}

		public E pop(){
			if (size == 0)
				throw new EmptyStackException();
			E result = elements[--size];
			elements[size] = null;
			return result;
		}
		...
	}
```

There will be one error:

```java

	//Error: Generic array creation. Can't create an array of a non-reifiable type.
	elements = new E [DEFAULT_INITIAL_CAPACITY];
```

**First option** (more commonly used.)

```java

	// Warning: Compiler can not prove the type safe, but we can.
	// This elements array will contain only E instances from push(E).
	// This is sufficient to ensure type safety, but the runtime
	// type of the array won't be E[]; it will always be Object[]!
	@SupressWarnings("unchecked")
	public Stack(){
		elements = (E[]) new Object [DEFAULT_INITIAL_CAPACITY];
	}
```

**Second Option**

```java

	...
	private Object[] elements;
	...
	result = elements[--size] // Error: found Object, required E
```

A cast will generate a warning. Beacuse E is a non-reifiable type, there is no way the compiler can check the cast at runtime.

```java

	result = (E) elements[--size]
```

The appropriate suppression of the unchecked warning

```java

	public E pop(){
		if (size == 0)
			throw new EmptyStackException();

		// push requires elements to be of type E, so cast is correct.
		@SupressWarnings("unchecked") E result = elements[--size];

		elements[size] = null;
		return result;
	}
```

## 27. Favor generic Methods

Generic Method

```java

	//
	public static <E> Set<E> union(Set<E> s1, Set<E> s2){
		Set<E> result = new HashSet<E>(s1);
		result.addAll(s2);
		return result;
	}
```

**Type inference**: Compiler knows because of `Set<String>` that `E` is a _String_

In generic constructors the type parameters have to be on both sides of the declaration. (Java 1.7 might have fix it)

```java

	Map<String,List<String>> anagrams = new HashMap<String, List<String>>();
```

To avoid ic create a _generic static factory method_

```java

	public static <K,V> HashMap<K,V> newHashMap(){
		return new HashMap<K,V>();
	}

	//Use
	Map<String,List<String>> anagrams = newHashMap();


```

**Generic Singleton Pattern** Create an object that is immutable but applicable to many different types.

```java

	public interface UnaryFunction<T>{
		T apply(T arg);
	}

	// Generic singleton factory pattern
	private static UnaryFunction<Object> IDENTITY_FUNCTION = new UnaryFunction<Object>(){
		public Object apply(Object arg) {return arg;}
	};

	//IDENTITY_FUNCTION is stateless and its type parameter is unbounded so it's safe to share one instance across all types.
	@SuppressWarnings("unchecked")
	public static<T> UnaryFunction<T> identityFunction(){
		return(UnaryFunction<T>) IDENTITY_FUNCTION;
	}
```

**Recursive Type Bound** : When the type paremeter is bounded by some expression involving that type parameter itself.

```java

	public static<T extends Comparable<T>> T max (List<T> list){
		Iterator <T> i = list.iterator():
		T result = i.next();
		while (i.hasNext()) {
			T t = i.next();
			if(t.compareTo(result) > 0)
				result = t;
		}
		return result;
	}
```

## 28. Use bounded wildcards to increase API flexibility

Parameterized types are invariant.([Item 25](#25-prefer-lists-to-arrays)) Ie `List<String>` is not a subtype of `List<Object>`

```java

	public void pushAll(Iterable<E> src){
		for(E e : src)
			puhs(e)
	}

	// Integer is a subtype of Number
	Stack<Number> numberStack = new Stack<Number>();
	Iterable<Integer> integers = ...
	numberStack.pushAll(integers); //Error message here: List<Integer> is not a subtype of List<Number>
```

**Bounded wildcard type**

Producer

```java

	public void pushAll(Iterable<? Extends E> src){
		for (E e : src)
			push(e);
	}
```

Consumer

```java

	public void popAll(Collection<? super E> dst){
		while(!isEmpty())
			dst.add(pop());
	}
```

**PECS: producer-extends, consumer-super**

If the parameter is a producer and a conusmer don't use _wildcards_

Never use _wildcards_ in return values.

Type inference in generics

```java

	Set<Integer> integers =...
	Set<Double> doubles =...
	Set<Number> numbers = union(integers,doubles);//Error

	//Needs a 'explicit type parameter'
	Set<Number> numbers = Union.<Number>union(integers,doubles);
```

Comparable and Comparators are always consumers. Use `Comparable<? super T>` and `Comparator<? super T>`

If a type parameter appears only once in a method declaration, replace it with a wildcard.

## 29. Consider _typesafe heterogeneous containers_

A container for accessing a heterogeneous list of types in a typesafe way.

Thanks to the type of the class literal. `Class<T>`

**API**

```java

	public class Favorites{
		public void putFavorites(Class<T> type, T instance);
		public <T> getFavorite(Class<T> type);
	}
```

**Client**

```java

	Favorites f = new Favorites();
	f.putFavorites(String.class, "JAVA");
	f.putFavorites(Integer.class, 0xcafecace);
	f.putFavorites(Class.class, Favorite.class);

	String s = f.getFavorites(String.class);
	int i =f.getFavorites(Integer.class);
	Class<?> c = f.getFavorites(Class.class);
```

**Implementation**

```java

	public class Favorites{
		private Map<Class<?>, Object> favorites = new HashMap<Class<?>, Object>();

		public <T> void putFavorites(Class<T> type, T instance){
			if(type == null)
				throw new NullPointerException("Type is null");
			favorites.put(type, type.cast(instance));//runtime safety with a dynamic cast
		}

		public <T> getFavorite(Class<T> type){
			return type.cast(favorites.get(type));
		}
	}
```

