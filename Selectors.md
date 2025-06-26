# New Selectors

This document details proposals for an evolution of the specification of selectors in the [W3C Annotation Data Model](https://www.w3.org/TR/annotation-model/)). 

It complements the [Readium Annotations specification](./README.md), which so far introduces serializations totally compatible with the W3C Annotation Data Model (plus a Thorium experiment).

## Use the EPUB CFI Selector? 

Reference in the W3C Annotation Data Model: [EPUB CFI Selector](https://www.w3.org/TR/annotation-model/#fragment-selector)

This Selector describes a range of text in an EPUB resource by expressing a starting and ending character using the [EPUB Canonical Frangment Identifier](http://www.idpf.org/epub/linking/cfi/epub-cfi.html) syntax.  

To correspond to a text segment, the EPUB CFI expression MUST correspond to a range in the form epubcfi(P,S,E), where P represents a common Parent path, S and E the start and end subpaths respectively. 

Note: Both the left-hand part (resource location in the publication) and the right-hand part (anchoring of the text fragment) of the path must be present.

Important: The reading system community lacks a reference and open-source implementation of EPUB CFI to DOM Range conversion (back and forth). Current implementations are not fully interoperable, essentially because some implementations use unicode code points instead of code units. Our implementation assumes the use of code units.

Implementation: This selector is exported from and processed by Thorium Reader 3.1 when the publication is in EPUB format. 

Sample 4: A text segment represented as an EPUB CFI:

```json
{
  "selector": [
    {
    "type": "FragmentSelector",	 
    "conformsTo": "http://www.idpf.org/epub/linking/cfi/epub-cfi.html",
    "value": "epubcfi(/6/4!/4[body01]/10[para05],/2/1:1,/3:4)"
    }
  ]
}
```

Note: It would be more efficient to create a simpler EPUBCFISelector, as an equivalent of the FragmentSelector conforming to http://www.idpf.org/epub/linking/cfi/epub-cfi.html defined in the W3C Annotation Model, which would avoid expressing the "epubcfi()" part and the left part of the CFI which relates to the resource already selected by the "source" property of the annotation. See below. 

## Use the CSS Selector

Reference in the W3C Annotation Data Model: [CSS Selector](https://www.w3.org/TR/annotation-model/#css-selector)

This Selector describes an HTML element in an EPUB resource using the [CSS Selectors Level 3](https://www.w3.org/TR/selectors-3/) syntax.  

It is used to select an image, an audio or video clip or any other element in the DOM that does not correspond to a text fragment. The annotation is therefore contextual to the HTML resource, not directly attached to the media resource.

Note: such construct can be mapped from and to a DOM Range using simple code. See https://www.npmjs.com/package/css-selector-generator for an example of open-source codebase generating CSS Selectors from a Node; For getting a Node from a CSS Selector, developers will use document.querySelector().

Implementation: Thorium Reader 3.1 does not allow the selection of a media resource. This will be added in a future version.

Sample 5: A CSS Selector points at the 5th img in the resource.

```json
{
  "selector": [
    {
    "type": "CssSelector",
    "value": "img:nth-child(5)"
    }
  ]
}
```


## Create a TextFragmentSelector

There is a trend in Web browsers to highlight text using a url-compliant and short fragment identifier. 

Warning: mapping a text fragment to a DOM Range is still uneasy, bacause of a lack of standard API in web browsers.   

A TextFragmenSelector contains a shortened version of the annotated text segment and optional indication of the preceding and following fragments.

TextFragmenSelector could be defined as a FragmentSelector conforming to the W3C Draft Community Group report named [URL Fragment Text Directives](https://wicg.github.io/scroll-to-text-fragment/). We prefer defining a dedicated Selector for the purpose, which simplifies the writing and processing of the structure. 

Sample: A text segment represented as a TextFragmentSelector.

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

## Replace the generic FragmentSelector by a set of specific selectors

### EPUBCFISelector

The EPUBCFISelector replaces the FragmentSelector conforming to http://www.idpf.org/epub/linking/cfi/epub-cfi.html. The "epubcfi()" expression, which has no use, is removed from the selector value; same for the left part of the CFI, which relates to the resource already selected by the "source" property of the annotation.

Sample: a text fragment identified by an EPUB CFI Selector.

```json
{
  "selector": [
    {
    "type": "EPUBCFISelector",	 
    "value": "/4[body01]/10[para05],/2/1:1,/3:4"
    }
  ]
}
```

### SpatialSelector

The SpatialSelector is the equivalent of a FragmentSelector conforming to http://www.w3.org/TR/media-frags/. The "xywh=" prefix is removed from the selector value.

This selector is adapted to the description of an area of interest in an image (i.e. in a Divina publication). It is also adapted to PDF publications, where the page is already set via a source property. 

Sample: An image fragment identified by a Spatial Selector.

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

### TemporalSelector

The TemporalSelector is the equivalent of a FragmentSelector conforming to http://www.w3.org/TR/media-frags/. The "t=" prefix is removed from the selector value.

This selector is adapted to the description of an area of interest in an audio file (i.e. in an Audiobook).

Sample: An audio fragment identified by a Temporal Selector.

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


## Create TextNodeIndexSelector and CodeUnitSelector for direct DOM Range mapping

Using these two Selectors, it is possible to obtain an optimized mapping between annotation selectors and DOM ranges while using the elegant but verbose `refinedBy` mechanism offered by the W3C Annotation Data Model.

A RangeSelector identifies the beginning and the end of the selection by using other Selectors. It contains two CssSelectors. Each CssSelector references the parent element of the text node containing the annotation start or end character. Each CssSelector is refined by a TextNodeIndexSelector which points at a specific text node in the parent element and a CodeUnitSelector that targets a character in this text node, using unicode code units.

Question: If EPUB CFIs are correctly implemented in EPUB, and CssSelector + TextPositionSelector sufficiently efficient, is this form really useful?   

Sample: A text segment represented using a RangeSelector and a cascade of CssSelector, TextNodeIndexSelector and CodeUnitSelector; note that the start and end selectors are not at the same level in the DOM tree:

```json
{
    "type": "RangeSelector",
    "startSelector": {
        "type": "CssSelector",
        "value": "#intro > p:nth-child(2)",
        "refinedBy": {
            "type": "TextNodeIndexSelector",
            "value": 0,
            "refinedBy": {
                "type": "CodeUnitSelector",
                "value": 4
            }
        }
    },
    "endSelector": {
        "type": "CssSelector",
        "value": "#intro > p:nth-child(2)",
        "refinedBy": {
            "type": "TextNodeIndexSelector",
            "value": 2,
            "refinedBy": {
                "type": "CodeUnitSelector",
                "value": 10
            }
        }
    }
}
```

This selects "j" from "jumps" as start selector and (after) "e" from "white" as end selector.  

```html 
<div id="intro">
  <p>Some text.</p>
  <p>The quick <em>brown</em> fox jumps over the lazy dog.</p>
  <p>The lazy <em>white</em> dog sleeps with the crazy fox.</p>
</div>
```

There would be an alternative, compatible with the current W3C Annotation Data Model, but much more verbose.

The TextNodeIndexSelector is similar to an XPathSelector, defined in the W3C Annotation Model, pointing at one of the text nodes in the parent element.  

The CodeUnitSelector is similar but not equivalent to a FragmentSelector conforming to http://tools.ietf.org/rfc/rfc5147, defined in the W3C Annotation Model. Note that while the former is using unicode code units, the latter is using code points. 

Sample: a more verbose but equivalent selector for the previous HTML snippet. 

```json
{
  "selector": [
    {
      "type": "RangeSelector",	 
      "startSelector": {
        "type": "CssSelector",
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
        "type": "CssSelector",
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




