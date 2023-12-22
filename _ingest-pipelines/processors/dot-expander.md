---
layout: default
title: Dot expander
parent: Ingest processors
nav_order: 65
---

# Dot expander 

The `dot_expander` processor transforms fields containing dots into object fields, making them accessible to other processors in the pipeline. Without this transformation, fields with dots cannot be processed.

The following is the syntax for the `dot_expander` processor:

```json
{
  "dot_expander": {
    "field": "field.to.expand" 
  }
}
```
{% include copy-curl.html %}

## Configuration parameters

The following table lists the required and optional parameters for the `dot_expander` processor.

Parameter | Required/Optional | Description |
|-----------|-----------|-----------|
`field`  | Required  | The field to expand into an object field. If set to `*`, all top-level fields will be expanded. |
`path` | Optional | The field is only required if the field to be expanded is nested within another object field. This is because the `field` parameter only recognizes leaf fields, which are fields that are not nested within any other objects. |
`override` | Optional | The field determines how the processor handles conflicts when expanding a field that overlaps with an existing nested object. Setting `override` to `false` instructs the processor to merge the conflicting values into an array, preserving both the original and expanded values. Conversely, setting `override` to `true` causes the processor to replace the existing nested object's value with the expanded field's value. |
`description`  | Optional  | A brief description of the processor. |
`if` | Optional | A condition for running this processor. |
`ignore_failure` | Optional | If set to `true`, failures are ignored. Default is `false`. |
`on_failure` | Optional | A list of processors to run if the processor fails. |
`tag` | Optional | An identifier tag for the processor. Useful for debugging to distinguish between processors of the same type. |

## Using the processor

Follow these steps to use the processor in a pipeline.

### Step 1: Create a pipeline

The following query expands two fields named `user.address.city` and `user.address.state` into nested objects named `city` and `state`: 

```json
PUT /_ingest/pipeline/dot-expander
{
  "description": "Dot expander processor",
  "processors": [
    {
      "dot_expander": {
        "field": "user.address.city"
      }
    },
    {
        "dot_expander":{
         "field": "user.address.state"
        }
    }
  ]
}
```
{% include copy-curl.html %}

### Step 2 (Optional): Test the pipeline

It is recommended that you test your pipeline before you ingest documents.
{: .tip}

To test the pipeline, run the following query:

```json
POST _ingest/pipeline/dot-expander/_simulate
{
  "docs": [
    {
      "_index": "testindex1",
      "_id": "1",
      "_source": {
        "field": "city, state"
      }
    }
  ]
}
```
{% include copy-curl.html %}

#### Response

The following example response confirms that the pipeline is working as expected:

```json
{
  "docs": [
    {
      "doc": {
        "_index": "testindex1",
        "_id": "1",
        "_source": {
          "field": "city, state"
        },
        "_ingest": {
          "timestamp": "2023-11-17T23:49:27.597933805Z"
        }
      }
    }
  ]
}
```

### Step 3: Ingest a document

The following query ingests a document into an index named `testindex1`:

```json
PUT testindex1/_doc/1?pipeline=dot-expander
{
  "field": "Denver, CO"
}
```
{% include copy-curl.html %}

### Step 4 (Optional): Retrieve the document

To retrieve the document, run the following query:

```json
GET testindex1/_doc/1
```
{% include copy-curl.html %}

#### Response

The following response confirms the document was indexed:

```json
{
  "_index": "testindex1",
  "_id": "1",
  "_version": 58,
  "_seq_no": 57,
  "_primary_term": 30,
  "found": true,
  "_source": {
    "field": "Denver, CO"
  }
}
```

## Nested fields

The `dot_expander` processor consolidates the `user.address.city` and `user.address.state` fields by merging with an existing `address`, `city`, and `state` field nested under `user`. If the field is a scalar value, then it will turn that field into an array. Take for example the following document:

```json
{
    "dot_expander": {
        "field": "user.address.city",
        "override": true
    }
}
```

With `override:false` (or unset), the dot-pathed top-level field gets appended to the target object node, so it transforms into:

```json
{
   "user": {
        "address": {
            "city": ["Denver", "Boulder"],
            "state": "CO"
        }
    }
}
```

With `override:true`, the dot-pathed top-level field value overwrites the target object node, so it transforms into:

```json
{
   "user": {
        "address": {
            "city": "Boulder",
            "state": "CO"
        }
    }
}
```

If the field value is set to a wildcard `*`, the processor expands all top-level dotted field names, for example:  

```json
{
    "dot_expander": {
        "field": "*"
    }
}
```

Take for example the following document:
Take for example the following document:

```json
{
    "user.address.city": "Denver",
    "user.address.state": "CO"
}
```

The `dot_expander` processor transforms that document into the following fields, all nested under `user`:

```json
{
    "user": {
        "address": {
            "city": "Denver"
        }
    },
    "user": {
        "address": {
            "state": "CO"
        }
    }
}
```

If the field is nested within a structure without dots, use the `path` parameter to navigate the non-dotted structure, for example:

```json
{
    "dot_expander": {
        "path": "user.address",
        "field": "*"
    }
}
```

Take for example the following document: 

```json
{
    "user": {
        "address": {
            "city.one": "Denver",
            "city.two": "Houston",
            "state.one": "CO",
            "state.two": "TX"
        }
    }
}
```

The `dot_expander` processor transforms the document into:

```json
{
    "user": {
        "address": {
            "city": {
                "one": "Denver",
                "two": "Houston"
            },
            "state": {
                "one": "CO",
                "two": "TX"
            }
        } 
    }
}
```

To ensure proper expansion of the `user.address.city` and `user.address.state` fields and handle conflicts with pre-existing fields, use a similar configuration as the following document: 

```json
{
    "user": {
        "address": {
            "city": "Denver",
            "state": "CO"
        }
    }
}
```

To ensure the correct expansion of the `city` and `state` fields, the following pipeline uses the `rename` processor to prevent conflicts and allow for proper handling of scalar fields during expansion.

```json
{
    "processors": [
      {
        "rename": {
            "field": "user.address",
            "target_field": "user.address.original"
        }
      },
      {
        "dot_expander": {
            "field": "user.address.original"
        }
      }  
    ]
}
```
