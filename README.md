# Readium Annotations

User annotations convey textual information about a resource segment in a publication. 

This document defines a syntax for Annotation Documents, serialised in JSON and meant to be included in an EPUB, shared as a file or referenced in a [Readium Web Publication Manifest](https://readium.org/webpub-manifest).

**Editors:**

* Laurent Le Meur

**Participate:**

* [GitHub](https://github.com/readium/annotations/)
* [Discussions](https://github.com/readium/annotations/discussions/)

## Use cases

* A user decides to export from his reading application an EPUB ebook he has previously annotated. He selects the option to save his annotations in the ebook. The exported EPUB contains his annotations. If he imports the ebook into another reading application that supports this specification, his annotations will reappear. 
* A user decides to export the annotations he has created in an ebook from his reading application. He chooses a file name and a destination folder, and he validates his choice. An annotation file is created on his computer. If he imports this file into another reading application that supports this specification, and if the reading application already contains this ebook, his annotations will now appear in the ebook.
* A teacher reads an ebook and prepares annotations. He exports an annotation file and shares this file with his students. The students import the ebook and the detached annotation file into their reading application: the annotations made by the teacher appear in the ebook. The students can add their annotations to the ebook, and these student annotations will not be mixed with the annotations created by their teacher. 
* A user annotates an ebook. He shares his annotations with other users via a Cloud mechanism handled by his reading application.
* A user annotates an ebook. He adds semantics to his annotations, e.g. differentiating “to be corrected” from “to be discussed” annotations. 


## 1. Annotation 

### 1.1. Annotation Object

Annotations are modelled after the W3C Web Annotation Data Model (https://www.w3.org/TR/annotation-model/) and adopt its JSON-LD syntax. 

This document defines a profile of the W3C Annotation Data Model by defining a subset of the properties allowed in this model and adding a few properties. 

This document defines the following annotation properties: 

| Name | Description | Format | Required? |
| ---- | ----------- | ------ | --------- |
| `@context`| The context that determines the meaning of the JSON as an Annotation. It MUST be “http://www.w3.org/ns/anno.jsonld”. | string | Yes |
| `id` | The identity of the annotation. A uuid formatted as a URN is recommended. | URI | Yes |
| `type` | The RDF structure type. It MUST be "Annotation". | string | Yes |
| `motivation` | The motivation for the annotation's creation. Only used for tagging bookmarks. | "bookmarking" | No |
| `created` | The time when the annotation was created. | ISO 8601 datetime | Yes |
| `modified` | The time when the annotation was modified, after creation. | ISO 8601 datetime | No |
| `creator` | The creator of the annotation. This may be either a human or an organization. | Creator | No |
| `target` | The target content of the annotation. | Target | Yes |
| `body` | The annotation body. | Body | No |

An annotation without Body structure corresponds to a "highlight" or a "bookmark". 

A bookmark is defined by the inclusion of a `motivation` property with a value `bookmarking`.  

Sample 1: Core structure of a Readium annotation

```json
{
  "@context": "http://www.w3.org/ns/anno.jsonld",
  "id": "urn:uuid:123-123-123-123",
  "type": "Annotation",
  "created": "2023-10-14T15:13:28Z",
  "modified": "2024-01-29T09:00:00Z",
  "target": {
   },
  "body": {
  }
}
```

### 1.2. Creator

The creator of an annotation is a person or an organisation. 

This document defines the following creator properties: 

| Name | Description | Format | Required? |
| ---- | ----------- | ------ | --------- |
| `id`| The identity of the creator. | URI | Yes |
| `type` | The RDF structure type. It MUST be "Person" or "Organization". | string | Yes |
| `name`| The name of the creator.  | string | No |

### 1.3. Target

The target of an annotation associates the annotation to a specific segment of a resource in the current publication. 

This document defines three target sub-properties: 

| Name | Description | Format | Required? |
| ---- | ----------- | ------ | --------- |
| `source`| The identity of the target resource. | URI | Yes |
| `selector`| The segment of the target resource that is annotated.  | Array of Selector objects | No |
| `meta`| Indications that help locate the selector in the resource. | Meta | No |

A Target with no Selector indicates that the annotation is targeting the entire target resource. 

#### 1.3.1. Source 

The target resource MUST be identified by the URL of an existing resource in the publication.

If the annotation is attached to an EPUB, the source value must also be present as one of the item/@href values of the [manifest element](https://www.w3.org/TR/epub-33/#sec-manifest-elem).  

If the annotation is attached to a Web Publication, the source value must also be present as one of the readingOrder/@href or resource/@href values on the [Web Publication Manifest](https://github.com/readium/webpub-manifest?tab=readme-ov-file#21-sub-collections).

Sample 2: the source of the annotation is the relative URL identifying an HTML document in an EPUB.

```json
{
  "@context": "http://www.w3.org/ns/anno.jsonld",
  "type": "Annotation",
  "target": {
    "source": "OEBPS/text/chapter1.html",
    "selector": [
    ],
    "meta": {
    }
  }
}
```

#### 1.3.2. Selector

An annotation refers to a segment of a resource, which is identified by one or more Selectors. The nature of the Selector and methods to describe segments depend on the resource type. Providing more than one Selector allows an annotation software to choose the most accurate selector from those it can handle, and helps accomodating evolutions on the annotated resource. 

Annotation selectors are specified in [W3C Annotation Data Model, section Selectors](https://www.w3.org/TR/2017/REC-annotation-model-20170223/#selectors). This specification filters selectors deemed useful for annotating publications, details the use of these selectors and defines new selectors when deemed useful. 

Note: In (X)HTML documents, reading systems locate a segment of text using a [DOM Range](https://dom.spec.whatwg.org/#interface-abstractrange). Most Selectors are in practice serialized representations of such Range after normalisation.

##### 1.3.2.1. Text Quote Selector

Important: This selector contains full annotated text segments and, therefore, must not be used when the content is protected by a DRM which limits the number of characters that may be copied.   

Whitespaces present in the source document are preserved in the “prefix”, “exact”, and “suffix” segments and are represented as per the JSON Grammar (e.g. `\n`, `\t` etc.). 

Note: There is no restriction on the amount of the preceding and following text that can be included in the selector, but this amount should be left as low as possible.

Sample 3: A text segment represented as a TextQuoteSelector.

```json
{
  "selector": [
    {
    "type": "TextQuoteSelector",
    "exact": "Combien de fois \n\n    ne m’avait-il",
    "prefix": "trouver quelqu’un    \n  \t\t    comme vous. ",
    "suffix": " pas \n\n      reproché de travailler ma"
    }
  ]
}
```

##### 1.3.2.2. Text Fragment Selector

Important: this selector corresponds to a trend on the Web, to highlight text using a url-compliant and short fragment identifier. Mapping such a text fragment to or from a DOM Range is still uneasy, by lack of API.   

A TextFragmenSelector, which is not defined in the W3C Annotation Model, contains a shortened version of the annotated text segment and optional indication of the preceding and following fragments.

TextFragmenSelector could be defined as a FragmentSelector conforming to the W3C Draft Community Group report named [URL Fragment Text Directives](https://wicg.github.io/scroll-to-text-fragment/). We prefer defining a dedicated Selector for the purpose, which simplifies the writing and processing of the structure. 

Sample 4.1: A text segment represented as a TextFragmentSelector.

```json
{
  "selector": [
    {
    "type": "TextFragmentSelector",	 
    "value": "an%20example,text%20fragment"
    }
  ]
}
```

Anti-pattern 4.2: this would be correct as per the W3C Annotation Model but this would be more verbose:

```json
{
  "selector": [
    {
    "type": "FragmentSelector",	 
    "conformsTo": "https://wicg.github.io/scroll-to-text-fragment/",
    "value": "#:~:text=an%20example,text%20fragment"
    }
  ]
}
```

##### 1.3.2.3. EPUB CFI Selector

Important: Our community lacks a reference and open-source implementation of EPUB CFI to DOM Range conversion (back and forth). Current implementations are not fully interoperable.

Note: Both the left-hand part (resource location in the publication) and the right-hand part (anchoring of the text fragment) of the path must be present.

To correspond to a text segment, the EPUB CFI expression MUST correspond to a range in the form epubcfi(P,S,E), where P represents a common Parent path, S and E the start and end subpaths respectively. 

The EPUBCFISelector is the equivalent of a FragmentSelector conforming to http://www.idpf.org/epub/linking/cfi/epub-cfi.html, as defined in the W3C Annotation Model; the latter is more verbose and therefore deprecated.

Sample 5.1: A text segment represented as an EPUB CFI:

```json
{
  "selector": [
    {
    "type": "EPUBCFISelector",	 
    "value": "/6/4[chap01ref]!/4[body01]/10[para05],/2/1:1,/3:4"
    }
  ]
}
```

Anti-pattern 5.2: this would be correct as per the W3C Annotation Model but this would be more verbose:

```json
{
  "selector": [
    {
    "type": "FragmentSelector",	 
    "conformsTo": "http://www.idpf.org/epub/linking/cfi/epub-cfi.html",
    "value": "epubcfi(/6/4[chap01ref]!/4[body01]/10[para05],/2/1:1,/3:4)"
    }
  ]
}
```

##### 1.3.2.4. Range Selector + CSS Selector + XPath Selector + Character Position

A Range Selector identifies the beginning and the end of the selection by using other Selectors. It contains two CSS Selectors. Each CSS Selector references the parent element of the text node containing the annotation start or end character. It is optionally refined by a Text Node Selector which points a specific text node in the parent element, itself refined by a Character Selector which selects a character in this text node. The Text Node Selector can only be omitted if the parent element contains a single text node.  

The TextNodeSelector is the equivalent of an XPathSelector selecting a specific text node in an element node; the latter is more verbose and therefore deprecated.

The CharacterSelector is similar to a FragmentSelector conforming to http://tools.ietf.org/rfc/rfc5147, as defined in the W3C Annotation Model; the latter is more verbose and therefore deprecated. The disrepency between both selectors comes from the fact that a CharacterSelector is counting code units, not code points.  

Note: counting code units is quite simpler for JS developers.

Note: such construct can be mapped from and to a DOM Range using simple code. See https://www.npmjs.com/package/css-selector-generator for an example of open-source codebase generating CSS Selectors from a Node; For getting a Node from a CSS Selector, developers will use document.querySelector().

Sample 6.1: A text segment represented as two CSS Selectors; note that the start and end selectors are not at the same level of the DOM tree:

```json
{
  "selector": [
    {
      "type": "RangeSelector",	 
      "startSelector": {
        "type": "CSSSelector",
        "value": "#intro > p:nth-child(2)",
        "refinedBy": {
          "type": "TextNodeSelector",	 
          "value": 2,
          "refinedBy": {
            "type": "CharacterSelector",	 
            "value": 5
          },
        },
      },
      "endSelector": {
        "type": "CSSSelector",
        "value": "#intro > p:nth-child(3) > em",
        "refinedBy": {
          "type": "CharacterSelector",	 
          "value": 4
        }
      }
    }
  ]
}
```

This selects "j" from "jumps" as start selector and "e" from "white" as end selector.  

```html 
<div id="intro">
  <p>Some text.</p>
  <p>The quick <em>brown</em> fox jumps over the lazy dog.</p>
  <p>The lazy <em>white</em> dog sleeps with the crazy fox.</p>
</div>
```

Anti-pattern 6.2: this would be correct as per the W3C Annotation Model but this would be more verbose:

```json
{
  "selector": [
    {
      "type": "RangeSelector",	 
      "startSelector": {
        "type": "CSSSelector",
        "value": "#intro > p:nth-child(2)",
        "refinedBy": {
          "type": "XPathSelector",	 
          "value": "text()[2]",
          "refinedBy": {
            "type": "FragmentSelector",	 
            "conformsTo": "http://tools.ietf.org/rfc/rfc5147",
            "value": "char=5"
          },
        },
      },
      "endSelector": {
        "type": "CSSSelector",
        "value": "#intro > p:nth-child(3) > em",
        "refinedBy": {
          "type": "FragmentSelector",	 
          "conformsTo": "http://tools.ietf.org/rfc/rfc5147",
          "value": "char=4"
        }
      }
    }
  ]
}
```

##### 1.3.2.5. Range Selector + XPath Selector + Character Position

A Range Selector identifies the beginning and the end of the selection by using other Selectors. It contains two XPath Selectors. Each XPath Selector references the text node containing the annotation start or end character. It is refined by a Character Selector which selects a character in this text node.

Note: such construct is simpler than the one using an CSS Selector + an XPath Selector, but it is to be tested for performance.

Sample 7: A text segment represented as two XPath Selectors; note that the start and end selectors are not at the same level of the DOM tree:

```json
{
  "selector": [
    {
      "type": "RangeSelector",	 
      "startSelector": {
        "type": "XPathSelector",
        "value": "/div/p[2]/text()[2]",
        "refinedBy": {
          "type": "CharacterSelector",	 
          "value": 5
        },
      },
      "endSelector": {
        "type": "CSSSelector",
        "value": "/div/p[3]/em/text()",
        "refinedBy": {
          "type": "CharacterSelector",	 
          "value": 4
        }
      }
    }
  ]
}
```

The resulting selection is the same as in example 6.

##### 1.3.2.6. CSS Selector

If the annotation is relative to an image, an audio or video clip or any other element in the DOM that does not correspond to a text fragment, a CSS Selector can identify this object. The annotation is therefore contextual to the HTML resource, not directly attached to the media resource. 

Sample 8: A CSS Selectors identifies the 5th img in the resource.

```json
{
  "selector": [
    {
    "type": "CSSSelector",
    "value": "img:nth-child(5)"
    }
  ]
}
```

##### 1.3.2.7. Spatial Selector

If the resource is an image (i.e. in a Divina publication), it is possible to identify an area of interest using a Spatial Selector.

The SpatialSelector is the equivalent of a FragmentSelector conforming to http://www.w3.org/TR/media-frags/, as defined in the W3C Annotation Model; the latter is more verbose and therefore deprecated.

Note: this selector is also adequate for PDF publications, where the page is already set via a source property. 

Sample 9.1: An image fragment identified by a Spatial Selector.

```json
{
  "selector": [
    {
    "type": "SpatialSelector",	 
    "value": "50,50,650,480" 
    }
  ]
}
```

Anti-pattern 9.2: this would be correct as per the W3C Annotation Model but this would be more verbose:

```json
{
  "selector": [
    {
    "type": "FragmentSelector",	 
    "conformsTo": "http://www.w3.org/TR/media-frags/",
    "value": "xywh=50,50,650,480" 
    }
  ]
}
```

##### 1.3.2.8. Temporal Media Fragment Selector

If the resource is an audio file (i.e. in an Audiobook), it is possible to identify an area of interest using a Temporal Selector.

The TemporalSelector is the equivalent of a FragmentSelector conforming to http://www.w3.org/TR/media-frags/, as defined in the W3C Annotation Model; the latter is more verbose and therefore deprecated.

Sample 10.1: An audio fragment identified by a Temporal Selector.

```json
{
  "selector": [
    {
    "type": "TemporalSelector",	 
    "value": "30,60" 
    }
  ]
}
```

Anti-pattern 10.2: this would be correct as per the W3C Annotation Model but this would be more verbose:

```json
{
  "selector": [
    {
    "type": "FragmentSelector",	 
    "conformsTo": "http://www.w3.org/TR/media-frags/",
    "value": "t=30,60" 
    }
  ]
}
```

##### 1.3.2.10. ProgressionSelector

A ProgressionSelector, which is not defined in the W3C Annotation Model, contains a decimal value representing the annotation's position as a percentage of the total size of the resource.

While such positioning is imprecise and does not propertly identifies a fragment, it is useful as a way to order annotations in a list, and help positioning the annotation near the corresponding fragment if other selectors fail.

Sample 12: A ProgressionSelector indication that the annotation is positioned just after the middle of the resource:

```json
{
  "selector": [
    {
    "type": "ProgressionSelector",	 
    "value": 0.534234255
    }
  ]
}
```

#### 1.3.3. Meta

Meta information MAY be added to an annotation as “breadcrumbs”,  to ease the display of contextual information relative to the global position of the annotation in the publication.

The meta property contains: 

| Name | Description | Format | Required? |
| ---- | ----------- | ------ | --------- |
| `headings`| Ancestor headings of the annotation. | Array of Heading objects | No |
| `page`| Page of the publication containing the annotation. It may be either a synthetic page or a print equivalent. It is essentially a visual indicator. | string | No |

##### 1.3.3.1. Headings

The Headings object contains:

| Name | Description | Format | Required? |
| ---- | ----------- | ------ | --------- |
| `level`| Heading level. | number | No |
| `txt`| Heading title. | string | No |

Sample 10: Meta information contains ancestor headings and a page number:

```json
{
  "@context": "http://www.w3.org/ns/anno.jsonld",
  "type": "Annotation",
  "target": {
    "source": "OEBPS/text/chapter11.html",
    "selector": [
    ],
    "meta": {
      "headings": [
        {
        "level": 1,
        "txt": "Section 11"
        },
        {
        "level": 2,
        "txt": "Sub Section 1"
        }
      ],
      "page": "XI"       
    }
  }
}
```

### 1.4. Body

The body of an annotation contains plain text, style, and an optional keyword. 

The body property contains: 

| Name | Description | Format | Required? |
| ---- | ----------- | ------ | --------- |
| `type`| The body type. It MUST be “TextualBody”. | string | Yes |
| `value`| The textual content of the annotation. | string | Yes |
| `format`| The media-type of the annotation value; 'plain/text' by default. | rfc6838 | No |
| `color`| The colour of the annotation; yellow by default. | "pink" \| "orange" \| "yellow" \| "green" \| "blue" \| "purple" | No |
| `highlight`| The style of the annotation; solid background by default. | "solid" \| "underline" \| "strikethrough" \| "outline" | No |
| `language`| The language of the annotation. | BCP47 | No |
| `textDirection`| The direction of the text; left-to-right by default. | "ltr" \| "rtl" | No |
| `keyword`| Free text categorising the annotation. | string | No |

Note: read “Best practices for Reading Systems” about using a keyword in an annotation. 

Sample 11: An annotation Body. 

```json
{
  "@context": "http://www.w3.org/ns/anno.jsonld",
  "type": "Annotation",
  "body": {
    "type" : "TextualBody",
    "value" : "j'adore !",
    "keyword" : "teacher",   
    "color" : "blue",
    "language" : "fr",
    "textDirection" : "ltr"
  }
}
```

## 2. Annotation Set

An Annotation does not contain information about its associated publication. If a set of annotations is shared as a detached file, it is mandatory to export with them information that will help find the associated publication even if the publication is not adequately identified.

Note: the AnnotationCollection defined by the W3C does not provide an adequate structure for sharing annotations either as a detached file or as a file embedded in a Zip package. The AnnotationCollection is intrinsically paginated and provides a way to retrieve annotations via a REST API. 

The AnnotationSet object contains: 

| Name | Description | Format | Required? |
| ---- | ----------- | ------ | --------- |
| `@context`| The context that determines the meaning of the JSON as an annotation set. It MUST be “http://www.w3.org/ns/anno.jsonld”. | string | Yes |
| `id` | The identity of the annotation set. A uuid formatted as a URN is recommended. | URI | Yes |
| `type` | The RDF structure type. It MUST be "AnnotationSet". | string | Yes |
| `generator`| The agent responsible for the generation of the object serialisation. | Generator | No |
| `about`| Information relative to the publication. | About object | Yes |
| `generated`| The time when the set was generated. | ISO 8601 datetime | No |
| `title`| A title helping on the identification of the set. | string | No |
| `items`| The annotations of the set.  | Array of Annotation objects | Yes |

### 2.1. Generator

The Generator object contains information relative to the software from which the serialized annotation has been produced. 

| Name | Description | Format | Required? |
| ---- | ----------- | ------ | --------- |
| `id` | The identity of the generator software. The recommended value is the Github URL of the application source-code. | URI | Yes |
| `type` | The RDF structure type. It MUST be "Software". | string | Yes |
| `name`| The name of the generator software. | string | Yes |
| `homepage`| The home page presenting the generator software. | URL | No |

### 2.2. About

The About object contains information relative to the publication. Such metadata in intended to help associating an annotation set with a publication: 

| Name | Description | Format | Required? |
| ---- | ----------- | ------ | --------- |
| `dc:identifier`| Publication identifiers. An ISBN is preferred.  | Array of strings | No |
| `dc:title`| The title of the publication. | string | No |
| `dc:format`| The media type of the publication. | string | No |
| `dc:publisher`| The name of the publisher. | string | No |
| `dc:creator`| The author(s) of the publication. | array of strings | No |
| `dc:date`| The release year. | calendar year using four digits | No |

Note: all properties defined above are from the Dublin Core vocabulary, referenced in the Web Annotation Data Model. 

Sample 12: An AnnotationSet containing one annotation. 

```json
{
  "@context": "http://www.w3.org/ns/anno.jsonld",
  "id": "urn:uuid:123-123-123-123",
  "type": "AnnotationSet",
  "generator": "https://github.com/edrlab/thorium-reader/releases/tag/v3.1.0",
  "generated": "2023-09-01T10:00:00Z",
  "title": "Annotations Mme Prof, La Peste, cours 1ere B",
  "about": {
     "dc:identifier": [
        "urn:isbn:1234567890"
     ],
     "dc:format": "application/epub+zip",
     "dc:title": "Alice in Wonderland",
     "dc:publisher": "Example Publisher",
     "dc:creator": ["Anne O'Tater"],
     "dc:date": "1865"
  },
  "items": [
    {
      "@context": "http://www.w3.org/ns/anno.jsonld",
      "id": "urn:uuid:234-234-234-234",
      "type": "Annotation",
      "target": {
      },
      "body": {
      }
    }
  ]
}
```

### 2.3. Media type of an Annotation Set

This specification introduces a dedicated media type value to identify an AnnotationSet: `application/rd-annotations+json`.

HTTP responses associated with annotation files must indicate this media type in their header.

Note: I propose using a “rd-” prefix (for ReaDium). It would allow for easier registration at [IANA](https://www.iana.org/assignments/media-types/media-types.xhtml). Should we extend this to other media types, including “application/webpub+json”?

### 2.4. File extension of an Annotation Set

This specification introduces a dedicated file extension for serialized AnnotationSets: `.ann`.  

# 3. Embedding annotations in publications

## 3.1. In EPUB

The OPTIONAL `annotations.ann` file in the META-INF directory holds an AnnotationSet. 

## 3.2. In Readium Web Publications

The JSON file holding an AnnotationSet MUST be represented as a link object in the `links` collection, with an `annotations` relation. 

Sample 13: A Readium Web Publication Manifest containing a link to an annotations file.

```json
{
  "@context": "https://readium.org/webpub-manifest/context.jsonld",
  "metadata": {
  },
  "links": [
    { 
      "rel": "self", 
      "href": "http://example.com/manifest.json", 
      "type": "application/webpub+json"
    },
    {
      "rel": "annotations", 
      "href": "https://example.com/annotations.ann", 
      "type": "application/rd-annotations+json"
    }
  ]
}
```

When a Web Publication is packaged using the Readium Packaging Format, it is up to the generator to embed the annotation file in the package or keep it remote.

# 4. Best practices for Reading Systems

_This section is non-normative._ 

## 4.1. Displaying filtered annotations

Filtering by colour is not sufficient because imported annotations may have the same colour as personal annotations.

Reading systems should enable filtering by colour, highlight mode, keyword and creator. For instance, a user can display "blue" annotations only, or “teacher” annotations only. Filtering on multiple criterias is a plus. 

## 4.2. Using multiple selectors

It is recommended that Reading Systems export Progression Selectors, as they can be used for sorting annotations when no other sortable selector is present.

When displaying an annotation, a Reading System is free to use the most precise Selector available. It will select an alternative Selector as a fallback in case the preferred one does not return a correct position in the publication: this can happen if the publication has been modified after the annotation has been created. 

## 4.3. Exporting annotations as a detached file

When a user decides to export an annotation set from a reading system, he SHOULD be proposed to filter the annotations by keywords (multiple choice). “Annotations with no keyword” and “All annotations” SHOULD be proposed as options. The advantage of this practice is that, for instance, a user can export personal annotations (usually with no keyword) and let “teacher” annotations unexported. 

He MAY enter a title for the annotation set (empty by default). Such title SHOULD become the exported filename.  

He MUST be proposed to choose the directory in which the annotation set will be stored. 

The file extention of the file MUST be `.ann`. 

The application may propose alternative formats at export time: a HTML or markdown format may be handy, as a list of annotations with human-friendly references to the location of each annotation. 

## 4.4. Exporting annotations in a publication

When a user decides to export a publication from the Reading System, he SHOULD be proposed to embed the annotations associated with the publication. 

If the user decides to embed annotations in a publication, he SHOULD be be proposed to filter the annotations by keywords (multiple choice).

## 4.5. Importing annotations

To simplify the association of annotations with a publication, a Reading System MUST offer a way to select a publication before selecting an annotation set. The drag&drop of an annotation set into a Reading System MAY also be proposed, but identifying the proper publication from the metadata in the annotation set is more complicated.

When importing an annotation set, a Reading System SHOULD display a message with the title of the annotation set and the number of annotations in the set. The Reading System MUST offer the user the choice to abort the import.

Each annotation is uniquely identified. If during the import of an annotation set, one or more annotations are re-imported, the Reading System MUST offer to the user the choice to override existing annotations or abort the import of the annotation set. 

## 4.6. Dealing with colours

This document specifies a closed set of six colours, chosen because of their large support in well-known reading systems. But most existing reading apps offer a smaller set to their users.

If an application imports annotations with a colour it does not support, it should display these annotations with a neutral colour. The recommended neutral colour is grey.

Some applications may support colours that are not in the set defined by this specification (e.g. brown). In this case, a 1-to-1 substitution at export time is required (e.g. brown to orange). 

Note: we didn't spot applications which offer more than six annotation colours.

# 5. JSON Schemas

# 6. References

Open Annotation in EPUB, 2015: https://idpf.org/epub/oa/ 

Open Annotation in EPUB, CFI pros and cons, 2014: https://docs.google.com/document/d/1d0IRsb2h9LM-ZWPwjS4dZ4Fffg4u9qA3Dkw0P9_3o5Y/edit 

EPUB CFI cannot reference content in non-spine items, 2023: https://github.com/w3c/epub-specs/issues/226 

W3C Web Annotation Data Model, 2017: https://www.w3.org/TR/annotation-model/ 

EPUB CFI, 2017: https://idpf.org/epub/linking/cfi/ 

EPUB 3.3, 2023: https://www.w3.org/TR/epub-33/ 

Readium Web Publication Manifest: https://readium.org/webpub-manifest/ 