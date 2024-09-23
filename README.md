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
| `id` | The identity of the annotation. A uuid formatted as a URN is recommended. | IRI | Yes |
| `type` | The RDF structure type. It MUST be “Annotation”. | string | Yes |
| `created` | The time when the annotation was created. | ISO 8601 datetime | Yes |
| `modified` | The time when the annotation was modified, after creation. | ISO 8601 datetime | No |
| `target` | The target content of the annotation. | Target | Yes |
| `body` | The annotation body. | Body | Yes |

Note: created and modified are not part of the W3C Annotation Model. 

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

### 1.2. Target

The target of an annotation associates the annotation to a specific segment of a resource in the current publication. 

This document defines three target sub-properties: 

| Name | Description | Format | Required? |
| ---- | ----------- | ------ | --------- |
| `source`| The identity of the target resource. | IRI | Yes |
| `selector`| The segment of the target resource that is annotated.  | Array of Selector objects | Yes |
| `meta`| Indications that help locate the selector in the resource. | Meta | No |

#### 1.2.1. Source 

The target resource MUST be identified by a URL relative to the root of its parent publication.

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

#### 1.2.2. Selector

An annotation refers to a segment of a resource, which is identified by one or more Selectors. The nature of the Selector and methods to describe segments are dependent on the type of resource.

An EPUB annotation (where the publication is not protected by DRM with character copy restriction):

* MUST contain a TextQuoteSelector,
* MUST contain an EPUB CFI FragmentSelector,
* MAY contain a DomRangeSelector, 
* MAY contain an HTML FragmentSelector,
* MAY contain a ProgressionSelector.

An EPUB annotation (where the publication is protected by DRM with character copy restriction):

* MUST NOT contain a TextQuoteSelector,
* MUST contain an EPUB CFI FragmentSelector,
* MAY contain an HTML FragmentSelector,
* MAY contain a DomRangeSelector, 
* MAY contain a ProgressionSelector.

A PDF annotation:

* MUST contain a PDF FragmentSelector,
* MAY contain a ProgressionSelector.

A Divina annotation:

* MUST contain a Rectangular Media FragmentSelector,
* MAY contain a ProgressionSelector.

An Audiobook annotation:
* MUST contain a Temporal Media FragmentSelector,
* MAY contain a ProgressionSelector.

A Reading System is free to select the Selector with the best precision. It will select an alternative Selector as a fallback in case the preferred one does not return a correct position in the publication: this can happen if the publication has been modified after the annotation has been created. 

##### 1.2.2.1. TextQuoteSelector

The TextQuoteSelector is defined in https://www.w3.org/TR/annotation-model/#text-quote-selector. 

This selector contains text segments and, therefore, must be used with caution when the content is protected by a DRM, which limits the number of characters that may be copied (e.g. clipboard).   

Note: white spaces present in the source document are preserved in the “prefix”, “exact”, and “suffix” segments.

Sample 3: A TextQuoteSelector contains the annotated text segment, a segment before and a segment after: 

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

##### 1.2.2.2. FragmentSelector

