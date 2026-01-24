# 11. SERIALIZATION

## 74. Implement _Serializable_ judiciously

Adding `implements Serializable` is the easiest way to serialize a class, but it decreases the flexibility
to change a class's implementation once it has been released. The byte-stream encoding (or serialized form)
becomes part of its exported API.

It has three major drawbacks:

- Class's private and package-private instance fields become part of its exported API ([Item 13](#13-minimize-the-accessibility-of-classes-and-members))
- Change the class's internal representation, will cause make old versions of serialized objects incompatible.
- Increases the likelihood of bugs and security holes.
- Increases the testing burden associated with releasing a new version of a class.

Implementing the Serializable interface has many real costs.

**should implement Serializable:**

- value classes such as _Date_ and _BigInteger_
- as should most collection classes

**rarely implement Serializable:**

- Classes representing active entities, such as thread pools
- Classes designed for inheritance
- Interfaces should rarely extend it
- Inner classes ([Item 22](#22-favor-static-member-classes-over-nonstatic))

A subclass of a not serializable class can not be serializable, unless it has a parameterless constructor.

## 75. Consider using a custom serialized form

Do not accept the default serialized form without first considering whether it is appropriate.  
The default serialized form is likely to be appropriate if an object's physical representation is identical to its logical content. Like a Point or Person Name.  
Even if you decide that the default serialized form is appropriate, you often must provide a `readObject` method to ensure invariants and security

Using the default serialized form when an object's physical representation differs substantially from its logical data content has four disadvantages:

- It permanently ties the exported API to the current internal representation.
- It can consume excessive space.
- It can consume excessive time.
- It can cause stack overflows.

Every instance field that is not labeled _transient_ will be serialized when the defaultWriteObject method is invoked
(Whether or not you use the default serialized)  
Every instance field that can be made transient should be made so.(i.e. computed from “primary data fields)
Mark _nontransient_ every field whose value is part of the logical state of the object.

Impose synchronization on object serialization that you would impose on any other method that reads the entire state of the object.

```java

	// writeObject for synchronized class with default serialized form
	private synchronized void writeObject(ObjectOutputStream s)
		throws IOException {
			s.defaultWriteObject();
	}
```

Declare an explicit serial version UID in every serializable class you write.

```java

	private static final long serialVersionUID = randomLongValue ;
```

## 76. Write _readObject_ methods defensively

_readObject_ method is a public constructor that takes a byte stream as its sole parameter. It demands same care as any other public constructor:

- check its arguments for validity ([Item 38](#38-check-parameters-for-validity))
- make defensive copies of parameters where appropriate ([Item 39](#39-make-defensive-copies-when-needed))

```java

	// readObject method with defensive copying and validity checking
	private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
		s.defaultReadObject();

		// Defensively copy our mutable components
		start = new Date(start.getTime());
		end = new Date(end.getTime());

		// Check that our invariants are satisfied
		if (start.compareTo(end) > 0)
			throw new InvalidObjectException(start +" after "+ end);
	}
```

Every serializable immutable class containing private mutable components must defensively copy these components in its readObject method.

Do not use the _writeUnshared_ and _readUnshared_ methods(Java 1.4). Are faster but not safer than defensive copying.

Summary guidelines:

- For classes with object reference fields that must remain private, defensively copy each object in such a field. Mutable components of immutable classes fall into this category.
- Check any invariants and throw an InvalidObjectException if a check fails. The checks should follow any defensive copying.
- If an entire object graph must be validated after it is deserialized, use the _ObjectInputValidation_ interface \[JavaSE6, Serialization\].
- Do not invoke any overridable methods in the class, directly or indirectly.

## 77. For instance control, prefer _enum_ types to _readResolve_

_Singleton_ classes would no longer be singletons if they “implements Serializable”.
The _readResolve_ feature allows you to substitute another instance for the one created by _readObject_. So the original instance is returned.

To prevent attacks when using _readResolve_ for instance control, all instance fields with object reference types must be declared transient.

Another way to prevent attacks in instance-controlled classes and when instances are known at compile time,is using _enum_. JVM guarantees that only will be one instance.

Accessibility: _readResolve_ method on:

- final class: private.
- nonfinal class:
    - private: will not apply to any subclasses.
    - package-private: it will apply only to subclasses in the same package.
    - protected or public: it will apply to all subclasses that do not override it.

## 78. Consider serialization proxies instead of serialized instances

_serialization proxy_: A private static nested class of the serializable class that represents the logical state of an instance of the enclosing class.  
It has a single constructor, whose parameter type is the enclosing class, and copies the data from its arguments.  
No need of consistency checking or defensive copying.
Both the enclosing class and its serialization proxy must be declared to implement Serializable.

```java


		// Serialization proxy for Period class
		private static class SerializationProxy implements Serializable {
			private final Date start;
			private final Date end;

			SerializationProxy(Period p) {
				this.start = p.start;
				this.end = p.end;
			}

			private static final long serialVersionUID = 234098243823485285L; // Any number will do (Item 75)
		}


	}
```

_writeReplace_ translates an instance of the enclosing class to its serialization proxy prior to serialization.
The serialization system will never generate a serialized instance of the enclosing class.

```java

	// writeReplace method for the serialization proxy pattern
	private Object writeReplace() {
		return new SerializationProxy(this);
	}
```

If an attacker fabricates a serialized object in an attempt to violate the class's invariants, we throw an Exception.

```java

	// readObject method for the serialization proxy pattern
	private void readObject(ObjectInputStream stream) throws InvalidObjectException {
		throw new InvalidObjectException("Proxy required");
	}
```

Add a _readResolve_ method on the _SerializationProxy_ class to return a logically equivalent instance of the enclosing class.

```java

	// readResolve method for Period.SerializationProxy
	private Object readResolve() {
		return new Period(start, end); // Uses public constructor
	}
```

Limitations, not compatible with:

- classes that are extendable by their clients ([Item 17](#17-design-and-document-for-inheritance-or-else-prohibit-it))
- some classes whose object graphs contain circularities