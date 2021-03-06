# node-js2xmlparser #

## Overview ##

js2xmlparser is a Node.js module that parses JavaScript objects into XML.

## Features ##

Since XML is a data-interchange format, js2xmlparser is designed primarily for JSON-type objects, arrays and primitive
data types, like many of the other JavaScript to XML parsers currently available for Node.js.

However, js2xmlparser is capable of parsing any object, including native JavaScript objects such as Date and RegExp, by
taking advantage of each object's toString function. Functions are a special case where the return value of the function
itself is used instead of the toString function, if available.

js2xmlparser also supports a number of constructs unique to XML:

* attributes (through a unique attribute property in objects)
* mixed content (through a unique value property in objects)
* multiple elements with the same name (through arrays)

js2xmlparser can also pretty-print the XML it outputs with the option of customizing the indent string.

## Installation ##

The easiest way to install js2xmlparser is to use npm: `npm install js2xmlparser`.

Alternatively, you may download the source from GitHub and copy it to a folder named "js2xmlparser" within your
"node_modules" directory.

## Usage ##

The js2xmlparser module contains one function which takes the following arguments:

* `root` - string representing the XML root element's name
* `data` - object or JSON string to be converted to XML
* `options` - options object (optional)
    * `callFunctions` - if true, calls member functions (with no arguments) and includes their return value in the XML;
      if false, returns the text of the function (optional, default is true)
    * `wrapArray` - array wrapping options object (optional)
        * `enabled` - if true, all elements in an array will be added to a single XML element as child elements; if
          false, array elements will be placed in their own XML elements (optional, default is false)
        * `elementName` - name of XML child elements when array wrapping is enabled (optional, default is "item")
    * `useCDATA` - if true, all strings are enclosed in CDATA tags instead of escaping illegal XML characters (optional,
      default is false)
    * `convertMap` - object mapping certain types of objects (as given by `Object.prototype.toString.call(<object>)`) to
      functions to convert those types of objects to a particular string representation; "*" can be used as a wildcard
      for all types of objects (optional, default is an empty object)
    * `declaration` - XML declaration options object (optional)
        * `include` - if true, includes an XML declaration (optional, default is true)
        * `encoding` - string representing the XML encoding for the corresponding attribute in the declaration; a value
          of null represents no encoding attribute (optional, default is "UTF-8")
    * `attributeString` - string representing attribute property (optional, default is "@")
    * `valueString` - string representing the value property (optional, default is "#")
    * `prettyPrinting` - pretty-printing options object (optional)
        * `enabled` - if true, pretty-printing is enabled (optional, default is true)
        * `indentString` - string representing the indent (optional, default is "\t")

## Example ##

The following example illustrates the basic usage of js2xmlparser:

    var js2xmlparser = require("js2xmlparser");

    var data = {
        "firstName": "John",
        "lastName": "Smith"
    };

    console.log(js2xmlparser("person", data));

    > <?xml version="1.0" encoding="UTF-8"?>
    > <person>
    >     <firstName>John</firstName>
    >     <lastName>Smith</lastName>
    > </person>

Here's a more complex example that builds on the first:

    var js2xmlparser = require("js2xmlparser");

    var data = {
        "firstName": "John",
        "lastName": "Smith",
        "dateOfBirth": new Date(1964, 07, 26),
        "address": {
            "@": {
                "type": "home"
            },
            "streetAddress": "3212 22nd St",
            "city": "Chicago",
            "state": "Illinois",
            "zip": 10000
        },
        "phone": [
            {
                "@": {
                    "type": "home"
                },
                "#": "123-555-4567"
            },
            {
                "@": {
                    "type": "cell"
                },
                "#": "456-555-7890"
            }
        ],
        "email": function() {return "john@smith.com";},
        "notes": "John's profile is not complete."
    };

    console.log(js2xmlparser("person", data));

    > <?xml version="1.0" encoding="UTF-8"?>
    > <person>
    >     <firstName>John</firstName>
    >     <lastName>Smith</lastName>
    >     <dateOfBirth>Wed Aug 26 1964 00:00:00 GMT-0400 (Eastern Daylight Time)</dateOfBirth>
    >     <address type="home">
    >         <streetAddress>3212 22nd St</streetAddress>
    >         <city>Chicago</city>
    >         <state>Illinois</state>
    >         <zip>10000</zip>
    >     </address>
    >     <phone type="home">123-555-4567</phone>
    >     <phone type="cell">456-555-7890</phone>
    >     <email>john@smith.com</email>
    >     <notes>John&apos;s profile is not complete.</notes>
    > </person>

Here's an example that makes use of array wrapping:

    var js2xmlparser = require("js2xmlparser");

    var data  = {
        "phone": [
            {
                "@": {
                    "type": "home"
                },
                "#": "123-555-4567"
            },
            {
                "@": {
                    "type": "cell"
                },
                "#": "456-555-7890"
            }
        ]
    };

    var options = {
        wrapArray: {
            enabled: true
        }
    };

    console.log(js2xmlparser("person", data, options));

    > <?xml version="1.0" encoding="UTF-8"?>
    > <person>
    >     <phone>
    > 	      <item type="home">123-555-4567</item>
    > 	      <item type="cell">456-555-7890</item>
    >     </phone>
    > </person>

Here's an example that wraps strings in CDATA tags instead of escaping invalid characters.

    var js2xmlparser = require("js2xmlparser");

    var data = {
        "notes": "John's profile is not complete."
    };

    var options = {
        useCDATA: true
    };

    console.log(js2xmlparser("person", data, options));

    > <?xml version="1.0" encoding="UTF-8"?>
    > <person>
    >     <notes><![CDATA[John's profile is not complete.]]></notes>
    > </person>

Here's an example that uses the convert map feature:

    var js2xmlparser = require("js2xmlparser");

    var data = {
        "dateOfBirth": new Date(1964, 7, 26)
    };

    var options = {
        convertMap: {
            "[object Date]": function(date) {
                return date.toISOString();
            }
        }
    };

    console.log(js2xmlparser("person", data, options));

    > <?xml version="1.0" encoding="UTF-8"?>
    > <person>
    >     <dateOfBirth>1964-08-26T04:00:00.000Z</dateOfBirth>
    > </person>
