# niet

[![Build Status](https://travis-ci.org/openuado/niet.svg?branch=master)](https://travis-ci.org/openuado/niet)
![PyPI](https://img.shields.io/pypi/v/niet.svg)
![PyPI - Python Version](https://img.shields.io/pypi/pyversions/niet.svg)
![PyPI - Status](https://img.shields.io/pypi/status/niet.svg)

Get data from yaml file directly in your shell

Niet is like [xmllint](http://xmlsoft.org/xmllint.html) or
[jq](https://stedolan.github.io/jq/) but for YAML and JSON data -
you can use it to slice and filter and map and transform structured data.

You can easily retrieve data by using simple expressions or using
xpath advanced features to access non-trivial data.

You can easily convert YAML format into JSON format and vice versa.

## Features
- Extract elements by using xpath syntax
- Extract values from json format
- Extract values from yaml format
- Automaticaly detect format (json/yaml)
- Read data from file or pass data from stdin
- Format output values
- Format output to be reused by shell `eval`
- Convert YAML to JSON
- Convert JSON to YAML

## Install or Update niet

```sh
$ pip install -U niet
```

## Requirements

- Python 2.7 / Python 3+

## Usage

### Help and options

```shell
$ niet --help
usage: niet [-h] [-f {json,yaml,eval,newline,ifs,squote,dquote}] [-s] [-v]
            object [file]

Read data from YAML or JSON file

positional arguments:
  object                Path to object separated by dot (.). Use '.' to get
                        whole file. eg: a.b.c
  file                  Optional JSON or YAML filename. If not provided niet
                        read from stdin

optional arguments:
  -h, --help            show this help message and exit
  -f {json,yaml,eval,newline,ifs,squote,dquote}, --format {json,yaml,eval,newline,ifs,squote,dquote}
                        output format
  -s, --silent          silent mode, doesn't display message when element was
                        not found
  -v, --version         print the Niet version number and exit (also
                        --version)

output formats:
  json  Return object in JSON
  yaml  Return object in YAML
  eval  Return result in a string evaluable by a shell eval command as an input
  newline       Return all elements of a list in a new line
  ifs   Return all elements of a list separated by IFS env var
  squote        Add single quotes to result
  dquote        Add double quotes to result
```

### With Json from stdin

```shell
$ echo '{"foo": "bar", "fizz": {"buzz": ["1", "2", "Fizz", "4", "Buzz"]}}' | niet fizz.buzz
1
2
Fizz
4
Buzz
$ echo '{"foo": "bar", "fizz": {"buzz": ["1", "2", "Fizz", "4", "Buzz"]}}' | niet fizz.buzz -f squote
'1' '2''Fizz' '4' 'Buzz'
$ echo '{"foo": "bar", "fizz": {"buzz": ["1", "2", "fizz", "4", "buzz"]}}' | niet . -f yaml
fizz:
  buzz:
  - '1'
  - '2'
  - fizz
  - '4'
  - buzz
foo: bar
$ echo '{"foo": "bar", "fizz": {"buzz": ["zero", "one", "two", "three"]}}' | niet "fizz.buzz[2]"
two
$ echo '{"foo": "bar", "fizz": {"buzz": ["zero", "one", "two", "three"]}}' | niet -f dquote "fizz.buzz[0:2]"
"zero" "one"
$ echo '{"foo": "bar", "fizz": {"buzz": ["zero", "one", "two", "three"]}}' | niet -f dquote "fizz.buzz[:3]"
"zero" "one" "two"

```

### With YAML file

Consider the yaml file with the following content:
```yaml
# /path/to/your/file.yaml
project:
    meta:
        name: my-project
    foo: bar
    list:
        - item1
        - item2
        - item3
    test-dash: value
```

You can [download the previous example](https://gist.githubusercontent.com/4383/53e1599663b369f499aa28e27009f2cd/raw/389b82c19499b8cb84a464784e9c79aa25d3a9d3/file.yaml) locally for testing purpose or use the command line for this:
```shell
wget https://gist.githubusercontent.com/4383/53e1599663b369f499aa28e27009f2cd/raw/389b82c19499b8cb84a464784e9c79aa25d3a9d3/file.yaml
```

You can retrieve data from this file by using niet like this:
```sh
$ niet ".project.meta.name" /path/to/your/file.yaml
my-project
$ niet ".project.foo" /path/to/your/file.yaml
bar
$ niet ".project.list" /path/to/your/file.yaml
item1 item2 item3
$ # assign return value to shell variable
$ NAME=$(niet ".project.meta.name" /path/to/your/file.yaml)
$ echo $NAME
my-project
$ niet project.'"test-dash"' /path/to/your/file.json
value
```

### With JSON file

Consider the json file with the following content:
```json
{
    "project": {
        "meta": {
            "name": "my-project"
        },
        "foo": "bar",
        "list": [
            "item1",
            "item2",
            "item3"
        ],
        "test-dash": "value"
    }
}
```

You can [download the previous example](https://gist.githubusercontent.com/4383/1bab8973474625de738f5f6471894322/raw/0048cd2310df2d98bf4f230ffe20da8fa615cef3/file.json) locally for testing purpose or use the command line for this:
```shell
wget https://gist.githubusercontent.com/4383/1bab8973474625de738f5f6471894322/raw/0048cd2310df2d98bf4f230ffe20da8fa615cef3/file.json
```

You can retrieve data from this file by using niet like this:
```sh
$ niet "project.meta.name" /path/to/your/file.json
my-project
$ niet "project.foo" /path/to/your/file.json
bar
$ niet "project.list" /path/to/your/file.json
item1 item2 item3
$ # assign return value to shell variable
$ NAME=$(niet "project.meta.name" /path/to/your/file.json)
$ echo $NAME
my-project
$ niet project.'"test-dash"' /path/to/your/file.json
value
```

### Object Identifiers

An identifier is the most basic expression and can be used to extract a single
element from a JSON/YAML document. The return value for an identifier is
the value associated with the identifier. If the identifier does not
exist in the JSON/YAML document, than niet display a specific message and
return the error code `1`, example:

```sh
$ echo '{"foo": "bar", "fizz": {"buzz": ["1", "2", "3"]}}' | niet fizz.gogo
Element not found: fizz.gogo
$ echo $?
1
```

See the [related section](#deal-with-errors) for more info on how to manage
errors with `niet`.

Niet is based on `jmespath` to find results so for complex research you can
refer to the [jmespath specifications](http://jmespath.org/specification.html#identifiers)
to use identifiers properly.

If you try to search for an identifier who use some dash you need to surround
your research expression with simple and double quotes, examples:

```sh
$ echo '{"foo-biz": "bar", "fizz": {"buzz": ["zero", "one", "two", "three"]}}' | niet -f dquote '"foo-biz"'
bar
$ echo '{"key-test": "value"}' | niet '"key-test"'
value
```

However, `niet` will detect related issues and surround automatically your
identifier if `jmespath` fail to handle it.

Hence, the following examples will return similar results than the previous
examples:

```sh
$ echo '{"foo-biz": "bar", "fizz": {"buzz": ["zero", "one", "two", "three"]}}' | niet -f dquote foo-biz
bar
$ echo '{"key-test": "value"}' | niet key-test
value
```

If your object is not at the root of your path, an example is available in
`tests/sample/sample.json`, then you need to only surround the researched
identifier like this `project.'"test-dash"'`

```json
{
    "project": {
        "meta": {
            "name": "my-project"
        },
        "foo": "bar",
        "list": [
            "item1",
            "item2",
            "item3"
        ],
        "test-dash": "value"
    }
}

```

Example:
```sh
niet project.'"test-dash"' tests/sample/sample.json
```

Further examples with [`jmespath` identifiers](http://jmespath.org/specification.html#examples).

### Output formats 
You can change the output format using the -f or --format optional 
argument. 

By default, niet detect the input format and display complex objects
in the same format. If the object is a list or a value, newline output
format will be used.

Output formats are: 
  - ifs
  - squote
  - dquote
  - newline
  - yaml
  - json

#### ifs
Ifs output format print all values of a list or a single value in one line.
All values are separated by the content of IFS environment variable if defined,
space otherwise.

Examples (consider the previous [YAML file example](#with-yaml-file)):
```shell
$ IFS="|" niet .project.list /path/to/your/file.yaml -f ifs
item1|item2|item3
$ IFS=" " niet .project.list /path/to/your/file.yaml -f ifs
item1 item2 item3
$ IFS="@" niet .project.list /path/to/your/file.yaml -f ifs
item1@item2@item3
```

This is usefull in a shell for loop,
but your content must, of course, don't contain IFS value:
```shell
OIFS="$IFS"
IFS="|"
for i in $(niet .project.list /path/to/your/file.yaml -f ifs); do
    echo ${i}
done
IFS="${OIFS}"
```

Previous example provide the following output:
```sh
item1
item2
item3
```

For single quoted see [squote](#squote) ouput or [dquote](#dquote) double quoted output with IFS

#### squote
Squotes output format print all values of a list or a single value in one line.
All values are quoted with single quotes and are separated by IFS value.

Examples (consider the previous [YAML file example](#with-yaml-file)):
```shell
$ # With the default IFS
$ niet .project.list /path/to/your/file.yaml -f squote
'item1' 'item2' 'item3'
$ # With a specified IFS
$ IFS="|" niet .project.list /path/to/your/file.yaml -f squote
'item1'|'item2'|'item3'
```

#### dquote
Dquotes output format print all values of a list or a single value in one line.
All values are quoted with a double quotes and are separated by IFS value.

Examples (consider the previous [YAML file example](#with-yaml-file)):
```shell
$ # With the default IFS
$ niet .project.list /path/to/your/file.yaml -f dquote
'item1' 'item2' 'item3'
$ # With a specified IFS
$ IFS="|" niet .project.list /path/to/your/file.yaml -f dquote
"item1"|"item2"|"item3"
```

#### newline
Newline output format print one value of a list or a single value per line.
This format is usefull using shell while read loop. eg:
```sh
while read value: do
    echo $value
done < $(niet --format newline project.list your-file.json)
```

#### eval

Eval output format allow you to eval output string to initialize shell
variable generated from your JSON/YAML content.

You can intialize shell variables from your entire content, example:

```sh
$ echo '{"foo-biz": "bar", "fizz": {"buzz": ["zero", "one", "two", "three"]}}' | niet -f eval .
 foo_biz="bar";fizz__buzz=( zero one two three )
$ eval $(echo '{"foo-biz": "bar", "fizz": {"buzz": ["zero", "one", "two", "three"]}}' | niet -f eval .)
$ echo ${foo_biz}
bar
$ echo ${fizz__buzz}
zero one two three
$ eval $(echo '{"foo-biz": "bar", "fizz": {"buzz": ["zero", "one", "two", "three"]}}' | niet -f eval '"foo-biz"'); echo ${foo_biz}
bar
$ echo '{"foo-biz": "bar", "fizz": {"buzz": ["zero", "one", "two", "three"]}}' | niet -f eval fizz.buzz
fizz_buzz=( zero one two three );
```

Parent elements are separated by `__` by example the `fizz.buzz` element
will be represented by a variable named `fizz__buzz`. You need to consider
that when you call your expected variables.

Also you can initialize some shell array from your content and loop over in
a shell maner:

```sh
$ eval $(echo '{"foo-biz": "bar", "fizz": {"buzz": ["zero", "one", "two", "three"]}}' | niet -f eval fizz.buzz)
$ for el in ${fizz_buzz}; do echo $el; done
zero
one
two
three
```
 
#### yaml
Yaml output format force output to be in YAML regardless the input file format.

#### json
Json output format force output to be in JSON regardless the input file format.

### Result not found

By default when no results was found niet display a specific message and return
the error code `1`, example:
```sh
$ echo '{"foo": "bar", "fizz": {"buzz": ["1", "2", "3"]}}' | niet fizz.gogo
Element not found: fizz.gogo
$ echo $?
1
```

You can avoid this behavior by passing niet into a silent mode.

Silent mode allow you to hide the specific message error but continue to return
a status code equal to `1` when the key was not found.

You can use the silent mode by using the flag `-s/--silent`, example:
```sh
$ echo '{"foo": "bar", "fizz": {"buzz": ["1", "2", "3"]}}' | niet fizz.gogo -s
$ echo $?
1
```

### Deal with errors

When your JSON file content are not valid niet display an error and exit
with return code `1`

You can easily protect your script like this:
```sh
PROJECT_NAME=$(niet project.meta.name your-file.yaml)
if [ "$?" = "1" ]; then
    echo "Error occur ${PROJECT_NAME}"
else
    echo "Project name: ${PROJECT_NAME}"
fi
```

## Examples

You can try niet by using the samples provided with the project sources code.

> All the following examples use the sample file available in niet sources code
at the following location `tests/samples/sample.yaml`.

Sample example:
```yaml
# tests/samples/sample.yaml
project:
    meta:
        name: my-project
    foo: bar
    list:
        - item1
        - item2
        - item3
```

Retrieve the project name:
```sh
$ niet project.meta.name tests/samples/sample.yaml
my-project
```

Deal with list of items
```sh
$ for el in $(niet project.list tests/samples/sample.yaml); do echo ${el}; done
item1
item2
item3
```

Also you can `eval` your `niet` output to setput some shell variables
that you can reuse in your shell scripts, the following example is similar to
the previous example but make use of the eval ouput format (`-f eval`):

```sh
$ eval $(niet -f eval project.list tests/samples/sample.yaml)
$ for el in ${project__list}; do echo $el; done
zero
one
two
three
```

### Transform JSON to YAML

With niet you can easily convert your JSON to YAML
```shell
$ niet . tests/samples/sample.json -f yaml
project:
  foo: bar
  list:
  - item1
  - item2
  - item3
  meta:
    name: my-project
```

### Transform YAML to JSON

With niet you can easily convert your YAML to JSON
```shell
$ niet . tests/samples/sample.yaml -f json
{
    "project": {
        "meta": {
            "name": "my-project"
        },
        "foo": "bar",
        "list": [
            "item1",
            "item2",
            "item3"
        ]
    }
}
```

## Tips

You can pass your search with or without quotes like this:
```sh
$ niet project.meta.name your-file.yaml
$ niet "project.meta.name" your-file.yaml
```

## Contribute

If you want to contribute to niet [please first read the contribution guidelines](CONTRIBUTING.md)

## Licence

This project is under the MIT License.

[See the license file for more details](LICENSE)
