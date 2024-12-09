# New Selectors

This document details proposals for an evolution of the specification of selectors in the [W3C Annotation Data Model](https://www.w3.org/TR/annotation-model/)). 

It complements the [Readium Annotations specification](./README.md), which so far introduces serializations totally compatible with the W3C Annotation Data Model (plus a Thorium experiment).

## Create a ProgressionSelector

A ProgressionSelector contains a decimal value representing the annotation's position as a percentage of the total size of the resource.

While such positioning is imprecise and does not propertly identifies a fragment, it is useful as a way to order annotations in a list, and help positioning the annotation near the corresponding fragment if other selectors fail.

Sample: This ProgressionSelector indicates that the annotation is positioned just after the middle of the resource:

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

A RangeSelector identifies the beginning and the end of the selection by using other Selectors. It contains two CSSSelectors. Each CSSSelector references the parent element of the text node containing the annotation start or end character. Each CSSSelector is refined by a TextNodeIndexSelector which points at a specific text node in the parent element and a CodeUnitSelector that targets a character in this text node, using unicode code units.

Question: If EPUB CFIs are correctly implemented in EPUB, and CSSSelector + TextPositionSelector sufficiently efficient, is this form really useful?   

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




