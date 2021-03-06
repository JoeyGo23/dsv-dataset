# dsv-dataset
A metadata specification and parsing library for data sets.

One of the many recurring issues in data visualization is parsing data sets. Data sets are frequently represented in a delimiter-separated value (DSV) format, such as [comma-separated value (CSV)](https://en.wikipedia.org/wiki/Comma-separated_values) or tab-separated value (TSV). Conveniently, the [d3-dsv](https://github.com/d3/d3-dsv) library supports parsing such data sets. However, the resulting parsed data table has string values for every column, and it is up to the developer to parse those string values into numbers or dates, depending on the data.

The primary purpose of this library is to provide a way to annotate DSV data sets with type information about their columns, so they can be automatically parsed. This enables developers to shift the logic of how to parse columns out of visualization code, and into a separate metadata specification.

## Installation

Install via NPM: `npm install dsv-dataset`

Require the library via Node.js / Browserify:

```javascript
var dsvDataset = require("dsv-dataset");
```

You can also require the library via Bower: `bower install dsv-dataset`. The file `bower_components/dsv-dataset/dsv-dataset.js` contains a [UMD](https://github.com/umdjs/umd) bundle, which can be included via a `<script>` tag, or using [RequireJS](http://requirejs.org/).

## Example

Here is an example program that parses three columns from the [Iris dataset](https://archive.ics.uci.edu/ml/datasets/Iris).

```javascript
// This string contains CSV data that could be loaded from a .csv file.
var dsvString = [
  "sepal_length,sepal_width,petal_length,petal_width,class",
  "5.1,3.5,1.4,0.2,setosa",
  "6.2,2.9,4.3,1.3,versicolor",
  "6.3,3.3,6.0,2.5,virginica"
].join("\n");

// This metadata object specifies the delimiter and column types.
// This could be loaded from a .json file.
var metadata = {
  delimiter: ",",
  columns: [
    { name: "sepal_length", type: "number" },
    { name: "sepal_width",  type: "number" },
    { name: "petal_length", type: "number" },
    { name: "petal_width",  type: "number" },
    { name: "class",        type: "string" }
  ]
};

// Use dsv-dataset to parse the data.
var data = dsvDataset.parse(dsvString, metadata);

// Pretty-print the parsed data table as JSON.
console.log(JSON.stringify(data, null, 2));
```
The following JSON will be printed:
```json
[
  {
    "sepal_length": 5.1,
    "sepal_width": 3.5,
    "petal_length": 1.4,
    "petal_width": 0.2,
    "class": "setosa"
  },
  {
    "sepal_length": 6.2,
    "sepal_width": 2.9,
    "petal_length": 4.3,
    "petal_width": 1.3,
    "class": "versicolor"
  },
  {
    "sepal_length": 6.3,
    "sepal_width": 3.3,
    "petal_length": 6,
    "petal_width": 2.5,
    "class": "virginica"
  }
]
```
Notice how numeric columns have been parsed to numbers.

## API

<a name="parse" href="#parse">#</a> <i>dsvDataset</i>.<b>parse</b>(<i>dsvString</i>, <i>metadata</i>)

Parses the given DSV string using the given metadata. Returns the parsed data table.

Arguments:

 * `dsvString` The data table represented in DSV format. This will be parsed by [d3-dsv](https://github.com/d3/d3-dsv).
 * `metadata` An object that annotates the DSV data table with metadata, with properties
   * `delimiter` (string - single character) The delimiter used between values. Typical values are
     * `","` (CSV)
     * `"\t"` (TSV)
     * `"|"`.
   * `columns` (array of objects) An array of column descriptor objects with properties
     * `name` (String) The column name found on the first line of the DSV data set.
     * `type` (String - one of `"string"`, `"number"` or `"date"`) The type of this column.
       * If `type` is `"number"`, then [`parseFloat`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseFloat) will parse the string.
       * If `type` is `"date"`, then [moment(String)](http://momentjs.com/docs/#/parsing/string/) will parse the string.
       * If no type is specified, the default is "string".

## Future Plans

A future goal of this project is to provide recommentations for how descriptive metadata can be added to data sets. This includes human-readable titles and descriptions for data sets and columns. This metadata can be surfaced in visualizations to provide a nicer user experience. For example, the human-readable title for a column can be used as an axis label (e.g. "Sepal Width"), rather than the not-so-nice column name from the original DSV data (e.g. "sepal_width").

The `metadata` object will have the following optional properties:

 * `title` (string) A human readable name for the data set.
 * `description` (string - Markdown) A human readable free text description of the data set. This can be Markdown, so can include links. The length of this should be about one paragraph.
 * `sourceURL` (string - URL) The URL from which the data set was originally downloaded.

Each entry in the `columns` array will have the following optional properties:

 * `title` (string) A human readable name for the column. Should be a single word or as few words as possible. Intended for use on axis labels and column selection UI widgets.
 * `description` (string - Markdown) A human readable free text description of the data set. This can be Markdown, so can include links. The length of this should be about one sentence, and should communicate the meaning of the column to the user. Intended for use in tooltips when hovering over axes in a visualization, and for entries in user interfaces for selecting columns (e.g. dropdown menu or drag & drop column list).
