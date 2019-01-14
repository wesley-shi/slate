---
title: API Reference

includes:
  - errors

search: true
---

# Introduction

Welcome to the Cresta Taxonomy API! This is a work in progress powered by [Slate](https://github.com/lord/slate).

The goal of these APIs are to support write labeling functions applied to text input and returning a sub-taxonomy output. 

Note: I have the id/keys in the examples below as strings ex. "att_agent-actions.open" but they can be hashed to an ID. I think XXX (taxonomy) XXXXX (parent key hash) XXXXX (own name/key hash) or something could work if we want the digits to be non-random.

# Authentication

Perhaps we can probably just use an API key for now? Something like setting up session tokens based user auth could be an option later.

> Use API key in an authorization header

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: API_KEY"
```

```javascript
```

> Make sure to replace `API_KEY` with your API key.

# Taxonomy API

## POST yaml

> The above command should return the JSON response in GET taxonomy tree for the posted YAML tree

Uploads a YAML file into nodes to then be queried. See taxonomy YAMLs for samples.

### HTTP Request

`GET https://agile-binder-228011.appspot.com/post/yaml`

### URL Parameters

Parameter | Description
--------- | -----------
yaml | The YAML file to be posted
name | The name of the tree, if an existing tree already exists, the nodes will be overwritten and labeling function will be reassigned

## GET taxonomy list

As someone working on labeling functions for taxonomies, I need to know what taxonomies exist. The names of the lists are created by the user through the POST yaml API.

> The above command returns JSON structured like this:

```json
[{
    "key": "att_agent-actions",
},
{
    "key": "att_visitor-actions",
}]
```

### HTTP Request

`GET https://agile-binder-228011.appspot.com/get/taxonomy/list`

## GET taxonomy tree

Retrieves a taxonomy tree based on the name of the root.

In the future if graphs are ever very large, a node's key could be provided to retrieve only a sub-section of a taxonomy, the JSON return values could also be node's key (requiring a get/node/<key> operation to fetch details). For now, the JSON response should be a list of nodes that can be parsed into a tree structure with all complete values in the response. I imagine in the future filters and other options could be provided to only fetch the data we need.

> The above command returns JSON structured like this:

```json
// Response
{
    "label": "",
    "key": "att_agent-actions", // An id/key uniquely identifying each node
    "description": "AT&T Agent Actions",
    "example": [],
    "type": 0, // 0 = root node
    "children": [{
        "label": "open",
        "key": "att_agent-actions.open",
        "description": "agent enters chat.",
        "example": ["Agent yc4055 enters chat as Lauren"],
        "type": 2,
        "parent_key": "att_agent-actions"
        "children": [{
            "parent_key": "att_agent-actions.open",
            "key": "att_agent-actions.open.labeling_function_name_assigned_by_user",
            "description": "Labeling function description assigned by user",
            "function": "function as a string in readable format, can be editable",
            "type": "labeling_function",
            "data": { // An object representing metadata or test results from testing
            "last_run_date": <timestamp in ms>,
            "date_created": <timestamp in ms>,
            "subtaxonomy_run_results": { // An object representing text and subtaxonomy results },
            "author": <user id>
            "metadata": { // An object with additional relevant data }
        }
    }],
    },
    {
        "label": "greeting",
        "key": "att_agent-actions.greeting",
        "description": "agent greets visitor.",
        "example": [],
        "type": 1,
        "parent_key": "att_agent-actions"
        "children": [{
            ...
        }],
        }
    ],
    "parent_key": ""
}
```

### HTTP Request

`GET https://agile-binder-228011.appspot.com/get/taxonomy/key/<key>`

### URL Parameters

Parameter | Description
--------- | -----------
key | The key of the taxonomy node to retrieve along with its descendants

### Node Object JSON

Parameter | Description
--------- | -----------
create_date | Date created
update_date | Date last updated
owner | Username of the user to upload this node object
label | The label assigned to a node
key | The key assigned to a node. This should be unique and currently a key is the concatonation of the labels down from the root.
node_type | 0=root_node, 1=parent_node, 2=leaf_node, 3=labeling_function
description | A description of this node
parent_key | The parent's key. While the node key contains the parent by concatonation, this will be needed should our key format be altered
function | A string representing a function. Only present for labeling_function nodes
children | An array of nodes belonging to this parent node (optional)
example | An array of strings that to help a user better understand the description or label (optional)
data | An object representing metadata (optional)

## POST labeling function

As someone writing labeling functions, I need to add my labeling function to a database of functions so that a system can score the functions and others working with me can have access. I need to know that the labeling function is correctly scored and categorized under the most relevant sub-taxonomy. We can also have a list of utility functions with their APIs that are available in the cloud.

For the response, I think a 204 could just mean success but if there is a scored result, a 200 with the results would be nice.

### HTTP Request

`POST https://agile-binder-228011.appspot.com/post/functions`

### Function Object

Parameter | Description
--------- | -----------
function | The function as a string. Assume access to a variable `text` and return a JSON object of subtaxonomy to `-1|0|1`
taxonomy | The key of the taxonomy, this function is to be grouped under
name | A name to identify the labeling function
description | A description for the labeling function
id | The id of a functional that this function is meant to replace (optional)

### URL Parameters

Parameter | Description
--------- | -----------
functions | An array of function objects to be automatically run and labeled
run | A boolean flag to indicate whether snorkel should re-run data for taxonomy with these new functions

## Post labels

As someone managing how labeling functions are scored and categorized, I need to add pre-labeled data with labels that labeling functions will get scored against.

For the response, I think a 204 could just mean success but if there is a scored result, a 200 with the results would be nice.

### HTTP Request

`POST https://agile-binder-228011.appspot.com/post/labels`

### Label Object JSON

Parameter | Description
--------- | -----------
text | The message text
subtaxonomy | The key of the subtaxonomy this message is to be grouped under

### URL Parameters

Parameter | Description
--------- | -----------
labels | An array of label objects to posted 
run | A boolean flag to indicate whether snorkel should re-run labeling functions for subtaxonomies with new labels

## GET function

As someone managing how labeling functions are scored and categorized, I need to know how specific labeling functions performed against the correct sanitized inputs/outputs provided.

> The above command returns JSON structured like this:

```json
[{
    "parent_id": "att_agent-actions.open",
    "key": "att_agent-actions.open.labeling_function_name_assigned_by_user",
    "description": "Labeling function description assigned by user",
    "function": "function as a string in readable format, can be editable",
    "type": "labeling_function",
    "data": { // An object representing metadata or test results from testing
        "last_run_date": <timestamp in ms>,
        "date_created": <timestamp in ms>,
        "subtaxonomy_run_results": { // An object representing text and subtaxonomy results },
        "author": <user id>
        "metadata": { // An object with additional relevant data }
}],
```

### HTTP Request

`GET https://agile-binder-228011.appspot.com/get/function`

### URL Parameters

Parameter | Description
--------- | -----------
keys | An array of keys of labeling functions to get 
