# Scripting XML documents using Epsilon
 
In this article we demonstrate how you can create, query and modify plain standalone XML documents (i.e. no XSD/DTD needed)  in Epsilon programs using the  PlainXML driver added in version 0.8.9. All the examples in this article demonstrate using EOL to script XML documents. However, it's worth stressing that XML documents are supported throughout Epsilon. Therefore, you can use Epsilon to (cross-)validate, transform (to other models - XML or EMF-based -, or to text), compare and merge your XML documents.

<iframe width="90%" height="494" src="https://www.youtube.com/embed/GV1Wyx4SiQQ" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Querying an XML document
 
We use the following `library.xml` as a base for demonstrating the EOL syntax for querying XML documents.

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<library>
	<book title="EMF Eclipse Modeling Framework" pages="744">
		<author>Dave Steinberg</author>
		<author>Frank Budinsky</author>
		<author>Marcelo Paternostro</author>
		<author>Ed Merks</author>
		<published>2009</published>
	</book>
	<book title="Eclipse Modeling Project: A Domain-Specific Language (DSL) Toolkit" pages="736">
		<author>Richard Gronback</author>
		<published>2009</published>
	</book>
	<book title="Official Eclipse 3.0 FAQs" pages="432">
		<author>John Arthorne</author>
		<author>Chris Laffra</author>
		<published>2004</published>
	</book>
</library>
```

## Querying/modifying XML documents in EOL
 
The PlainXML driver uses predefined naming conventions to allow developers to programmatically access and modify XML documents in a concise way. This section outlines the supported conventions in the form of questions and answers followed by relevant examples.

### How can I access elements by tag name? 
 
The `t_` prefix in front of the name of the tag is used to represent a type, instances of which are all the elements with that tag. For instance, `t_book.all` can be used to get all elements tagged as `<book>` in the document, `t_author.all` to retrieve all `<author>` elements etc. Also, if `b` is an element with a `<book>` tag, then `b.isTypeOf(t_book)` shall return true.

```eol
// Get all <book> elements
var books = t_book.all;

// Get a random book
var b = books.random();

// Check if b is a book
// Prints 'true'
b.isTypeOf(t_book).println();

// Check if b is a library
// Prints 'false'
b.isTypeOf(t_library).println();
```

### How can I get the tag name of an element?
 
You can use the `.tagName` property for this purpose. For instance, if `b` is an element tagged as `<book>`, `b.tagName` shall return `book`. The `tagName` property is read-only.

```eol
// Get a random <book> element
var b = t_book.all.random();

// Print its tag
// Prints 'book'
b.tagName.println();
```

### How can I get/set the attributes of an element?
 
You can use the attribute name as a property of the element object, prefixed by `a_`. For example, if `b` is the first book of `library.xml`, `b.a_title` will return `EMF Eclipse Modeling Framework`. Attribute properties are read/write.

In this example, `b.a_pages` will return `744` as a string. For `744` to be returned as an integer instead, the `i_` prefix should be used instead (i.e. `b.i_pages`. The driver also supports the following preffixes: `b_` for boolean, `s_` for string (alias of `a_`) and `r_` for real values. 

```eol
// Print all the titles of the books in the library
for (b in t_book.all) {
	b.a_title.println();
}

// Print the total number of pages of all books
var total = 0;
for (b in t_book.all) {
	total = total + b.i_pages;
}
total.print();

// ... the same using collect() and sum() 
// instead of a for loop
t_book.all.collect(b|b.i_pages).sum();
```

### How can I get/set the text of an element?
 
You can use the `.text` read-write property for this.

```eol
for (author in t_author.all) {
	author.text.println();
}
```

### How do I get the parent of an element?
 
You can use the `.parentNode` read-only property for this.

```eol
// Get a random book
var b = t_book.all.random();

// Print the tag of its parent node
// Prints 'library'
b.parentNode.tagName.println();
```

### How do I get the children of an element?
 
You can use the `.children` read-only property for this.

```eol
// Get the <library> element
var lib = t_library.all.first();

// Iterate through its children
for (b in lib.children) {
	// Print the title of each child
	b.a_title.println();
}
```

### How do I get child elements with a specific tag name?
 
Using what you've learned so far, you can do this using a combination of the `.children` property and the select/selectOne() operations. However, the driver also supports `e_` and `c_`-prefixed shorthand properties for accessing one or a collection of elements with the specified name respectively. `e_` and `c_` properties are read-only.

```eol
// Get a random book
var b = t_book.all.random();

// Get its <author> children using the 
// .children property
var authors = b.children.select(a|a.tagName = "author");

// Do the same using the shorthand
authors = b.c_author;

// Get its <published> child and print
// its text using the
// .children property
b.children.selectOne(p|p.tagName = "published").text.println();

// Do the same using the shorthand
// (e_ instead of c_ this time as 
// we only want one element, 
// not a collection of them)
b.e_published.text.println();
```

### How do I create an element?
 
You can use the `new` operator for this. 

```eol
// Check how many <books> are in the library
// Prints '3'
t_book.all.size().println();

// Creates a new book element
var b = new t_book;

// Check again
// Prints '4'
t_book.all.size().println();
```

### How can I add a child to an existing element?
 
You can use the `.appendChild(child)` operation for this.

```eol
// Create a new book
var b = new t_book;

// Get the library element
var lib = t_library.all.first();

// Add the book to the library
lib.appendChild(b);
```

### How can I set the root element of an XML document?
 
You can use the `.root` property for this.

```eol
XMLDoc.root = new t_library;
```

## Adding an XML document to your launch configuration
 
To add an XML document to your Epsilon launch configuration, you need to select "Plain XML document" from the list of available model types.

![](select.png)

Then you can configure the details of your document (name, file etc.) in the screen that pops up. To load an XML document that is not in the Eclipse workspace, untick the "Workspace file" check box and provide a full uri for your document (e.g. `http://api.twitter.com/1/statuses/followers/epsilonews.xml` or `file:/c:/myxml.xml`). 

![](configure.png)

## Loading an XML document in your ANT buildfile
 
The following ANT build file demonstrates how you can use ANT to load/store and process XML documents with Epsilon.

```xml
<project default="main">
	<target name="main">
		
		<epsilon.xml.loadModel name="XMLDoc" file="library.xml"
			read="true" store="false"/>
		</epsilon.xml.loadModel>
		
		<epsilon.eol src="my.eol">
			<model ref="XMLDoc"/>
		</epsilon.eol>
		
	</target>
</project>
```
 
## Loading an XML document through Java code
 
The following excerpt demonstrates using XML models using Epsilon's Java API.

```java
EolModule module = new EolModule();
module.parse(new File("..."));

PlainXmlModel model = new PlainXmlModel();
model.setName("M");
model.setFile(new File("..."));
model.load();

module.getContext().getModelRepository().addModel(model);
module.getContext().setModule(module);
module.execute();
```
 
## Additional resources
 
* [http://java.sun.com/javase/6/docs/api/org/w3c/dom/Element.html](http://java.sun.com/javase/6/docs/api/org/w3c/dom/Element.html): Complete list of the operations that are applicable to XML elements