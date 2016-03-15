# GeoJSON Feature Sequences

A proposed standard for incrementally parseable GeoJSON data

[![Circle CI](https://circleci.com/gh/geojson/geojson-text-sequences.svg?style=svg)](https://circleci.com/gh/geojson/geojson-text-sequences)

Large JSON files pose a problem that is well explained in the motivation for
the [JSON Text Sequences RFC 7464](https://tools.ietf.org/html/rfc7464).

> The JavaScript Object Notation (JSON) [RFC7159] is a very handy
serialization format.  However, when serializing a large sequence of
values as an array, or a possibly indeterminate-length or
never-ending sequence of values, JSON becomes difficult to work with.
>
> Consider a sequence of one million values, each possibly 1 kilobyte
when encoded -- roughly one gigabyte.  It is often desirable to
process such a dataset in an incremental manner: without having to
first read all of it before beginning to produce results.
Traditionally the way to do this with JSON is to use a "streaming"
parser, but these are neither widely available, widely used, nor easy
to use.

GeoJSON has exactly this problem with sequences of features, amplified by the
large arrays of coordinates of each feature.

JSON Text Sequences provides a standard for dealing with this problem.

> This document describes the concept and format of "JSON text
sequences", which are specifically not JSON texts themselves but are
composed of (possible) JSON texts.  JSON text sequences can be parsed
(and produced) incrementally without having to have a streaming
parser (nor streaming encoder).

GeoJSON Feature Sequences are a straightforward adaptation of JSON Text
Sequences. A GeoJSON Feature Sequence is a document containing, instead of
a single GeoJSON Feature Collection, multiple GeoJSON Feature texts that can be
parsed and produced incrementally.

In a nutshell, GeoJSON Feature Sequences proposes to "explode" the
following GeoJSON Feature Collection,

```
{ "type": "FeatureCollection",
  "features": [
    { "type": "Feature",
      "geometry": {"type": "Point", "coordinates": [102.0, 0.5]},
      "properties": {"prop0": "value0"}},
    { "type": "Feature",
      "geometry": {
        "type": "LineString",
        "coordinates": [
          [102.0, 0.0], [103.0, 1.0], [104.0, 0.0], [105.0, 1.0]]},
      "properties": {
        "prop0": "value0",
        "prop1": 0.0}},
    { "type": "Feature",
       "geometry": {
         "type": "Polygon",
         "coordinates": [
           [[100.0, 0.0], [101.0, 0.0], [101.0, 1.0],
            [100.0, 1.0], [100.0, 0.0] ]]},
       "properties": {
         "prop0": "value0",
         "prop1": {"this": "that"}}}]}
```

into a document containing 3 separate JSON texts delimited by a leading ASCII
character RS "0x1e" (represented by "RS" below).

```
RS{ "type": "Feature", "geometry": {"type": "Point",
"coordinates": [102.0, 0.5]}, "properties": {"prop0": "value0"}}
RS{ "type": "Feature", "geometry": {"type": "LineString",
"coordinates": [[102.0, 0.0], [103.0, 1.0], [104.0, 0.0],
[105.0, 1.0]]}, "properties": {"prop0": "value0", "prop1": 0.0}}
RS{ "type": "Feature", "geometry": {"type": "Polygon",
"coordinates": [[[100.0, 0.0], [101.0, 0.0], [101.0, 1.0],
[100.0, 1.0], [100.0, 0.0]]]}, "properties":
{"prop0": "value0", "prop1": {"this": "that"}}}
```

Please note that this last representation is not GeoJSON anymore, because it
uses more than one GeoJSON object. It is a "GeoJSON Feature Sequence", which
has to be parsed accoring to the rules for JSON Text Sequences, and then can be
interpreted as a sequence of individual GeoJSON feature objects.

## Difference from "Line-delimited JSON"

RFC 7464 (JSON Text Sequences) defines how parsers should behave on
encountering malformed JSON records and allows for pretty-printed records,
while approaches such as the ones described in
https://en.wikipedia.org/wiki/JSON_Streaming#Line_delimited_JSON do not.
