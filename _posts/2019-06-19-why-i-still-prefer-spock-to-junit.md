---
layout: post
title: Why i still prefer Spock to Junit 5
date: 2019-06-12 00:00 +0200
category: personal
tags: fun
---

It's been two years now that Junit 5 was released, it's has been rewritten from scratch and has several improvements, among them:
- Lambda expression
- Better supports for Parametrized tests
- Custom display name
- Nested tests classes
- Dependecy injection

Spock leverages Java tests by using groovy under the hood.
Junit 5 redesigned and rewritten from scratch.

- PIT - mutation testing
- Maven for java surefire plugin, for spock GMavenPlus for Groovy

Let's compare this new feature with the SpockFramework


### Test Structure
Good tests should be separated in three sections
```java
// Arrange
// Act
// Assert
```

or in the BDD Terminolgy
```java
// Given
// When 
// Then
```
In Junit 5, there no way to separate this block in a compact way, it can be used just by plain code comments
```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

@Test
void adding_transaction_should_change_account_balance() {
// Given
Account account = new Account();

// When
account.addTransaction(200);

// Then
assertEquals(
    200, account.balance(), () -> "account balance should be 200"
);
}
```

But is Spock Framework this is seprated strictly:
```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

def "calculate diameter"() {
    given:
    Wheel wheel = new Wheel(26, 1.5);

    when:
    double diameter =  wheel.diameter()

    then:
    diameter == 29
}
```

There are some plugins that can reach this concept like [JBehave](https://jbehave.org/), but i still prefer the Spock way


Spock spport BDD Specification
Spock more structured tests
improved readability

### Exceptions
Junit 5 provide a new assertion to verify both exception and the associated message in a more compact syntax with lambda expressions. let's see an example.

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertThrows;

@Test
void cannot_create_account_controller_without_security_service() {
// When    
IllegalArgumentException thrown = assertThrows(
    IllegalArgumentException.class, () -> new AccountController(null)
);

// Then
assertEquals(
    "SecurityService is mandatory", thrown.getMessage()
);
}
```
In Spock 
```groovy
def "cannot create account controller without security service"() {
    when:
    new AccountController(null)

    then:
    IllegalArgumentException exception = thrown()
    exception.message == "SecurityService is mandatory" 
}
```

### Mock
In Junit 5 there is no built-in mocking framework, thanks to Mockito which is the defacto mocking framework.
Let's see an example:
```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.ArgumentMatchers.anyLong;
import static org.mockito.BDDMockito.given;
import static org.mockito.BDDMockito.then;
import static org.mockito.Mockito.mock;
import static org.mockito.Mockito.times;

@Test
void adding_transaction_to_an_account() {
// Given
var account = new Account(45);
given(accountRepository.findById(anyLong())).willReturn(Optional.of(account));

// When
accountService.addTransactionToAccount(45, 200);

// Then
then(accountRepository).should(times(1)).findById(anyLong());
assertEquals(200, account.balance());
}
```
let's see the same test in Spock
```groovy
def "adding transaction to an account"() {
given:
def account = new Account(45)

and:
accountRepository.findById(_) >> Optional.of(account)

when:
accountService.addTransactionToAccount(45, 200)

then:
1 * accountRepository.findById(_;

and:
account.balance() == 200
}
```
it's obvious that in spock way the test is more readable and concise.

### Reporting

### Naming tests

### Parametrized tests
- one test for various input data

### Groovy goodness

### Spring integration