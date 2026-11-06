#advanced_programming #computer_science

[[AP index]] (3)

<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">12/11/2025</span></p>
# Marshalling and serialization

At runtime, data exists in memory with a representation intimate to the running process. An outside viewer has no knowledge of how it is organized, which means that as soon as you need to move that data, this representation is useless. **Serialization** is the act of transforming in-memory representation of data into a suitable format for transmission and storage, without any assumption about any particular memory layout. Instead, the reverse act, i.e. transforming serialized data into a specific format suitable for our process memory layout, is called **deserialization**.

Serialization is a step in the <span style="color:rgb(255, 0, 0)"><b>marshalling</b></span> process, which consists in transferring an object between runtimes. It is slightly different from serialization, as this process also encompasses the intent of serialization. Not every object can be meaningfully serialized; for instance, pointers or file descriptor are context-specific of the process they are used in, meaning that when passed they would lose all significance. Furthermore, most serialization format prohibit the encoding of functional value, since they could lead to the execution of malignant code if the deserialization process can invoke functions (like constructors).

# Mark-up languages

A <span style="color:rgb(255, 0, 0)"><b>mark-up language</b></span> is a formal syntax for annotating data with meta-information. It provides only the syntax, the semantics depend on the process consuming it. When structured data needed to be read, every application had to build its own parser (more information about parsing in [[Syntax]]). With the introduction of GML (Generalized Mark-up Language) and later SGML (Standard GML), text was equipped with meta-information indicating, for example, which part was the heading, which a paragraph, a section, and so on. SGML also allowed users to define their own elements, which allowed it to be more general but ended up being overwhelming.
With the need to share hypertext documents, HTML (Hyper Text Mark-up Language) was designed, which was a descendent of SGML, but with a predetermined vocabulary of element names; e.g. `<h1>`, `<p>` or `<img>`. It was designed to display text, which is why it was never used as a serialization format.
Then, XML (eXtensible Mark-up Language) was developed. XML is a cleaned-up and simplified version of SGML that retained user-defined element names, while discarding its complexity. For this reason, XML became the dominant serialization format for many years.
The main criticism with XML regards its verbosity, which, with the proliferation of web APIs, caused too much overhead when read. This led to the development of lightweight formats like JSON (JavaScript Object Notation) and YAML (Yet Another Mark-up Language). The former uses JavaScript object literal syntax, removing tags entirely, while the latter replaced syntactic delimiters with indentation.
$$\texttt{GML/SGML} \rightarrow \texttt{HTML} \rightarrow \texttt{XML} \rightarrow \texttt{JSON/YAML}$$
Here is a SGML example:
```SGML
<!DOCTYPE person [
    <!ELEMENT person - - (name, age, active, skills)>
    <!ELEMENT name - - (#PCDATA)>
    <!ELEMENT age - - (#PCDATA)>
    <!ELEMENT active - - (#PCDATA)>
    <!ELEMENT skills - - (skill*)>
    <!ELEMENT skill - - (#PCDATA)>
]>

<person>
    <name>Alice</name>
    <age>25</age>
    <active>true</active>
    <skills>
        <skill>Python</skill>
        <skill>Java</skill>
    </skills>
</person>
```
It is important to note that SGML includes a prelude with a DTD (Document Type Definition), which is what defines the permitted document structure.

Here is the same example but in HTML:
```HTML
<!DOCTYPE html>
<html>
<head>
    <title>Person</title>
</head>
<body>
    <h1>Alice</h1>
    <p>Age: 25</p>
    <p>Active: true</p>
    <h2>Skills</h2>
    <ul>
        <li>Python</li>
        <li>Java</li>
    </ul>
</body>
</html>
```
Clearly, HTML is not suited for encoding data other than text.

## XML

An XML document is a text file made up of a small set of constructs, like elements and attributes. An XML document is said to be well-formed if it respects the XML grammar; however, it is said to be valid if it conforms to a schema (like a DTD). Well-formedness can be checked by any XML parser, but a document can be said to be valid only with respect to a certain schema.

**Elements** are the primary structural unit, and it consists of an opening tag, optionally followed by content, and then they must be followed by a closing tag. **Attributes** are name-value pairs that can appear inside an element's opening tag. **Character data** is the textual content between tags. **Entities** are references that start with `&`and end with `;`, and expand to predefined strings. It is important to note that the restriction that every element must be closed causes the existence of some well-formed HTML documents to not be XML well-formed.

Respecting the syntax given by these constructs guarantees well-formedness. However, for serialization to be useful between systems, they must agree also on the order and content each element may contain. For this reason, XML uses a DTD document similar to that of SGML; here is the example we already used previously but in XML:
```XML
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE person [
    <!ELEMENT person (name, age, active, skills)>
    <!ELEMENT name (#PCDATA)>
    <!ELEMENT age (#PCDATA)>
    <!ELEMENT active (#PCDATA)>
    <!ELEMENT skills (skill*)>
    <!ELEMENT skill (#PCDATA)>
]>

<person>
    <name>Alice</name>
    <age>25</age>
    <active>true</active>
    <skills>
        <skill>Python</skill>
        <skill>Java</skill>
    </skills>
</person>
```
We can see it being fairly similar to SGML.

XSD (XML Schema Definition) extends the structural grammar of DTD with a full type system (primitive types and complex types).

## JSON

JSON is a data interchange format whose syntax is a subset of JavaScript object literal notation. Here is the previous example written in JSON:
```JSON
{
    "person": {
        "name": "Alice",
        "age": 25,
        "active": true,
        "skills": [
            "Python",
            "Java"
        ]
    }
}
```
Curly brackets (`{` and `}`) delimit objects, each having fields with associated values. It is important to note that other objects can be values, and also arrays are expressible using `[]`.

The rapid popularity of JSON is mainly due to the fact that it could be parsed using JavaScript inner `eval` function. This is also the reason why JSON is a *subset* of JavaScript object notation; if `eval` was used indiscriminately, it would be possible to pass as JSON malignant code, which is also why functional values are not allowed.

Similarly to XSD or DTD for XML, JSON also has JSON Schema, although it is not part of the official JSON specification. JSON Schema is a vocabulary for annotating and validating JSON documents, and it is not a JSON object.

## YAML

YAML is a data serialization language that uses indentation rather than syntactic delimiters like `{}` or tags. Here is the previous example written in YAML:
```YAML
person:
	name: Alice
	age: 25
	active: true
	skills:
	- Python
	- Java
```

YAML is generally easier to read than JSON; however, the fact that indentation plays such a significant role in understanding the data it represents, makes it quite susceptible to errors in big datasets.

Generally speaking, XML is better for complex data with a complicated hierarchical structure that requires strict validation. JSON is better for operations that need a lightweight serialization and do not require complex structures, like APIs and web application.  YAML is better for data mainly maintained by humans, like configuration files.