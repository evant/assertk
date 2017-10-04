**This project has moved to https://github.com/willowtreeapps/assertk**


# assertk
assertions for kotlin inspired by assertj

## Download

```groovy
repositories {
  maven {
    url "https://oss.sonatype.org/content/repositories/snapshots"
  }
}

dependencies {
  testCompile 'me.tatarka.assertk:assertk:1.0-SNAPSHOT'
}
```

## Usage

Simple usage is to wrap the value you are testing in `assert()` and call assertion methods on the result.

```kotlin
import me.tatarka.assertk.assert
import me.tatarka.assertk.assertions.*

class PersonTest {
    val person = Person(name = "Bob", age = 18)

    @Test
    fun testName() {
        assert(person.name).isEqualTo("Alice")
        // -> expected:<["Alice"]> but was:<["Bob"]>
    }

    @Test
    fun testAge() {
        assert("age", person.age).isGreaterThan(20)
        // -> expected [age] to be greater than:<20> but was:<10>
    }
}
```

### Nullability
Since null is a first-class concept in kotlin's type system, you need to be explicit in your assertions.

```kotlin
val nullString: String? = null
assert(nullString).hasLength(4)
```
will not compile, since `hasLength()` only makes sense on non-null values. You can use `isNotNull()` which takes an
optional lambda to handle this.

```kotlin
val nullString: String? = null
assert(nullString).isNotNull {
    it.hasLength(4)
}
// -> expected to not be null
```
This will first ensure the string is not null before running any other checks.

### Multiple assertions

You can assert multiple things on a single value by providing a lambda as the second argument. All assertions will be
run even if the first one fails.

```kotlin
val string = "Test"
assert(string) {
    it.startsWith("L")
    it.hasLength(3)
}
// -> The following 2 assertions failed:
//    - expected to start with:<"L"> but was:<"Test">
//    - expected to have length:<3> but was:<"Test"> (4)
```

You can wrap multiple assertions in an `assertAll` to ensure all of them get run, not just the first one.

```kotlin
assertAll {
    assert(false).isTrue()
    assert(true).isFalse()
}
// -> The following 2 assertions failed:
//    - expected to be true
//    - expected to be false
```

### Exceptions

If you expect an exception to be thrown, you have a couple of options:

The first is to wrap in a `catch` block to store the result, then assert on that.

```kotlin
val exception = catch { throw Exception("error") }
assert(exception).isNotNull {
    it.hasMessage("wrong")
}
// -> expected [message] to be:<["wrong"]> but was:<["error"]>
```

Your other option is to use an `assert` with a single lambda arg to capture the error.

```kotlin
assert {
    throw Exception("error")
}.throwsError {
    it.hasMessage("wrong")
}
// -> expected [message] to be:<["wrong"]> but was:<["error"]>
```

This method also allows you to assert on return values.
```kotlin
assert { 1 + 1 }.returnsValue {
    it.isNegative()
}
// -> expected to be negative but was:<2>
```

### Table Assertions

If you have multiple sets of values you want to test with, you can create a table assertion.

```kotlin
tableOf("a", "b", "result")
    .row(0, 0, 1)
    .row(1, 2, 4)
    .forAll { a, b, result ->
        assert(a + b).isEqualTo(result)
    }
// -> the following 2 assertions failed:
//    on row:(a=<0>,b=<0>,result=<1>)
//    - expected:<[1]> but was:<[0]>
//    on row:(a=<1>,b=<2>,result=<4>)
//    - expected:<[4]> but was:<[3]>
```

Up to 4 columns are supported.

## Custom Assertions

One of the goals of this library is to make custom assertions easy to make. All assertions are just extension methods.

```kotlin
fun Assert<Person>.hasAge(expected: Int) {
    assert("age", actual.age).isEqualTo(expected)
}

assert(person).hasAge(10)
// -> expected [age]:<10> but was:<18>
```

If you want to customize the message, you usually want to use `expected()` and `show()`.

```kotlin
fun Assert<Person>.hasAge(expected: Int) {
    if (actual.age == expected) return
    expected("age:${show(expected)} but was age:${show(actual.age)}")
}

assert(person).hasAge(10)
// -> expected age:<10> but was age:<18>
```
