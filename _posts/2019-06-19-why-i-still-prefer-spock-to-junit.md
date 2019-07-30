---
layout: post
title: Why I prefer Spock to Junit
date: 2019-06-12 00:00 +0200
category: personal
tags: fun
---

It's been two years now that Junit 5 was released, it's has been rewritten from scratch and bring many improvements, among them:
- Lambda expression
- Better supports for Parametrized tests
- Custom display name
- Nested tests classes
- Dependecy injection
- Tagging
- ...

But even with all this new stuff, i still believe that the Spockframework is better for writing more expressive and readable tests by using the BDD style structure.

Let's take some real example of tests writen in Junit and refactor theme to Spockframework, and do the comparison.

### Example 1
Let's take a simple test from the [Apache Camel](https://github.com/apache/camel/blob/master/core/camel-core/src/test/java/org/apache/camel/support/jsse/SecureRandomParametersTest.java) project. it consists on testing a creation of a SecureRandom instance for a given algorithm. 

{% highlight java %}
@Test
public class SecureRandomParametersTest extends AbstractJsseParametersTest {
    ...
    public void testCreateSecureRandom() throws Exception {    
        if (this.canTest()) {
            SecureRandomParameters srp = new SecureRandomParameters();
            srp.setAlgorithm("SHA1PRNG");
            
            SecureRandom sr = srp.createSecureRandom();
            assertEquals("SHA1PRNG", sr.getAlgorithm());
            
            String providerName = sr.getProvider().getName();
            srp.setProvider(providerName);
            
            sr = srp.createSecureRandom();
            assertEquals("SHA1PRNG", sr.getAlgorithm());
            assertEquals(providerName, sr.getProvider().getName());
        }
    }
    
    protected boolean canTest() {
        try {
            SecureRandom.getInstance("SHA1PRNG");
            return true;
        } catch (NoSuchAlgorithmException e) {
            return false;
        }
    }
    ...
}
{% endhighlight %}
This test seems to be executed only if the method _canTest_ returns true, otherwise it will be not executed. This is a code smell, because if the method _canTest_ returns false, the code inside it will be not executed, and the test will pass success.

Anyway this test still be easy to understand what we expect from it.

Okey let's rewrite this test with Spock.

{% highlight groovy %}
class SecureRandomParametersSpec extends Specification {
    static def SHA1PRNG = {
        try {
            SecureRandom.getInstance("SHA1PRNG");
            return true
        } catch (NoSuchAlgorithmException ex) {
            return false
        }
    }

