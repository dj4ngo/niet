# Examples of niet usage

## Get values

## File manipulation

### Transform YAML to JSON
With niet you can easily convert your YAML to JSON
<!--- TEST START -->
```shell
$ niet . tests/samples/sample.yaml -f json
```
<!--- TEST START -->

Output:
```shell
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

### Indent JSON file

This is an example of how to indent a JSON file :
```shell
$ niet . tests/samples/sample_not_indented.json 
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
