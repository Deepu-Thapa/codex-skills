# 8. GENERAL PROGRAMMING

## 45. Minimize the scope of local variables.

Declare local variable where it is first used.  
Most local variable declaration should contain an initializer.  
Prefer for loops to while loops.  
Keep methods small and focused.

## 46. Prefer for-each loops to traditional for loops.

```java

	 // No longer the preferred idiom to iterate over a collection!
		for (Iterator i = c.iterator(); i.hasNext(); ) {
			doSomething((Element) i.next()); // (No generics before 1.5)
	}
	// No longer the preferred idiom to iterate over an array!
	   for (int i = 0; i < a.length; i++) {
	       doSomething(a[i]);
	}
```

Use for each loop:

```java

	// The preferred idiom for iterating over collections and arrays
	   for (Element e : elements) {
	       doSomething(e);
	}
```

Error when iterating in nested loops

```java

	 // Can you spot the bug?
	   enum Suit { CLUB, DIAMOND, HEART, SPADE }
	   enum Rank { ACE, DEUCE, THREE, FOUR, FIVE, SIX, SEVEN, EIGHT,
	               NINE, TEN, JACK, QUEEN, KING }
	   ...
	   Collection<Suit> suits = Arrays.asList(Suit.values());
	   Collection<Rank> ranks = Arrays.asList(Rank.values());
	   List<Card> deck = new ArrayList<Card>();
	   for (Iterator<Suit> i = suits.iterator(); i.hasNext(); )
	       for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
	           deck.add(new Card(i.next(), j.next())); // i.next() should be run only in the outer loop

```

A solution

```java

	// Fixed, but ugly - you can do better!
	for (Iterator<Suit> i = suits.iterator(); i.hasNext(); ) {
	   Suit suit = i.next();
	   for (Iterator<Rank> j = ranks.iterator(); j.hasNext(); )
			deck.add(new Card(suit, j.next()));
	}
```

For each loop fix this directly

```java

	// Preferred idiom for nested iteration on collections and arrays
	   for (Suit suit : suits)
	       for (Rank rank : ranks)
	           deck.add(new Card(suit, rank));
```

Situations where you can’t use a for-each loop:

- **Filtering—If** you need to traverse a collection and remove selected elements, then you need to use an explicit iterator so that you can call its remove method.
- **Transforming—If** you need to traverse a list or array and replace some or all of the values of its elements, then you need the list iterator or array index in order to set the value of an element.
- **Parallel iteration—If** you need to traverse multiple collections in parallel, then you need explicit control over the iterator or index variable, so that all it- erators or index variables can be advanced in lockstep (as demonstrated unin-tentionally in the buggy card examples above).

## 47. Know and use libraries

By using a standard library:

- Advantage of the knowledge of the experts who wrote it and the experience of those who used it before you.
- Don’t have to waste your time writing ad hoc solutions to problems that are only marginally related to your work.
- Their performance tends to improve over time
- Your code will be easily readable, maintainable, and reusable.

Numerous features are added to the libraries in every major release, and it pays to keep abreast of these additions

Every programmer should be familiar with:

- java.lang
- java.util
- java.io
- java.util.concurrent

## 48. Avoid float and double if exact answer are required

For monetary calculations use _int_(until 9 digits) or _long_ (until 18 digits) taken you care of the decimal part and you don't care too much about the rounding. Use _BigDecimal_ for numbers bigger that 18 digits and if you need full control of the rounding methods used.

## 49. Prefer primitive types to boxed primitives

Primitives: _int_, _double_, _boolean_  
Boxed Primitives: _Integer_, _Double_, _Boolean_  
Differences:

- Two boxed primitives could have the same value but different identity.
- Boxed primitives have one nonfunctional value: _null_.
- Primitives are more space and time efficient.

Don't use **==** between boxed primitives.

```java

	first = new Integer(1);
	second = new Integer(1);
	first == second; //Uses unboxing  Don't have to be true.
```

Use Auto-unboxing to create new primitives

```java

	...
	int f = first;  //Auto-unboxing
	int s = second  //Auto-unboxing
	f == s;// This is true
```

If a Boxed primitive is not initialize it will return null

```java

	Integer  i;
	i == 42 // NullPointerException
```

Performance can be perturbed when boxing primitives values due to the creation of unnecessary objects.

When you **must** use boxed primitives:

