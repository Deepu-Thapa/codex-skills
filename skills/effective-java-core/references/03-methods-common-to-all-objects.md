# 3. METHODS COMMON TO ALL OBJECTS

## 8. Obey the general contract when overriding _equals_

**_Don't override if:_**

- Each instance of the class is inherently unique. I.e._Thread_
- You don't care whether the class provides a "logical equality" test. I.e. _java.util.Random_
- A superclass has already overridden _equals_, and the superclass behavior is appropriate for this class I.e. _Set_
- The class is private or package-private, and you are certain that its _equals_ method will never be invoked

**_Override if:_**

A class has a notion of _logical equality_ that differs from mere object identity, and a superclass has not already overridden _equals_ to implement the desired behavior.

**_Equals implements an "equivalence relation"_**

- Reflexive: _x.equals(x)==true_
- Symmetric: _x.equals(y)==y.equals(x)_
- Transitive: _x.equals(y)==y.equals(z)==z.equals(x)_
- Consistent: _x.equals(y)==x.equals(y)==x.equals(y)==..._
- Non-nullity: _x.equals(null)->false_

**_The Recipe_**

1. Use the == operator to check if the argument is a reference to this object (for performance)
2. Use the _instanceof_ operator to check if the argument has the correct type
3. Cast the argument to the correct type
4. For each "significant" field in the class, check if that field of the argument matches the corresponding field of this object
5. When you are finished writing your _equals_ method, ask yourself three questions: Is it Symmetric? Is it Transitive? Is it Consistent? (the other 2 usually take care of themselves)

```java

	@Override
	public boolean equals (Object o){
		if(o == this)
			return true;

		if (!(o instanceof PhoneNumber))
			return false;

		PhoneNumber pn = (PhoneNumber)o;
		return pn.lineNumber == lineNumber
			&& pn.prefix == prefix
			&& pn.areaCode == areaCode;
	}
```

**_Never Forget_**

- Always override _hashCode_ when you override _equals_
- Don't try to be too clever (simplicity is your friend)
- Don't substitute another type for _Object_ in the _equals_ declaration

## 9. Always override _hashCode_ when you override _equals_

**_Contract of hashCode_**

- Whenever _hashCode_ is invoked in the same object it should return the same integer.
- If two objects are equals according to the _equals_, the should return the same integer calling _hashCode_.
- Is not required (but recommended) that two non _equals_ objects return distinct _hashCode_.

The second condition is the one that is more often violated.

**_The Recipe_**

1. Store constant value i.e. 17 in an integer called _result_.
2. For each field _f_ used in _equals_ do:

- Compute _c_
    - boolean: _(f ? 1 : 0)_
    - byte, char, short or int: _(int) f_
    - long: _(int) (f ^ (.f >>> 32))_
    - float: _Float.floatToIntBits(f)_
    - double: _Double.doubleToLongBits(f)_ and compute as a long
    - object reference: if _equals_ of the reference use recutsivity, use recursivity for the _hashCode_
    - array: each element as a separate field.
- Combine: _result = 31 \* result + c_

3. Return _result_
4. Ask yourself if equal instances have equal hash codes.

```java

	private volatile int hashCode; // Item 71 (Lazily initialized, cached hashCode)

	@Override public int hashCode(){
		int result = hashCode;
		if (result == 0){
			result = 17;
			result = 31 * result + areaCode;
			result = 31 * result + prefix;
			result = 31 * result + lineNumber;
			hashCode = result;
		}
		return result;
	}
```

## 10. Always override _toString_

Providing a good _toString_ implementation makes your class much more pleasant to read.

When practical, the _toString_ method return all of the interesting information contained in the object.

It is possible to specify the format of return value in the documentation.

Always provide programmatic access to all of the information contained in the value returned by _toString_ so the users of the object don't need to parse the output of the _toString_

## 11. Override _clone_ judiciously

Cloneable interface does not contain methods
If a class implements Cloneable, Object's clone method returns a field-by-field copy of the object.
Otherwise it throws CloneNotSupportedException.

If you override the clone method in a nonfinal class, you should return an object obtained by invoking _super.clone_.
A class that implements _Cloneable_ is expected to provide a properly functioning public _clone_ method.

Simple clone method if object does **not** contain fields that refer to mutable objects.

```java

	@Override public PhoneNumber clone() {
		try {
			//PhoneNumber.clone must cast the result of super.clone() before returning it.
			return (PhoneNumber) super.clone();
		} catch(CloneNotSupportedException e) {
			throw new AssertionError(); // Can't happen
		}
	}
```

If object **contains** fields that refer to mutable objects, we need another solution. Mutable fields will point to same objects in memory and the original and the cloned method will share these objects.

_clone_ is another constructor and therefore it must ensure not harming the original object and establishing invariants.  
Calling _clone_ recursively in the mutable objects is the easiest way.

```java

	@Override public Stack clone() {
		try {
			Stack result = (Stack) super.clone();
			// From Java 1.5, don't need casting when cloning arrays
			result.elements = elements.clone();
			return result;
		} catch (CloneNotSupportedException e) {
			throw new AssertionError();
		}
	}
```

Mutable objects and finals: The _clone_ architecture is incompatible with normal use of final fields referring to mutable objects.
More complex objects would need specific approaches where recursively calling _clone_ won't work.

A _clone_ method should not invoke any nonfinal methods on the clone under construction ([Item 17](#17-design-and-document-for-inheritance-or-else-prohibit-it)).

Object's _clone_ method is declared to throw _CloneNotSupportedException_, but overriding clone methods can omit this declaration.  
Public _clone_ methods should omit it. ([Item 59](#59-avoid-unnecessary-use-of-checked-exceptions)).  
If a class overrides clone, the overriding method should mimic the behavior of _Object.clone_:

- it should be declared protected,
- it should be declared to throw CloneNotSupportedException,
- it should not implement Cloneable.

Subclasses are free to implement Cloneable or not, just as if they extended Object directly

_clone_ method must be properly synchronized just like any other method ([Item 66](#66-synchronize-access-to-shared-mutable-data)).

Summary: classes that implement Cloneable should create a method that:

- override clone
- return type is the class
- call _super.clone_
- fix fields that need to be fixed

Better provide an alternative of object copying, or don't provide it at all.

**Copy Constructor**

```java

	public Yum(Yum yum);
```

**Copy Factory**

```java

	public static Yum newInstance(Yum yum);
```

These alternatives:

- don't rely on a risk-prone extra-linguistic object creation mechanism
- don't demand adherence to thinly documented conventions
- don't conflict with the proper use of final fields
- don't throw unnecessary checked exceptions
- don't require casts.

Furthermore they can use its Interface-based copy constructors and factories, _conversion constructors_ and _conversion factories_ and allow clients to choose the implementation type `public HashSet(Set set) -> TreeSet;`

## 12. Consider implementing _Comparable_

_Comparable_ is an interface. It is not declared in _Object_

Sorting an array of objects that implement _Comparable_ is as simple as `Arrays.sort(a);`

The class will interoperate with many generic algorithms and collection implementations that depend on this interface. You gain lot of power with small effort.

Follow this provisions (Reflexive, Transitive, Symmetric):

1. `if a > b then b < a` `if a == b then b == a` `if a < b then b > a`
2. `if a > b and b > c then a > c`
3. `if a ==  b and b == c then a == c`
4. Strong suggestion: `a.equals(b) == a.compareTo(b)`

For integral primitives use `<` and `>`operators.

For floating-point fields use _Float.compare_ or _Double.compare_

For arrays start with the most significant field and work your way down.

