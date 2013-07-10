## Introduction

This extension provides a simple DBUnit-Extension that enables you to define your testdata in simple Java methods instead of separate data files.

This library depends on spring-test-dbunit, a nice project that integrates spring test framework an DBUnit. You can find more information about spring-test-dbunit here: https://github.com/springtestdbunit/spring-test-dbunit

What is the need for it?

- you can define your test data in java and let DBUnit insert the data into the database. You can still use all DBUnit-features (e.g. @DatabaseSetup and @ExpectedDatabase)
- when you have a lot of tests that need lots of test data, you do not need to create that many test data files. lots of data files mean good naming conventions etc.
- you can generate test your way
- you can implement all logic you need to provide test data. You can for example read test data from a webservice and use this for your tests.
- you can reuse testdata when you extract the data generation into a separate java class. 

But: you can still define your test data in separate files. When you need lots of data that cannot be generated, XML oder CVS could still be a better solution than a Java method. 

## Using DB-Unit with springtestdbunit

Let's start with a "normal" DB-Unit-Test. First, you create a Testclass like this:

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(locations =  { "classpath:application-context.xml" })
    @TestExecutionListeners({ DependencyInjectionTestExecutionListener.class,
       DirtiesContextTestExecutionListener.class,
       TransactionalTestExecutionListener.class,
       DbUnitTestExecutionListener.class })
    public class ContactDataServiceIntegrationTest {


    }

With the annotation @RunWith you invoke the testcase with SpringJUnit4ClassRunner. This is a default runner of the Spring-Framework. To integrate DBUnit, I use DBUnitTestExecutionLister as one of the TestExecutionListeners. DBUnitTestExecutionLister ist a class of spring-test-dbunit. 

Next, you write a test method:

    @Test
    @DatabaseSetup("classpath:testdata/ContactDatabaseService_testSelect.xml")
    public void testSelect_NOK(){
      Contact c = service.getContactByFirstname("Birgit");
      Assert.assertNull(c);
    }

With the Annotation @DatabaseSetup you tell DBUnit where to find the test data. DBUnit will read the named file and insert the data into the database before the test is executed. The file ContactDatabaseService_testSelect.xml looks like this:

    <?xml version="1.0" encoding="UTF-8"?>
    <dataset>
      <CONTACT ID="1" FIRST_NAME="Pina" LAST_NAME="Schneider"/>
      <CONTACT ID="2" FIRST_NAME="Carsten" LAST_NAME="Schneider"/>
    </dataset>

## Using DBUnit with spring-test-dbunit and spring-test-dbunit-ext

This extension gives you the ability to define the test data directly in Java. First, you need to add the @DBUnitConfiguration-annotation at class-level:

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(locations =  { "classpath:application-context.xml" })
    @DbUnitConfiguration(dataSetLoader=MethodDataSetLoader.class)
    @TestExecutionListeners({ DependencyInjectionTestExecutionListener.class,
      DirtiesContextTestExecutionListener.class,
      TransactionalTestExecutionListener.class,
      DbUnitTestExecutionListener.class })
    public class ContactDataServiceIntegrationTest { 

With @DBUnitConfiguration you define MethodDataSetLoader as the default DataSetLoader for the test class. Now, DBUnit does not look for XML data files anymore. Instead of a filename, you have to define a method-name to tell DBUnit where to find the data:

    @DatabaseSetup("dataTestSelect")
    public void testSelect_NOK(){
      Contact c = service.getContactByFirstname("Birgit");
      Assert.assertNull(c);
    }	

Next you habe to write a <b>static method</b> within the testclass with the name <b>dataTestSelect</b>:

    public static DataSet dataTestSelect(){
      DataSet ds = new DataSet();
      ds.row("CONTACT").attr("ID", "1").attr("FIRST_NAME",  "Pina").attr("LAST_NAME", "Schneider");
      ds.row("CONTACT").attr("ID", "2").attr("FIRST_NAME",  "Carsten").attr("LAST_NAME", "Schneider");	
      return ds;
   }

Be aware! It is important to make the method static! As you can see, the data within the method stays the same as in the XML example above. The only difference is, that you define the data with a Java-API instead of XML.

## Mixing Data from XML and Java
Without MethodDataSetLoader, we had to define all the data in separate XML-Files. With the use of MethodDataSetLoader, we had to define all Data in Java-Methods. Is it possible to to use both ways in one Test-Class? To do this, I created a small class TypedDataSetLoader that gives us the ability to define the source-type of the data per Test-Method (method or XML). First, you have to replace <b>MethodDataSetLoader</b> with <b>TypesDataSetLoader</b>:

    @RunWith(SpringJUnit4ClassRunner.class)
    @ContextConfiguration(locations =  { "classpath:application-context.xml" })
    @DbUnitConfiguration(dataSetLoader=TypedDataSetLoader.class)
    @TestExecutionListeners({ DependencyInjectionTestExecutionListener.class,
      DirtiesContextTestExecutionListener.class,
      TransactionalTestExecutionListener.class,
      DbUnitTestExecutionListener.class })
    public class ContactDataServiceIntegrationTest {  

Now you can simply put a prefix in front of the parameter of <b>@DatabaseSetup</b>:

    @Test
    @DatabaseSetup("xml:classpath:testdata/ContactDatabaseService_testSelect.xml")
    public void testSelect_NOK(){
      Contact c = service.getContactByFirstname("Birgit");
      Assert.assertNull(c);
    }   

Or:

    @DatabaseSetup("method:dataTestSelect")
    public void testSelect_NOK(){
      Contact c = service.getContactByFirstname("Birgit");
      Assert.assertNull(c);
    }   


By now, allowed prefixes are:
- method:
- xml:

A better solution would be to extend the @DatabaseSetup-Annotation in order to set the source-type, but i did not want to extend any of the original classes of spring-test-dbunit.

Thats it. Very simple. To use this library in your gradle-Project, you need the following dependancies:

    testCompile 'com.github.springtestdbunit:spring-test-dbunit:1.0.0'
    testCompile 'com.github.springtestdbunitext:spring-test-dbunit-ext:1.0.0.RELEASE'

In this project you find the source-code as well as the complete runnable example.