- As elements, keys and values in Collections
- As type parameters in parametrized types ([Chapter 5](#5-generics))
- When making reflective invocations ([Item 53](#53-prefer-interfaces-to-reflection))

In other cases prefer primitives.

## 50. Avoid Strings where other types are more appropriate

- Strings are more cumbersome than other types.
- Strings are less flexible than other types.
- String are slower than other types.
- Strings are more error-prone than other types.
- Strings are poor substitutes for other value types.
- Strings are poor substitutes for _enum_ types.
- Strings are poor substitutes for _aggregate_ types.
- Strings are poor substitutes for _capabilities_.

So, use String to represent text!

## 51. Beware the performance of string concatenation

Using the string concatenation operator repeatedly to concatenate _n_ strings requires time quadratic in _n_.

```java

	// Inappropriate use of string concatenation - Performs horribly!
	public String statement()
		String result = "";
		for (int i = 0; i < numItems(); i++)
			result += lineForItem(i);
		return result;

```

To achieve acceptable performance, use StringBuilder in place of String.

```java

	public String statement(){
		StringBuilder b = new StringBuilder(numItems() * LINE_WIDTH);
		for (int i = 0; i < numItems(); i++)
			b.append(lineForItem(i));
		return b.toString();
	}
```

## 52. Refer to objects by their interface

If appropriate interface types exist, then parameters, return values, variables, and fields should all be declared using interface types.

```java

	// Good - uses interface as type
	List<Subscriber> subscribers = new Vector<Subscriber>();
```

rather than this:

```java

	// Bad - uses class as type!
	Vector<Subscriber> subscribers = new Vector<Subscriber>();
```

It makes the program much more flexible. We could change the implementation of the `subscribers` changing just one line.

```java

	List<Subscriber> subscribers = new ArrayList<Subscriber>();
```

**Caveat**: if the original implementation has a special functionality not required the interface contract and the code dependent on that functionality, the new implementation must provide this functionality.

If there is not an appropriate interface we can refer to the object by a class. Like:

- Value classes: String, BigDecimal...
- Framework classes
- Classes that extend the interface functionality with extra methods.

## 53. Prefer interfaces to reflection

_java.lang.reflection_ offers access to information about loaded classes.

Given a _Class_ object, you can obtain _Constructor_, _Method_ and _Field_ instances.

Allows one class to use another, even if the latter class did not exist when the former was compiled.

- Lose all benefits of compile-time type checking
- Code to perform reflective access is clumsy and verbose
- Performance suffers.

**As a rule, objects should not be accessed reflectively in normal applications at runtime**

Obtain many of the benefits of reflection incurring few of its costs by **creating instances reflectively and access them normally via their interface or superclass**.

```java

	// Reflective instantiation with interface access
	public static void main (String[] args){
		// Translate the class name into a class object
		Class<?> cl =  null;
		try{
			cl = Class.forName(args[0]);// Class is specified by the first command line argument
		}catch(ClassNotFoundException e){
			System.err.println("Class not found");
			System.exit(1);
		}

		//Instantiate the class
		Set<String> s = null;
		try{
			s = (Set<String>) cl.newInstance(); //  The class can be either a HashSet or a TreeSet
		} catch(IllegalAccessException e){
			System.err.println("Class not accessible");
			System.exit(1);
		}catch(InstantionationException e){
			System.err.println("Class not instantiable");
			System.exit(1);
		}

		//Excercise the Set
		// Print the remaining arguments. The order depends in the class. If it is a HashSet
		// the order will be random, if it is a TreeSet it will be alphabetically
		s.addAll(Arrays.asList(args).subList(1,args.length));
		System.out.println(s);
	}
```

A legitimate use of reflection is to manage a class's dependencies on other classes, methods or fields that may be absent at runtime.

Reflection is powerful and useful in some sophisticated systems programming tasks. It has many disadvantages.
Use reflection, if possible, only to instantiate objects and access the objects using an interface or a superclass that is known at compile time.

## 54. Use native methods judiciously

Historically, native methods have had three main uses.

- They provided access to platform-specific facilities.
- They provided access to libraries of legacy code.
- To write performance-critical parts

New Java versions make use of NDK rarely advisable for improve performance.

## 55. Optimize judiciously

Strive to write good programs rather than fast ones, speed will follow.  
If a good program is not fast enough, its architecture will allow it to be optimized.

- More computing sins are committed in the name of efficiency (without necessarily achieving it) than for any other single reason — including blind stupidity.
- We should forget about small efficiencies, say about 97% of the time: premature
  optimization is the root of all evil.
- We follow two rules in the matter of optimization:
    - Rule 1. Don't do it.
    - Rule 2 (for experts only). Don't do it yet — that is, not until you have a perfectly clear and unoptimized solution.

If you finally do it **measure performance before and after each attempted optimization**, and focus firstly in the choice of algorithms rather than in low level optimizations.

## 56. Adhere to generally accepted naming conventions

**Typographical naming conventions**

| Indentifier Type   | Examples                                      |
| ------------------ | --------------------------------------------- |
| Package            | com.google.inject, org.joda.time.format       |
| Class or Interface | Timer, FutureTask, LinkedHashMap, HttpServlet |
| Method or Field    | remove, ensureCapacity, getCrc                |
| Constant Field     | MIN_VALUE, NEGATIVE_INFINITY                  |
| Local Variable     | i, xref, houseNumber                          |
| Type Parameter     | T, E, K, V, X, T1, T2                         |

**Grammatical naming conventions**

| Type                                                                    | Convention                                   | Example                                                   |
| ----------------------------------------------------------------------- | -------------------------------------------- | --------------------------------------------------------- |
| Classes and enum types                                                  | Singular noun or noun phrase                 | Timer, BufferedWriter, ChessPiece                         |
| Interfaces                                                              | Like classes                                 | Collection, Comparator                                    |
| Interfaces                                                              | With an adjective ending in _able_ or _ible_ | Runnable, Iterable, Accessible                            |
| Annotation types                                                        | Nouns, verbs, prepositions, adjectives ...   | BindingAnnotation, Inject, ImplementedBy, Singleton       |
| Static factories (common names)                                         | ---                                          | valueOf, of, getInstance, newInstance, getType, newType   |
| **Methods that...**                                                     | ---                                          | ---                                                       |
| perform actions                                                         | verb or verb phrase                          | append, drawImage                                         |
| return a boolean                                                        | names beginning with _is_ or, _has_          | isDigit, isProbablePrime, isEmpty, isEnabled, hasSiblings |
| return a non-boolean or attribute                                       | noun, a noun phrase, or begin with _get_     | size, hashCode, or getTime                                |
| convert the type of an object                                           | _toType_                                     | toString, toArray                                         |
| return a view ([Item 5](#5-avoid-creating-objects)) of a different type | _asType_                                     | asList                                                    |
| return a primitive with the same value                                  | _typeValue_                                  | intValue                                                  |

