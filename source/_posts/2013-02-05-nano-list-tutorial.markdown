---
layout: post
title: "nano list tutorial"
date: 2013-02-05 21:12
comments: true
categories: [nano]
keywords: nano binding famework, xml json data binding, jaxb, android
description: show how nano framework handle list binding
---

In this tutorial, I will show you how to marshall and unmarshall list of objects using Nano binding framework.

<!--more-->

###1. Create Bean Class
Create a new Book class like below:

{% codeblock lang:java %}

package com.leansoft.nano.sample;

import com.leansoft.nano.annotation.Element;
import com.leansoft.nano.annotation.RootElement;

@RootElement(name = "book")
public class Book {

	@Element
	private String name;

	@Element
	private String author;

	@Element
	private String publisher;

	@Element
	private String isbn;
	
	public String getName() {
	    return name;
	}
	
	public void setName(String name) {
	    this.name = name;
	}
	
	public String getAuthor() {
	    return author;
	}
	
	public void setAuthor(String author) {
	    this.author = author;
	}
	
	public String getPublisher() {
	    return publisher;
	}
	
	public void setPublisher(String publisher) {
	    this.publisher = publisher;
	}
	
	public String getIsbn() {
	    return isbn;
	}
	
	public void setIsbn(String isbn) {
	    this.isbn = isbn;
	}
	
	@Override
	public String toString() {
	    return "Book [name=" + name + ", author=" + author + ", publisher="
	            + publisher + ", isbn=" + isbn  + "]";
	}
}

{% endcodeblock %}


This is simple bean class containing Nano annotations, indicating this is a Nano bindable class. When a top level class is annotated with @RootElement, then its instance maps to XML element, in our case &lt;book&gt; tag. At field level, all fields of Book are annotated with @Element, indicating these fileds map to XML elements.

###2. Create Container Class to Hold List of Objects
Now we need to create a new Class "Books.java" as container to hold the list of Book objects by having an ArrayList instance in our class:
{% codeblock lang:java %}

package com.leansoft.nano.sample;

import java.util.ArrayList;
import java.util.List;

import com.leansoft.nano.annotation.Element;

public class Books {
	 
    @Element(name = "book")
    private List<Book> books = new ArrayList<Book>();
 
    public Books() {}
 
    public Books(List<Book> books) {
        this.books = books;
    }
 
    public List<Book> getBooks() {
        return books;
    }
 
    public void setBooks(List<Book> books) {
        this.books = books;
    }
}


{% endcodeblock %}

Just annotate the books field with @Element annotation, then Nano framework will handle list marshalling or unmarshalling automatically for us.


###3. Create a Java Main Client
{% codeblock lang:java %}

package com.leansoft.nano.sample;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.util.ArrayList;
import java.util.List;

import com.leansoft.nano.IReader;
import com.leansoft.nano.IWriter;
import com.leansoft.nano.NanoFactory;

/**
 * A demo show Nano list handling
 * 
 * @author bulldog
 *
 */
public class ListExample {

	public static void main(String[] args) {
		Book bookOne = new Book();
		bookOne.setAuthor("Kathy Sierra");
		bookOne.setName("SCJP");
		bookOne.setPublisher("Tata McGraw Hill");
		bookOne.setIsbn("856-545456736");
		
		Book bookTwo = new Book();
		bookTwo.setAuthor("Christian Bauer");
		bookTwo.setName("Java Persistence with Hibernate");
		bookTwo.setPublisher("Manning");
		bookTwo.setIsbn("978-3832180577");
		
		List<Book> bookList = new ArrayList<Book>();
		bookList.add(bookOne);
		bookList.add(bookTwo);
		
		Books books = new Books();
		books.setBooks(bookList);
		
		IWriter xmlWriter = NanoFactory.getXMLWriter();
		try {
			xmlWriter.write(books, new FileOutputStream("books.xml"));
		} catch (Exception e) {
			e.printStackTrace();
		}
		
		IReader xmlReader = NanoFactory.getXMLReader();
		try {
			books = xmlReader.read(Books.class, new FileInputStream("books.xml"));
		} catch (Exception e) {
			e.printStackTrace();
		} 
		
		System.out.println(books.getBooks());
        
	}

}

{% endcodeblock %}

>1. In the main method, we create two Book objects and store them in an ArrayList, then we create a Books container object and put the book list into it.  
2. We use a Nano xml writer instance to write the Books object to an xml file.
3. We use a Nano xml reader instance to read the xml file back into Books object.
4. Finally, we print the list of books(which will eventually call toString() on Book object).

###4. Output
{% codeblock %}
[Book [name=SCJP, author=Kathy Sierra, publisher=Tata McGraw Hill, isbn=856-545456736], Book [name=Java Persistence with Hibernate, author=Christian Bauer, publisher=Manning, isbn=978-3832180577]]
{% endcodeblock %}

Refesh your project in IDE(eclipse in my case) to see the generated XML file:
{% codeblock %}

<?xml version='1.0' encoding='utf-8' ?>
<books>
  <book>
    <author>Kathy Sierra</author>
    <isbn>856-545456736</isbn>
    <name>SCJP</name>
    <publisher>Tata McGraw Hill</publisher>
  </book>
  <book>
    <author>Christian Bauer</author>
    <isbn>978-3832180577</isbn>
    <name>Java Persistence with Hibernate</name>
    <publisher>Manning</publisher>
  </book>
</books>

{% endcodeblock %}

###5. Project Source
You can get the whole source of this project [here](https://github.com/bulldog2011/nano/tree/master/sample/nanolist).

