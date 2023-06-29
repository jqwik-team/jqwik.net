# jqwik Cheatsheet

_Updated for jqwik version 1.7.3_

_See the [jqwik user guide](https://jqwik.net/docs/current/user-guide.html) for a comprehensive coverage of jqwik features._

<!-- Generated toc must be stripped of `nbsp` occurrences in links -->
<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
## Table of Contents

- [Properties](#properties)
- [Constrain Data Generation](#constrain-data-generation)
  - [Annotations](#annotations)
  - [Assumptions](#assumptions)
- [Constraining Annotations](#constraining-annotations)
  - [Strings](#strings)
  - [Lists, Sets, Arrays and Stream](#lists-sets-arrays-and-stream)
  - [Integral Numbers (Integer etc.](#integral-numbers-integer-etc)
  - [Decimal Numbers (Float, Double, BigDecimal](#decimal-numbers-float-double-bigdecimal)
- [Programmatic Data Generation](#programmatic-data-generation)
- [Important Methods in Arbitraries class](#important-methods-in-arbitraries-class)
  - [Generate selection of values](#generate-selection-of-values)
  - [Numbers and String](#numbers-and-string)
  - [Choose among Arbitraries](#choose-among-arbitraries)
- [Important Methods of all Arbitrary types](#important-methods-of-all-arbitrary-types)
  - [Filter](#filter)
  - [Map](#map)
  - [Flat Map](#flat-map)
  - [Generate Collections](#generate-collections)
  - [Generate Nulls](#generate-nulls)
  - [Generate Optionals](#generate-optionals)
- [Combine Arbitraries](#combine-arbitraries)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


## Properties

```java
import java.util.*;
import net.jqwik.api.*;

class ListReverseProperties {

  @Property(tries = 100) // tries is optional
  @Report(Reporting.GENERATED) // optional
  boolean reverseTwiceIsOriginal(@ForAll List<Integer> original) {
    return reverse(reverse(original)).equals(original);
  }
  
  private <T> List<T> reverse(List<T> original) {
    List<T> clone = new ArrayList<>(original);
    Collections.reverse(clone);
    return clone;
  }
}
```

## Constrain Data Generation

### Annotations

```java
@Property
void reverseMakesFirstElementLast(
  @ForAll @Size(min = 2, max = 50)
    List<@IntRange(min = -100, max = 100) Integer> original
) {
    Integer lastReversed = reverse(original).get(original.size() - 1);
    Assertions.assertEquals(lastReversed, original.get(0));
}
```

### Assumptions

```java
@Property
void reverseMakesFirstElementLast(@ForAll List<Integer> original) {
    Assume.that(original.size() > 2);
    Integer lastReversed = reverse(original).get(original.size() - 1);
    Assertions.assertEquals(lastReversed, original.get(0));
}
```

## Constraining Annotations

### Strings

`@StringLength(int value = 0, int min = 0, int max = 0)`

`@Chars(chars[] value = {‘C‘,‘G‘,‘A‘,‘T‘})`: Specify a set of characters

`@CharRange(char from = 0, char to = 0)`: Specify start & end character

`@NumericChars`: Use digits 0 through 9

`@LowerChars`: Use lower case chars a through z

`@UpperChars`: Use upper case chars A through Z

`@AlphaChars`: Lower and upper case chars are allowed

`@Whitespace`: All whitespace characters are allowed


### Lists, Sets, Arrays and Stream

`@Size(int value = 0, int min = 0, int max = 0)`: 
    Set either fixed size through value or configure the size range between min and max


### Integral Numbers (Integer etc.)

`@ByteRange(byte min = 0, byte max)`

`@ShortRange(short min = 0, short max)`

`@IntRange(int min = 0, int max)`

`@LongRange(long min = 0L, long max)`

`@BigRange(String min = "", String max = "")`

`@Positive`: Numbers larger than 0

`@Negative`: Numbers lower than -0


### Decimal Numbers (Float, Double, BigDecimal)

`@FloatRange(float min = 0.0f, float max)`

`@DoubleRange(double min = 0.0, double max)`

`@BigRange(String min = "", String max = "")`

`@Scale(int value)`: Specify the maximum number of decimal places

`@Positive`: Numbers larger than 0.0

`@Negative`: Numbers lower than -0.0


## Programmatic Data Generation

```java
class FizzBuzzTests {
  @Property
  @Label("numbers divisible by 3 return Fizz")
  boolean divisibleBy3(@ForAll("divisibleBy3") int i) {
    return fizzBuzz().get(i - 1).startsWith("Fizz");
  }
  @Provide
  Arbitrary<Integer> divisibleBy3() {
    return Arbitraries
      .integers().between(1, 100)
      .filter(i -> i % 3 == 0);
  }
  @Provide
  Arbitrary<Integer> divisibleBy5() {
    return Arbitraries
      .integers().between(1, 20)
      .map(i -> i * 5);
  }
  private List<String> fizzBuzz() { … }
}
```

## Important Methods in Arbitraries class

### Generate selection of values

`<T> Arbitrary<T> of(T... values)`

`Arbitrary<Character> of(char[] values)`

`<T extends Enum> Arbitrary<T> of(Class<T> enumClass)`


### Numbers and Strings

`IntegerArbitrary integers()`

`LongArbitrary longs()`

`BigIntegerArbitrary bigIntegers()`

`DoubleArbitrary doubles()`

`BigDecimalArbitrary bigDecimals()`

`StringArbitrary strings()`


### Choose among Arbitraries

`<T> Arbitrary<T> oneOf(Arbitrary<T> first, Arbitrary<T>... rest)`


## Important Methods of all Arbitrary types

### Filter

`Arbitrary<T> filter(Predicate<T> filterPredicate)`

### Map

`<U> Arbitrary<U> map(Function<T, U> mapper)`


### Flat Map

`<U> Arbitrary<U> flatMap(Function<T, Arbitrary<U>> mapper)`


### Generate Collections

`Arbitrary<List<T>> list()`

`Arbitrary<Set<T>> set()`

`Arbitrary<Stream<T>> stream()`

`<A> Arbitrary<A> array(Class<A> arrayClass)`


### Generate Nulls

`Arbitrary<T> injectNull(double nullProbability)`


### Generate Optionals

`Arbitrary<Optional<T>> optional()`


## Combine Arbitraries

```java
@Property
boolean anyValidPersonHasAFullName(@ForAll Person aPerson) {
  return aPerson.fullName().length() >= 5;
}

@Provide
Arbitrary<Person> validPerson() {
  Arbitrary<String> firstName =
    Arbitraries.strings()
      .withCharRange('a', 'z')
      .ofMinLength(2).ofMaxLength(10);

  Arbitrary<String> lastName =
    Arbitraries.strings()
	  .withCharRange('a', 'z')
	  .ofMinLength(2).ofMaxLength(20);

  return Combinators.combine(firstName, lastName).as(Person::new);
}

class Person {
  Person(String firstName, String lastName) { … }
}
```
