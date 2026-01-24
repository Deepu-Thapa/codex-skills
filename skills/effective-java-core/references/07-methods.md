# 7. METHODS

## 38. Check parameters for validity

Check parameters before execution as soon as possible.

Add in public methods _@throw_, and use _assertions_ in non public methods

Do it also in constructors.

## 39. Make defensive copies when needed.

You must program defensively, with the assumption that clients of your class will do their best to destroy its invariants.

```java

	//Broken "immutable" time period
	public final class Period{
		private final Date start;
		private final Date end;
		/**
		* @param start the beginning of the period
		* @param end the end of the period; must not precede start;
		* @throws IllegalArgumentException if start is after end
		* @throws NullPointerException if start or end is null
		*/
		public Period(Date start, Date end) {
			if(start.compare(end) > 0)
				throw new IllegalArgumentException(start + " after " + end );
			this.start = start;
			this.end = end;
		}

		public Date start(){
			return start;
		}

		public Date end(){
			return end;
		}
		...
	}
```

Attack. Because the client keep a copy (pointer) of the parameter, it can always change it after the constructor.

```java

	Date start = new Date();
	Date end = new Date();
	Period p = new Period(start, end);
	end.setYear(78)// Modifies internal of p!
```

Make a _defensive copy_ of each mutable parameter to the constructor.

```java

	public Period(Date start, Date end) {
		this.start = new Date(start.getTime());
		this.end = new Date(end.getTime());
		if(start.compare(end) > 0)
			throw new IllegalArgumentException(start + " after " + end );
		}
```

