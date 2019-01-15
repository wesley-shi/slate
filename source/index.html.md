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

Uploads a YAML file into nodes to then be queried. See taxonomy YAMLs for samples. Yaml labels should not have periods.

### HTTP Request

`GET https://agile-binder-228011.appspot.com/post/yaml`

### URL Parameters

Parameter | Description
--------- | -----------
yaml | The YAML file to be posted
name | The name of the tree, if an existing tree already exists, the nodes will be overwritten and labeling function will be reassigned

## GET taxonomy list

Gets the list of taxonomys available

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

Retrieves a taxonomy tree based on a node key (ex. `att_agent-actions` or `att_agent-actions.open`)

In the future, this API can support filters and other options.

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

> The below is an example of a function post:

```bash
curl -X POST \
  http://localhost:8000/api/post/functions/ \
  -H 'Authorization: Token cdc57387d401b7f9a97d6dd200142764a6fd5ad249e27e8ae6d8152cd1f61250' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: ce7b16cd-0172-4860-8ebb-d04bb79c374e' \
  -H 'cache-control: no-cache' \
  -d '{
        "functions": [{
            "function": "node = '\''agent-actions.business.employee_type'\'' if '\''who'\'' in text and '\''chatting'\'' in text and text.find('\''chatting'\'') > text.find('\''who'\'') else None",
            "name": "sample",
            "key": "unique_key",
            "description": "Checks for words '\''who'\'' and '\''chatting'\'' and that they are in order",
            "taxonomy": "agent-actions",
            "node_key": "agent-actions.business.employee_type"
        }],
        "run": true
    }'
```

> If the function is run, it should return a data object with the node positive count for exisitng labels:

```json
{
    "agent-actions.open.greeting.ask_name__1": {
        "score_date": 1547477906793,
        "total_samples": 802,
        "nodes": {
            "agent_actions_open.greeting.ask_name": 30,
            "agent_actions_open.greeting": 35,
        }
    }
}
```

Upload a labeling function. The labeling function will be run with `exec`. The text of a particular data message will be available in the variable `text` and the key output `text` should be assigned to the `node` variable like `node = 'agent-actions.open.greeting.ask_name'`.

### HTTP Request

`POST https://agile-binder-228011.appspot.com/post/functions`

### Function Object

Parameter | Description
--------- | -----------
function | The function as a string. Assume access to a variable `text` and adding string keys to the object `subtaxonomy` as `-1|0|1`
taxonomy | The key of the taxonomy, this function is to be grouped under
name | A name to identify the labeling function
description | A description for the labeling function
key | A unique key for this function. If a new function uses this key, it will replace the previous function
nodes | An array of node keys the labeling funtion will be assigned under. All nodes in array should be in the same hierarchy

### URL Parameters

Parameter | Description
--------- | -----------
functions | An array of function objects to be automatically run and labeled
run | A boolean flag to indicate whether snorkel should re-run data for taxonomy with these new functions

## POST labels

Upload data for a taxonomy. Should the text be labeled, add the property `label` to each label object.

### HTTP Request

`POST https://agile-binder-228011.appspot.com/post/labels`

### Label Object JSON

Parameter | Description
--------- | -----------
text | The message text
taxonomy | The taxonomy for this label
label | The correct subtaxonomy label for this text (optional)

### URL Parameters

Parameter | Description
--------- | -----------
labels | An array of label objects to posted