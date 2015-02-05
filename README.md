# GeoJSON Feature Sequences

A proposed standard for incrementally parseable GeoJSON data

Large JSON files pose a problem that is well explained in the motivation for the [JSON Text Sequences I-D](http://tools.ietf.org/html/draft-ietf-json-text-sequence-13).

> The JavaScript Object Notation (JSON) [RFC7159] is a very handy
serialization format.  However, when serializing a large sequence of
values as an array, or a possibly indeterminate-length or never-
ending sequence of values, JSON becomes difficult to work with.
>
> Consider a sequence of one million values, each possibly 1 kilobyte
when encoded -- roughly one gigabyte.  It is often desirable to
process such a dataset in an incremental manner: without having to
first read all of it before beginning to produce results.
Traditionally the way to do this with JSON is to use a "streaming"
parser, but these are neither widely available, widely used, nor easy
to use.

GeoJSON has exactly this problem with sequences of features, amplified by the large arrays of coordinates of each feature.

JSON Text Sequences proposes a standard for dealing with this problem.

> This document describes the concept and format of "JSON text
sequences", which are specifically not JSON texts themselves but are
composed of (possible) JSON texts.  JSON text sequences can be parsed
(and produced) incrementally without having to have a streaming
parser (nor streaming encoder).

GeoJSON Feature Sequences are a straight forward adaptation of JSON Text Sequences. A GeoJSON Feature Sequence is a document containing, instead of a single GeoJSON Feature Collection, multiple GeoJSON Feature texts that can be parsed and produced incrementally.

In a nutshell, GeoJSON Feature Sequences proposes to "explode" the following GeoJSON Feature Collection

```json
  { "type": "FeatureCollection",
    "features": [
      { "type": "Feature",
        "geometry": {"type": "Point", "coordinates": [102.0, 0.5]},
        "properties": {"prop0": "value0"}
        },
      { "type": "Feature",
        "geometry": {
          "type": "LineString",
          "coordinates": [
            [102.0, 0.0], [103.0, 1.0], [104.0, 0.0], [105.0, 1.0]
            ]
          },
        "properties": {
          "prop0": "value0",
          "prop1": 0.0
          }
        },
      { "type": "Feature",
         "geometry": {
           "type": "Polygon",
           "coordinates": [
             [ [100.0, 0.0], [101.0, 0.0], [101.0, 1.0],
               [100.0, 1.0], [100.0, 0.0] ]
             ]
         },
         "properties": {
           "prop0": "value0",
           "prop1": {"this": "that"}
           }
         }
       ]
     }
```

into a document of 3 separate incrementally parseable JSON texts, each containing a GeoJSON Feature, delimited by the ASCII RS `0x1e` (represented by `^^` below).

```json
^^{ "type": "Feature", "geometry": {"type": "Point", "coordinates": [102.0, 0.5]}, "properties": {"prop0": "value0"} }
^^{ "type": "Feature", "geometry": {"type": "LineString", "coordinates": [[102.0, 0.0], [103.0, 1.0], [104.0, 0.0], [105.0, 1.0]]}, "properties": {"prop0": "value0", "prop1": 0.0}}
^^{ "type": "Feature", "geometry": {"type": "Polygon", "coordinates": [[[100.0, 0.0], [101.0, 0.0], [101.0, 1.0], [100.0, 1.0], [100.0, 0.0]]]}, "properties": {"prop0": "value0", "prop1": {"this": "that"}}}
```

## Generating Docs

### Dependencies

 * [`xml2rfc`](https://pypi.python.org/pypi/xml2rfc/), a Python module
 * [`pandoc2rfc`](https://raw.github.com/miekg/pandoc2rfc/master/pandoc2rfc)
 * pandoc

### Transform Markdown to XML etc.

Inside the working copy of the repo perform (current practice):

```bash
bash ./build
```

which does the following:

```bash
bash pandoc2rfc -R -t template.xml -x transform.xsl back.mkd middle.mkd && mv draft.txt draft-unpaginated.txt && for i in H N T X; do bash pandoc2rfc -$i -t template.xml -x transform.xsl back.mkd middle.mkd; done
```
