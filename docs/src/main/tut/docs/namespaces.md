---
layout: docs
title:  "Namespaces"
position: 4
---

## Namespaces

As long as you don't care about namespaces in your XML you should use plain Strings when selecting for elements
or attributes. As a consequence your code may look simple as that: `root \ "c1"`, where plain string `c1` indicates you
want to query for `c1` labeled elements. In some cases such approach might be OK. That will be if your XML contains 
elements from just one namespace or there's no name clash between elements defined in various 
namespaces. In other cases you should consider using full qualified names instead of local names.

In all other sections of the documentation namespaces are ignored and therefore plain Strings are used. Mind that in case
you want to make your code namespace sensitive you need to replace those Strings with constructs described in this section.

### Prefixes vs namespace URLs

Due to the fact the the same prefixes may refer to different namespaces, `xml-lens` does not to rely on them. Instead, 
it relies on namespace's URL. Thus, to create a namespace it suffices to:

```tut:book
import net.michalsitko.xml.optics.Namespace

val a = Namespace("http://a.com")
```

And now to create fully qualified name within namespace created above you need to:

```tut:book
a.name("elementLabel")
```

### Simple example

To see working with namespace in action, let's consider following XML:

```tut:silent
import net.michalsitko.xml.parsing.XmlParser

val input =
    """<?xml version="1.0" encoding="UTF-8"?>
      |<a xmlns="http://a.com" xmlns:b="http://b.com">
      |   <c1>
      |      <f>a.com</f>
      |      <b:f>b.com</b:f>
      |   </c1>
      |   <c1 xmlns="http://c.com" xmlns:b="http://d.com">
      |      <f>c.com</f>
      |      <b:f>d.com</b:f>
      |   </c1>
      |</a>""".stripMargin
val xml = XmlParser.parse(input).right.get
```

For sake of this example, we define helper method `withCriteria`:

```tut:silent
import monocle.Traversal
import net.michalsitko.xml.entities.{Element, LabeledElement}
import net.michalsitko.xml.optics.LabeledElementOptics._
import net.michalsitko.xml.optics.ElementOptics._

def withCriteria(criteria: Traversal[Element, Element]): Traversal[LabeledElement, String] = {
  deep("c1").composeTraversal(criteria).composeOptional(hasTextOnly)
}
```

It returns `Traversal` which looks for elements defined by argument `criteria`. We will call this method a few times 
with different `criteria`.

First, let's analyze the output when we use plain String as element label; consequently ignoring namespaces:

```tut:book
val ignoreNs = withCriteria(deeper("f"))
ignoreNs.getAll(xml)
```

As expected, we fetched texts of all `f`-labeled elements.

Now, let's see how output changes after we query for `f`-labeled elements within `http://a.com` namespace:
 
```tut:book
val withNsA = {
  val ns = Namespace("http://a.com")
  withCriteria(deeper(ns.name("f")))
}
withNsA.getAll(xml)
```

This time result contains just `"a.com"`.

### Attributes

### Default namespace

There's no notion of default namespace in `xml-lens` as such notion heavily depends on prefixes. Default namespace is 
ambiguous because it may refer to different namespaces within a single XML document.

While you don't need a concept of default namespace, sometimes you may want to access elements with no namespace at all.
Element has no namespace if a document has no default namespace or the applied namespace was declared with empty value 
(i.e. `xmlns=""`).

Let's consider following XML:

```tut:silent
val input2 =
    """<?xml version="1.0" encoding="UTF-8"?>
      |<a xmlns:b="http://b.com">
      |   <c1>
      |      <f>no namespace</f>
      |      <b:f>b.com</b:f>
      |   </c1>
      |</a>""".stripMargin
```

As you see there's no default namespace at all. If you need to access `f`-labeled element without namespace (one with 
text `no namespace`) you need to call `name` on `Namespace.empty`:

```tut:book
val defaultNs = {
  val ns = Namespace.empty
  withCriteria(deeper(ns.name("f")))
}

defaultNs.getAll(xml)
```