    @Requires({ SecureRandomParametersSpec.SHA1PRNG() })
    def "create SecureRandom with SHA1PRNG algorithm"() {
        given:
        def srp = new SecureRandomParameters()
        srp.algorithm = "SHA1PRNG"

        when:
        SecureRandom sr = srp.createSecureRandom()

        then:
        sr.algorithm == "SHA1PRNG"

        when:
        String providerName = sr.getProvider().getName()
        srp.provider = providerName
        sr = srp.createSecureRandom()

        then:
        sr.algorithm == "SHA1PRNG"
        sr.getProvider().getName() == providerName
    }
}
{% endhighlight %}
The first improvement with the _Spock Framework_ is the condition method which is pulled out to the _@Requires_ annotation, and will be executed only on certain condition (in this case is the supporting of the _SHA1PRNG_ by the System). Therefore this test will be not executed and it will be reported as skipped by reporting tools (Surefire plugin).
We can achieve the same things in Junit5 with the [ExecutionCondition](https://junit.org/junit5/docs/5.0.3/api/org/junit/jupiter/api/extension/ExecutionCondition.html) extension interface.


The second improvement is the structure of the test. In _Spock Framework_, a test is designed around blocks such _given, when, then_. With this blocks, we achieve a clean separation of concernes in a test.    

### Example 2
let's look at a more complexe test from the [jodd](https://github.com/oblac/jodd) project.

{% highlight java %}
class AsmUtilTest {
    @ParameterizedTest
    @MethodSource(value = "testdata_testTypeToSignature_Class")
    void testTypeToSignature_Class(final String expected, final Class input) {
        assertEquals(expected, AsmUtil.typeToSignature(input));
    }

    private static List<Arguments> testdata_testTypeToSignature_Class() {
        final List<Arguments> params = new ArrayList<>();

        // primitives
        params.add(Arguments.of("int", int.class));
        params.add(Arguments.of("long", long.class));
        params.add(Arguments.of("boolean", boolean.class));
        params.add(Arguments.of("double", double.class));
        params.add(Arguments.of("float", float.class));
        params.add(Arguments.of("short", short.class));
        params.add(Arguments.of("void", void.class));
        params.add(Arguments.of("byte", byte.class));
        params.add(Arguments.of("char", char.class));
        // non primitives
        params.add(Arguments.of("java/lang/String", String.class));
        params.add(Arguments.of("jodd/asm/AsmUtil", AsmUtil.class));

        return params;
    }
}
{% endhighlight %}
This code tests conversion of some primtive or Class types to there corresonding signature. This code uses Parameterized Test for testing a same logic with different input and output data.

The _SpockFramework_ have a more convenient way for this kind of tests. The input and output data are set in a Data Table like format. The code below is the refactored Spock version.

{% highlight groovy %}
class AsmUtilSpec extends Specification {

  @Unroll
  def "Converts java-class name '#input' to bytecode-name should be '#expectedSignature'"(input, expectedSignature) {
    when:
    def signature = AsmUtil.typeToSignature(input)

    then:
    signature == expectedSignature

    where:
    input     ||  expectedSignature
    int       || 'int'
    long      || 'long'
    boolean   || 'boolean'
    double    || 'double'
    float     || 'float'
    short     || 'short'
    void      || 'void'
    byte      || 'byte'
    char      || 'char'
    String    || 'java/lang/String'
    AsmUtil   || 'jodd/asm/AsmUtil'
  }
}
{% endhighlight %}

### Example 3
The underlying example is from the [Zookeeper Project](https://github.com/apache/zookeeper/blob/master/zookeeper-server/src/test/java/org/apache/zookeeper/server/util/SerializeUtilsTest.java). 

{% highlight java %}
@Test
public void testSerializeRequestWithTxn() throws IOException {
// Arrange
TxnHeader header = mock(TxnHeader.class);
    doAnswer(new Answer() {
        @Override
        public Object answer(InvocationOnMock invocation) throws Throwable {
        Object[] args = invocation.getArguments();
        OutputArchive oa = (OutputArchive) args[0];
        oa.writeString("header", "test");
        return null;
        }
    }).when(header).serialize(any(OutputArchive.class), anyString());
    Record txn = mock(Record.class);
    doAnswer(new Answer() {
        @Override
        public Object answer(InvocationOnMock invocation) throws Throwable {
        Object[] args = invocation.getArguments();
        OutputArchive oa = (OutputArchive) args[0];
        oa.writeString("record", "test");
        return null;
        }
    }).when(txn).serialize(any(OutputArchive.class), anyString());
    Request request = new Request(1, 2, 3, header, txn, 4);

    // Act
    byte[] data = SerializeUtils.serializeRequest(request);

    // Assert
    Assertions.assertNotNull(data);
    InOrder inOrder = inOrder(header, txn);
    inOrder.verify(header).serialize(any(OutputArchive.class), eq("hdr"));
    inOrder.verify(txn).serialize(any(OutputArchive.class), eq("txn"));
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    BinaryOutputArchive boa = BinaryOutputArchive.getArchive(baos);
    boa.writeString("header", "test");
    boa.writeString("record", "test");
    baos.close();
    assertArrayEquals(baos.toByteArray(), data);
}
{% endhighlight %}

Basically, this test check the behaviour of Serializing Request. it checks the interaction with Two collaboratos (_TxnHeader_ and _Record_) and expect the created request contains a header and a body.
Let's refactore this test in Spock way:

{% highlight groovy %}
def "serialize Request with Transaction"() {
    given:
    TxnHeader header = Mock()
    Record record = Mock()
    Request request = new Request(1, 2, 3, header, record, 4)

    when: "Serializing a request"
    byte[] data = SerializeUtils.serializeRequest(request)

    then: "it should call header serialize"
    1 * header.serialize(_, "hdr") >> { outputArchive, tag ->
        outputArchive.writeString("header", "test")
    }

    then: "it should call record serialize"
    1 * record.serialize(_, "txn") >> { outputArchive, tag ->
        outputArchive.writeString("record", "test")
    }

    and: "serialized request should contain a header and a record"
    ByteArrayOutputStream baos = new ByteArrayOutputStream()
    BinaryOutputArchive boa = BinaryOutputArchive.getArchive(baos)
    boa.writeString("header", "test")
    boa.writeString("record", "test")
    baos.close()
    baos.toByteArray() == data
}
{% endhighlight %}
The previous code showed how mocks are handled by using closure, which i consider much powerful even than Java 8 Lambda. In Junit we still need a _Mock Framework_ for mocking classes and interfaces, but in _Spock_, we have already a built-in one. 
Also with Groovy language our tests are less verbose and more readable.


### Conclusion
IMO _SpockFramework_ still the best testing framework for JVM languages like _Java_ or _Kotlin_. It allows writing simple, readable and concise tests, even in complex tests.