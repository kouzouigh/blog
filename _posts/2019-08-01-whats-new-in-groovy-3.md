---
layout: post
title: What's new in Groovy 3
date: 2019-08-16 00:00 +0200
category: programming
tags: Tech
url: https://kouzouigh.github.io/blog/whats-new-in-groovy-3
---

![Logo of Groovy](https://upload.wikimedia.org/wikipedia/commons/3/36/Groovy-logo.svg)
In this article, we’ll have a quick look at some of the most interesting new features in Groovy 3.

### Identity Operator
In Groovy 2 to express _equal-identity_ we use the _is()_ expression and  _==_ and _!=_ to express just equality(through _equals()_ method). This created some confusion, hence in Groovy 3 introduce the operators _===_ and _!==_ for expressing respectively **identity equal** and **non identity equal**.

**Example**
{% highlight groovy %}
import groovy.transform.EqualsAndHashCode
import groovy.transform.TupleConstructor

@TupleConstructor
@EqualsAndHashCode
class Book { String isbn }

def effectiveJava = new Book(isbn: '0134685997')
def refToEffectiveJava = effectiveJava
def copyOfEffectiveJava = new Book(isbn: '0134685997') 

refToEffectiveJava === effectiveJava // return true
refToEffectiveJava.is(effectiveJava) // return true
copyOfEffectiveJava == effectiveJava // return true
copyOfEffectiveJava === effectiveJava // return false

copyOfEffectiveJava !== effectiveJava // return true
refToEffectiveJava !== effectiveJava  // return false
{% endhighlight %}

### Elvis assignment
In Java, the Elvis operator is used only in a ternary operation(it's function is equivalent to a
java method that tests a condition and depending of the outcome, returns a value)
```groovy
def variable = book.isbn == null ? '0000000000' : book.isbn
```
In the example above the variable take _0000000000_ if condition is success, otherwise _0134685997_

Groovy support a shortcut of this operation in providing a default value:
```groovy
def variable = book.isbn ?: '0000000000'
```

We can also use this operator for safe navigation between object in order to easily avoid null pointer exception
```groovy
def isbn = book?.isbn // return isbn value if book is not null
```

In Groovy 3 we can use this operator for safe indexing arrays. this allows to access an index of an array, but if
the array is null, it will return null instead of throwing a _java.lang.NullPointerException_.
```groovy
List<Book> books = null
println books?[1] // now return null
```

In Groovy 3 also added a new feature for this operator, it's called **Elvis assignement**
```groovy
def book = new Book('0134685997')
book ?= new Book('0000000000')
assert book.isbn == '0134685997'

book = null
book ?= new Book('0000000000')
assert book.isbn == '0000000000'
```
### Negative relational Operator
This is a little shorthand of the negation of the opertaors _instanceof_ and _in_
```groovy
// before Groovy 3
assert !('abc' instanceof Integer)
assert !(2 in [1, 3, 5])

// In Groovy 3
assert 'abc' !instanceof Integer
assert 2 !in [1, 3, 5]
```

### @NullCheck annotation
This new annotation automaticaly check all your method parameters, constructor parameters are not _null_

**Example**
```groovy
import groovy.transform.NullCheck
import static groovy.test.GroovyAssert.shouldFail

@NullCheck
class Book { 
    String isbn 
    Book(String isbn) {
        this.isbn = isbn
    }
    
    void write(OutputStream output) {
        println("Write book on an output stream")
    }
}

shouldFail(IllegalArgumentException) { new Book(null) }
shouldFail(IllegalArgumentException) { new Book('0134685997').write(null) }
```
### Java alignments
Before Groovy 3, some java syntaxes are not supported with Groovy like:

**do/while loop**
{% highlight groovy %}
int count = 0;
do {
  count++
} while (count < 10)

assert count == 10
{% endhighlight %}

**Lambda expressions**
{% highlight groovy %}
assert 9 == [1, 2, 3].stream()
                     .map(n -> n + 1)
                     .reduce(0, (total, n) -> total + n)
{% endhighlight %}
**Method reference**
{% highlight groovy %}
assert ['1', '2', '3'] == [1, 2, 3].stream()
                                   .map(Integer::toString)
                                   .collect(toList())
{% endhighlight %}                                   

**try-with-resources**
{% highlight groovy %}
class Resource implements Closeable {
 
 def logs
 
 void execute() {
    logs << 'Executing Resource'
 }
     
 void close() {
    logs << 'Closing Resource'
 }   
}

def logs = []

try (def resource = new Resource(logs: logs)) {
    resource.execute()
}

assert logs == ['Executing Resource', 'Closing Resource']

// but may be, it's better in Groovy with the extension method withCloseable()
logs = []
new Resource(logs: logs).withCloseable {
    it.execute()
}

assert logs == ['Executing Resource', 'Closing Resource']
{% endhighlight %}

**default method interface**
{% highlight groovy %}
// Note: In Groovy this feature is already implemented with traits
interface Cache {
    
    default void evict() {
        println "Evict cache"
    }
}
{% endhighlight %}