Defensive copies are made before checking the validity of the parameter ([Item 38](#38-check-parameters-for-validity)), and the validity check is performed on the copies rather than on the originals. It protects the class against changes to the parameters from another thread during the time between the parameters are checked and the time they are copied.(_Window of vulnerability_,time-of-check/time-of-use _TOCTOU_ attack)

Do not use _clone_ method to make a defensive copy of a parameter whose type is subclass-able by untrusted parties.

Second Attack. Because the accessors returns the object used in the Period class, the client can change its value without passing the constrains.

```java

	Date start = new Date();
	Date end = new Date();
	Period p = new Period(start, end);
	p.end.setYear(78)// Modifies internal of p!
```

Return _defensive copies_ of mutable internal fields.

```java

	public Date start(){
		return new Date(start.getTime());
	}

	public Date end(){
		return new Date(end.getTime());
	}
```

Preferable is to use **immutable objects**([Item 15](#15-minimize-mutability))

## 40. Design method signatures carefully

- Choose method names carefully. ([Item 56](#56-adhere-to-generally-accepted-naming-conventions))
- Don't go overboard in providing convenience methods. Don't add too many.
- Avoid long parameter list. Make a subset of methods, helper classes ([Item 22](#22-favor-static-member-classes-over-nonstatic)), or a builder ([Item 2](#2-use-builders-when-faced-with-many-constructors)) instead.
- For parameter types, favor interfaces over classes ([Item 52](#52-refer-to-objects-by-their-interface)) No reason to write a method that takes a _HashMap_ on input, use _Map_ instead.
- Prefer two-element enum types to _boolean_ parameters. `public enum TemperatureScale {CELSIUS, FARENHEIT}`

## 41. Use overloading judiciously

The choice of which overloading to invoke is made at compile time.
Selection among overloaded methods is static, while selection among overridden methods is dynamic.

```java

	// Broken! - What does this program print?
	public class CollectionClassifier {
		public static String classify(Set<?> s) {
			return "Set";
		}
		public static String classify(List<?> lst) {
			return "List";
		}
		public static String classify(Collection<?> c) {
			return "Unknown Collection";
		}
		public static void main(String[] args) {
			Collection<?>[] collections = {
				new HashSet<String>(),
				new ArrayList<BigInteger>(),
				new HashMap<String, String>().values()
			};
		for (Collection<?> c : collections)
			System.out.println(classify(c)); // Returns "Unknown Collection" 3 times
		}
	}
```

Overriding works different. The “most specific” overriding method always gets executed.

```java

	class Wine {
		String name() { return "wine"; }
	}
	class SparklingWine extends Wine {
		@Override String name() { return "sparkling wine"; }
	}
	class Champagne extends SparklingWine {
		@Override String name() { return "champagne"; }
	}
	public class Overriding {
		public static void main(String[] args) {
			Wine[] wines = {
				new Wine(), new SparklingWine(), new Champagne()
			};
			for (Wine wine : wines)
				System.out.println(wine.name()); // prints: wine, sparkling wine, and champagne
		}
	}
```

Overloading does not give the functionallity we want in the first sample. A possible solution is:

```java

	public static String classify(Collection<?> c) {
		return 	c instanceof Set ? "Set" :
				c instanceof List ? "List" : "Unknown Collection";
	}
```

Do not have overloaded methods in APIs to avoid confusing the clients of the API.

A conservative policy is to never export two overloadings with the same number of parameters. Use different names.`writeBoolean(boolean)`, `writeInt(int)`, and `writeLong(long)`

For constructors you can use static factories ([Item 1](#1-use-static-factory-methods-instead-of-constructors))

If parameters are radically different this rule can be violated but always ensure that all overloadings behave identically
when passed the same parameters. To ensure this, have the more specific overloading forward to the more general.

```java

	public boolean contentEquals(StringBuffer sb) {
		return contentEquals((CharSequence) sb);
	}
```

## 42. Use varargs judiciously

varargs methods are a convenient way to define methods that require a variable number of arguments, but they should not be overused.

```java

	// The right way to use varargs to pass one or more arguments
	static int min(int firstArg, int... remainingArgs) {
		int min = firstArg;
		for (int arg : remainingArgs)
			if (arg < min)
				min = arg;
		return min;
	}
```

## 43. Return empty arrays or collections, not nulls

There is no reason ever to return null from an array- or collection-valued method instead of returning an empty array or collection

Return an immutable empty array instead of null.

```java

	// The right way to return an array from a collection
	private final List<Cheese> cheesesInStock = ...;

	private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

	/**
	* @return an array containing all of the cheeses in the shop.
	*/
	public Cheese[] getCheeses() {
		return cheesesInStock.toArray(EMPTY_CHEESE_ARRAY);
	}
```

In Collections emptySet, emptyList and emptyMap methods do the same job.

```java

	// The right way to return a copy of a collection
	public List<Cheese> getCheeseList() {
		if (cheesesInStock.isEmpty())
			return Collections.emptyList(); // Always returns same list
		else
			return new ArrayList<Cheese>(cheesesInStock);
	}
```

## 44. Write _doc comments_ for all exposed API elemnts

To document your API properly, you must precede every exported class, interface, constructor, method, and field declaration with a doc comment.

The doc comment for a method should describe succinctly:

- The _contract_ between the method and its client, what the method does rather than how it does its job.
- The _preconditions_ described implicity by the _@throws_ tag.
- The _postconditions_, things that will be true after the invocation has completed successfully.
- The _side effects_, change in the state of the system.
- The _thread safety_ of the class and methods
- The _summary description of the element_, the first “sentence” of each doc comment.Should not contains a space after a period.

```java

	/**
	* Returns the element at the specified position in this list. *
	* <p>This method is <i>not</i> guaranteed to run in constant
	* time. In some implementations it may run in time proportional * to the element position.
	*
	* @param index index of element to return; must be
	* non-negative and less than the size of this list
	* @return the element at the specified position in this list
	* @throws IndexOutOfBoundsException if the index is out of range
	*		({@code index < 0 || index >= this.size()})
	*/
	E get(int index);
```

Have special care in:

- Generics: document all type parameters
- Enums: document all the constants, the type and the public methods.
- Annotatons: document all members an the type.

Don't forget to documment:

- The _thread-safety level_ ([Item 70](#70-document-thread-safety))
- The _serialized form_ ([Item 75](#75-consider-using-a-custom-serialized-form)), if the class is _serializable_