A FragmentSelector (https://www.w3.org/TR/annotation-model/#fragment-selector) MUST conform to a specific type; the fragment type is indicated by the conformsTo property and the value property gives the fragment value.

In EPUB CFI fragment selectors, both the left-hand part (resource location in the publication) and the right-hand part (location of the annotation in the resource) are present.

PDF fragment selectors MUST conform to the [RFC 8118](https://www.rfc-editor.org/rfc/rfc8118).

Note: the RFC 8118 obsolates the [RFC 3778](https://www.rfc-editor.org/rfc/rfc3778) initially defined by the W3C Annotatopn Model. 

Sample 4: A couple of FragmentSelectors in an EPUB; the first selector is an idref, the second is an EPUB CFI:

```json
{
  "selector": [
    {
    "type": "FragmentSelector",	 
    "conformsTo": "http://tools.ietf.org/rfc/rfc3236",
    "value": "footnote1"
    },
    {
    "type": "FragmentSelector",	 
    "conformsTo": "http://www.idpf.org/epub/linking/cfi/epub-cfi.html",
    "value": "epubcfi(/6/4!/4[body01]/10[para05]/3:/10[para05]/10)"
    }
  ]
}
```

Sample 5: A PDF FragmentSelector:

```json
{
  "selector": [
    {
    "type": "FragmentSelector",	 
    "conformsTo": "https://www.rfc-editor.org/rfc/rfc8118",
    "value": "page=10&viewrect=50,50,650,480" 
    }
  ]
}
```

Sample 6: A Divina Rectangular Media FragmentSelector.

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

Sample 7: An Audiobook Temporal Media FragmentSelector:

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

##### 1.2.2.3. DomRangeSelector

A DomRangeSelector, not defined in the W3C Annotation Model, contains start and end container information; for each of these extremities, it provides a CSS Selector, a child text node index and an offset. 

Sample 8: A DomRangeSelector:

```json
{
  "selector": [
    {
    "type": "DomRangeSelector",
    "startContainerElementCssSelector": ".calibre_3",
    "startContainerChildTextNodeIndex": 0,
    "startOffset": 1066,
    "endContainerElementCssSelector": ".calibre_3",
    "endContainerChildTextNodeIndex": 0,
    "endOffset": 1095
    }
  ]
}
```

##### 1.2.2.4. ProgressionSelector

A ProgressionSelector, which is not defined in the W3C Annotation Model, contains a decimal value representing the annotation's position as a percentage of the total size of the resource.  

Sample 9: A ProgressionSelector indication that the annotation is positioned just after the middle of the resource:

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

#### 1.2.3. Meta

Meta information MAY be added to an annotation as “breadcrumbs”,  to ease the display of contextual information relative to the global position of the annotation in the publication.

The meta property contains: 

| Name | Description | Format | Required? |
| ---- | ----------- | ------ | --------- |
| `headings`| Ancestor headings of the annotation. | Array of Heading objects | No |
| `page`| Page of the publication containing the annotation. It may be either a synthetic page or a print equivalent. It is essentially a visual indicator. | string | No |

##### 1.2.3.1. Headings

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

### 1.3. Body

The body of an annotation contains plain text, style, and tags. 

The body property contains: 

| Name | Description | Format | Required? |
| ---- | ----------- | ------ | --------- |
| `type`| The body type. It MUST be “TextualBody”. | string | Yes |
| `value`| The textual content of the annotation. | string | Yes |
| `color`| The recommended colour of the annotation. | [CSS Color Module Level 3](https://www.w3.org/TR/css-color-3/) | No |
| `highlight`| The style of the annotation; solid background by default. | "solid" \| "underline" \| "strikethrough" \| "outline" | No |
| `language`| The language of the annotation. | BCP47 | No |
| `textDirection`| The direction of the text; left-to-right by default. | "ltr" \| "rtl" | No |
| `tag`| Free text categorising the annotation. | string | No |

Note: read “Best Practices for Reading Systems” about using tags in annotations. 

Sample 11: An annotation Body. 

```json
{
  "@context": "http://www.w3.org/ns/anno.jsonld",
  "type": "Annotation",
  "body": {
    "type" : "TextualBody",
    "value" : "j'adore !",
    "tag" : "teacher",   
    "color" : "#01E3F6",
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
| `id` | The identity of the annotation set. A uuid formatted as a URN is recommended. | IRI | Yes |
| `type` | The RDF structure type. It MUST be “AnnotationSet”. | string | Yes |
| `generator`| The agent responsible for the generation of the object serialisation. The preferred value is the Github URL of the application source-code. | IRI | Yes |
| `about`| Information relative to the publication. | About object | Yes |
| `generated`| The time when the set was generated. | ISO 8601 datetime | No |
| `title`| A label helping to identify the set. | string | No |
| `items`| The annotations of the set.  | Array of Annotation objects | Yes |

### 2.1. About

  The About object contains information relative to the publication. Such metadata in intended to help associating an annotation set with a publication: 

| Name | Description | Format | Required? |
| ---- | ----------- | ------ | --------- |
| `dc:identifier`| Publication identifiers. An ISBN is preferred.  | Array of strings | No |
| `dc:title`| The title of the publication. | string | No |
| `dc:format`| The media type of the publication. | string | No |
| `dc:publisher`| The name of the publisher. | string | No |
| `dc:creator`| The author, or comma separated authors. | string | No |
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
     "dc:creator": "Anne O'Tater",
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

### 2.2 Media type of an Annotation Set

This specification introduces a dedicated media type value to identify an AnnotationSet: `application/rd-annotations+json`.

HTTP responses associated with annotation files must indicate this media type in their header.

Note: I propose using a “rd-” prefix (for ReaDium). It would allow for easier registration at [IANA](https://www.iana.org/assignments/media-types/media-types.xhtml). Should we extend this to other media types, including “application/webpub+json”?

# 3. Embedding annotations in publications

## 3.1. In EPUB

The OPTIONAL `annotations.json` file in the META-INF directory holds an AnnotationSet. 

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
      "href": "https://example.com/annotations.json", 
      "type": "application/rd-annotations+json"
    }
  ]
}
```

When a Web Publication is packaged using the Readium Packaging Format, it is up to the generator to embed the annotation file in the package or keep it remote.

# 4. Best practices for Reading Systems

## 4.1. Displaying filtered annotations

Reading systems SHOULD enable filtering by color or by tag. Filtering by color is not sufficient because imported annotations may have the same color as personal annotations, or both may have no colour to enable them to be disambiguated.  

For instance, a user can display blue annotations only, or “teacher” annotations only. 

## 4.2. Importing annotations

To simplify the association of annotations with a publication, a Reading System MUST offer a way to select a publication before selecting an annotation set. The drag&drop of an annotation set into a Reading System MAY also be proposed, but identifying the proper publication from the metadata in the annotation set is more complicated.

When importing an annotation set, a Reading System SHOULD display a message with the title of the annotation set and the number of annotations in the set. The Reading System MUST offer the user the choice to abort the import.

When a user imports an annotation set, he SHOULD be offered to enter a tag to be added automatically to every annotation in the set. By default, the proposed tag MAY be the title of the annotation set. If the user decides to tag the annotations, the new tag will override each tag already present in the annotations contained in the set.

The advantage of doing this is that every annotation in a set titled “Grade B 2024” can be tagged by the user as “teacher”. It will become easy to only display “teacher” annotations. 

Each annotation is uniquely identified. If during the import of an annotation set, one or more annotations are re-imported, the Reading System MUST offer to the user the choice to override existing annotations or abort the import of the annotation set. 

## 4.3. Exporting annotations as a detached file

When a user decides to export an annotation set from a reading system, he SHOULD be proposed to filter the annotations by tags (multiple choice). “Untagged annotations” and “All annotations” SHOULD be proposed as options. The advantage of this practice is that, for instance, a user can export personal annotations (usually untagged) and let “teacher” annotations unexported. 

He MAY enter a title for the annotation set (empty by default). Such title SHOULD become the exported filename.  

He MUST be proposed to choose the directory in which the annotation set will be stored. 

The application may propose alternative formats at export time: a HTML or markdown format may be handy, as a list of annotations with human-friendly references to the location of each annotation. 

## 4.4. Exporting annotations in a publication

When a user decides to export a publication from the Reading System, he SHOULD be proposed to embed the annotations associated with the publication. 

If the user decides to embed annotations in a publication, he SHOULD be be proposed to filter the annotations by tags (multiple choice).

# 5. JSON Schemas

# 6. References

Open Annotation in EPUB, 2015: https://idpf.org/epub/oa/ 

Open Annotation in EPUB, CFI pros and cons, 2014: https://docs.google.com/document/d/1d0IRsb2h9LM-ZWPwjS4dZ4Fffg4u9qA3Dkw0P9_3o5Y/edit 

EPUB CFI cannot reference content in non-spine items, 2023: https://github.com/w3c/epub-specs/issues/226 

W3C Web Annotation Data Model, 2017: https://www.w3.org/TR/annotation-model/ 

EPUB CFI, 2017: https://idpf.org/epub/linking/cfi/ 

EPUB 3.3, 2023: https://www.w3.org/TR/epub-33/ 

Readium Web Publication Manifest: https://readium.org/webpub-manifest/ 