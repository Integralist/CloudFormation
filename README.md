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
