# 9. EXCEPTIONS

## 57. Use exceptions only for exceptional conditions

Exceptions are for exceptional conditions.  
Never use or (expose in the API) exceptions for ordinary control flow.

## 58. Use checked exceptions for recoverable conditions and runtime exceptions for programming errors

Throwables:

- checked exceptions: for conditions from which the caller can reasonably be expected to recover
- unchecked exceptions: shouldn't, be caught. recovery is impossible and continued execution would do more harm than good.
    - runtime exceptions: to indicate programming errors. The great majority indicate precondition violations.
    - errors : are reserved for use by the JVM. (as a convention)

Unchecked throwables that you implement should **always** subclass _RuntimeException_.

## 59. Avoid unnecessary use of checked exceptions

Use checked exceptions only if these 2 conditions happen:

- The exceptional condition cannot be prevented by proper use of the API
- The programmer using the API can take some useful action once confronted with the exception.

Refactor the checked exception into a unchecked exception to make the API more pleasant.

Invocation with checked exception

```java

	try {
		obj.action(args);
	} catch(TheCheckedException e) {
		// Handle exceptional condition
		...
	}
```

Invocation with state-testing method and unchecked exception

```java

	if (obj.actionPermitted(args)) {
		obj.action(args);
	} else {
		// Handle exceptional condition
		...
	}
```

## 60. Favor the use of standard exceptions

| Exception                       | Occasion for Use                                                              |
| ------------------------------- | ----------------------------------------------------------------------------- |
| IllegalArgumentException        | Non-null parameter value is inappropriate                                     |
| IllegalStateException           | Object state is inappropriate for method invocation                           |
| NullPointerException            | Parameter value is null where prohibited                                      |
| IndexOutOfBoundsException       | Index parameter value is out of range                                         |
| ConcurrentModificationException | Concurrent modification of an object has been detected where it is prohibited |
| UnsupportedOperationException   | Object does not support method                                                |

### Java 8 Exceptions

|                                |                                   |                                 |
| ------------------------------ | --------------------------------- | ------------------------------- |
| AclNotFoundException           | InvalidMidiDataException          | RefreshFailedException          |
| ActivationException            | InvalidPreferencesFormatException | RemarshalException              |
| AlreadyBoundException          | InvalidTargetObjectTypeException  | RuntimeException                |
| ApplicationException           | IOException                       | SAXException                    |
| AWTException                   | JAXBException                     | ScriptException                 |
| BackingStoreException          | JMException                       | ServerNotActiveException        |
| BadAttributeValueExpException  | KeySelectorException              | SOAPException                   |
| BadBinaryOpValueExpException   | LambdaConversionException         | SQLException                    |
| BadLocationException           | LastOwnerException                | TimeoutException                |
| BadStringOperationException    | LineUnavailableException          | TooManyListenersException       |
| BrokenBarrierException         | MarshalException                  | TransformerException            |
| CertificateException           | MidiUnavailableException          | TransformException              |
| CloneNotSupportedException     | MimeTypeParseException            | UnmodifiableClassException      |
| DataFormatException            | MimeTypeParseException            | UnsupportedAudioFileException   |
| DatatypeConfigurationException | NamingException                   | UnsupportedCallbackException    |
| DestroyFailedException         | NoninvertibleTransformException   | UnsupportedFlavorException      |
| ExecutionException             | NotBoundException                 | UnsupportedLookAndFeelException |
| ExpandVetoException            | NotOwnerException                 | URIReferenceException           |
| FontFormatException            | ParseException                    | URISyntaxException              |
| GeneralSecurityException       | ParserConfigurationException      | UserException                   |
| GSSException                   | PrinterException                  | XAException                     |
| IllegalClassFormatException    | PrintException                    | XMLParseException               |
| InterruptedException           | PrivilegedActionException         | XMLSignatureException           |
| IntrospectionException         | PropertyVetoException             | XMLStreamException              |
| InvalidApplicationException    | ReflectiveOperationException      | XPathException                  |

## 61. Throw exceptions appropriate to the abstraction

Higher layers should catch lower-level exceptions and, in their place, throw exceptions that can be explained in terms of the higher-level abstraction.

```java

	// Exception Translation
	try {
		// Use lower-level abstraction to do our bidding
		...
	} catch(LowerLevelException e) {
		throw new HigherLevelException(...);
	}
```

Do not overuse. The best way to deal with exceptions from lower layers is to avoid them, by ensuring that lower-level methods succeed.

**Exception chaining**
When the lower-level exception is utile for the debugger, pass the lower-level to the higher-level exception, with an accessor method (Throwable.getCause) to retrieve the lower-level exception.

```java

	// Exception with chaining-aware constructor
	class HigherLevelException extends Exception {
		HigherLevelException(Throwable cause) {
		super(cause);
		}
	}
```

## 62. Document all exceptions thrown by each method

Unchecked exceptions generally represent programming errors ([Item 58](#58-use-checked-exceptions-for-recoverable-conditions-and-runtime-exceptions-for-programming-errors)), and familiarizing programmers with all of the errors they can make helps them avoid making these errors.

Always declare checked exceptions individually, and document precisely the conditions under which each one is thrown using the Javadoc @throws tag.

Do not use the throws keyword to include unchecked exceptions in the method declaration.

## 63. Include failure-capture information in detail messages

It is critically important that the exception’s `toString` method return as much information as possible concerning
the cause of the failure.
To capture the failure, the detail message of an exception should contain the values of all parameters and fields that contributed to the exception.
One way to ensure that is to require this information in their constructors instead of a string detail message. Also provide accessors to this parameters could help useful to recover from the failure

```java

	// Alternative IndexOutOfBoundsException.
	public IndexOutOfBoundsException(int lowerBound, int upperBound, int index) {...}
```

## 64. Strive for failure atomicity

A failed method invocation should leave the object in the state that it was in prior to the invocation.
Options to achieve this:

- Design immutable objects
- Order the computation so that any part that may fail takes place before any part that modifies the object.
- Write recovery code (Undo operation)
- Perform the operation on a temporary copy of the object, and replace it once is completed.

## 65. Don't ignore exceptions

Don't let catch blocks empty.

```java

	// Empty catch block ignores exception - Highly suspect!
	try {
	...
	} catch (SomeException e) {
	}
```

