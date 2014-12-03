# AWS CloudFormation

> AWS CloudFormation gives developers and systems administrators an easy way to create and manage a collection of related AWS resources, provisioning and updating them in an orderly and predictable fashion

This repository is an example of simplified CloudFormation documents. They're written in [YAML](http://www.yaml.org/) to help aid simplifying the understanding of CloudFormation.

CloudFormation needs to be written in the [JSON](http://json.org/) format, so we provide a [simple example script](#convert-yaml-into-json) to allow the use of YAML during development and JSON for pushing to production.

## YAML

YAML is very simple to write; review the following example:

```yaml
foo: bar
baz:
  - "a"
  - "b"
  - "c"
  - 
    - "x"
    - "y"
    - "z"
  - another_key: "another value"
```

...which equates to the following JSON...

```json
{
   "baz" : [
      "a",
      "b",
      "c",
      [
         "x",
         "y",
         "z"
      ],
      {
         "another_key" : "another value"
      }
   ],
   "foo" : "bar"
}
```

Notice that by default all key/values are evaluated into the relevant key/value of a standard JavaScript object when converted from YAML into JSON. Also, any reference to `-` indicates that when converted from YAML to JSON the data will become an item within an Array. 

In the above example we also demonstrate nesting an Array and an Object within an Array.

## Understanding CloudFormation

CloudFormation is easy to write but difficult to understand and to memorise. The best way to use CloudFormation is to read the AWS documentation and to write your requirements from scratch, otherwise you'll find that by copy-and-pasting from existing CloudFormation can cause confusion and mis-understanding.

There are also AWS functions provided that allow us to inject data dynamically into our CloudFormation document.These include getting attributes (such as the ARN) from a Resource. For a full list of functions see [the AWS reference](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference.html)

## Convert YAML into JSON

```sh
ruby \
  -rjson \
  -ryaml \
  -e \
  "puts JSON.generate(YAML.load_file('EC2.yml'))" | \
  json_pp
```

...or as a one-liner...

```sh
ruby -rjson -ryaml -e "puts JSON.generate(YAML.load_file('EC2.yml'))" | json_pp
```

...or an alternative...

```sh
cat EC2.yml | ruby -rjson -ryaml -e "puts YAML.load(STDIN.read).to_json" | json_pp
```

## YAML Features

YAML provides a couple of features to help reduce complexity and duplication.

To reduce complexity, YAML will allow values to not be quoted. What this means 
is that YAML will intelligently recognise the data type of your content. So if 
you write `123` then it'll interpret this as an integer, but if you write `abc` 
then it'll interpret that as a String. But there is one item worth noting: if 
you write `Yes` then YAML will interpret that as `true`. If you really mean `Yes` 
to be a String then either write it as `"Yes"` (i.e. quoted) or use YAML's 
explicit data typing feature: `!!str Yes` (which is ugly, so best use quotes).

To reduce duplication, YAML provides an "anchor" (`&`) which let's us tag a key 
and to then "reference" (`*`) that tag later on in our document. We demonstrate 
this in the below YAML example file (and in the JSON conversion after it).

> Note: YAML also provides a way to insert whole chunks of content using `<<` (e.g. `<<: *reference`). You can even use an Array to insert multiple references `<<: [ *batch_data, *whitelist ]`

```yaml
my_json:
  # Value is { "bar": "baz", "qux": "qiz" }
  foo: &my_alias
    bar: baz
    qux: qiz

  # Value is { "bar": "baz", "qux": "qiz" }
  abc: *my_alias

  # Value is ["foo", "bar", "baz"]
  an_array: &array_alias
    - &single_element_alias foo
    - bar
    - baz

  # Value is the copied Array ["foo", "bar", "baz"]
  xyz: *array_alias

  # Value is set to "foo"
  beep: *single_element_alias

  # Set data type to String (explicitly)
  hey: !!str there

  # Set data type to Integer (explicitly)
  integer_age: 123
  string_age: !!str 123

  # Set data type to Boolean (explicitly)
  boolean_yes: Yes
  string_yes: !!str Yes

# --- # the three hyphens denote a separate document,
      # and so the following isn't processed when
      # converting this entire file to json

  another_array:
    - 1
    - 2

```

When the above YAML file is converted into JSON it looks like the following:

```json
{
   "my_json" : {
      "another_array" : [
         1,
         2
      ],
      "integer_age" : 123,
      "hey" : "there",
      "string_yes" : "Yes",
      "beep" : "foo",
      "boolean_yes" : true,
      "string_age" : "123",
      "abc" : {
         "bar" : "baz",
         "qux" : "qiz"
      },
      "an_array" : [
         "foo",
         "bar",
         "baz"
      ],
      "foo" : {
         "bar" : "baz",
         "qux" : "qiz"
      },
      "xyz" : [
         "foo",
         "bar",
         "baz"
      ]
   }
}
```
