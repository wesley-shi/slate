---
title: API Reference

includes:
  - errors

search: true
---

# Introduction

Welcome to the Cresta Taxonomy API!

The goal of these APIs are to support write labeling functions applied to text input and returning a sub-taxonomy output.

# Authentication

You will need to retrieve an API token and add the token as a header in the format

> Use API key in an authorization header

```shell

# You can make a post request to the login endpoint to get an API token
curl -X POST \
  https://agile-binder-228011.appspot.com/api/auth/login/ \
  -H 'Content-Type: application/json' \
  -H 'cache-control: no-cache' \
  -d '{
    "username": "sample_username",
    "password": "sample_password"
  }'

# The JSON response of the above curl request
{
    "user": {
        "id": 3,
        "username": "sample_username"
    },
    "token": "e5323551ff1d2da6b666e8745b8c88e14939012981b8b035f5961f7c2213f320"
}

# With shell, you can just pass the correct header with each request
curl "api_endpoint_here"
  -H "Authorization: API_KEY"
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
    "create_date": <Date created in ms (UTC)>,
    "update_date": <Date created in ms (UTC)>,
    "label": "",
    "key": "att_agent-actions", // An id/key uniquely identifying each node
    "description": "AT&T Agent Actions",
    "type": 0, // 0 = root node
    "parent_key": "",
    "children": [{
        "create_date": <Date created in ms (UTC)>,
        "update_date": <Date created in ms (UTC)>,
        "label": "open",
        "key": "att_agent-actions.open",
        "description": "agent enters chat.",
        "example": ["Agent yc4055 enters chat as Lauren"],
        "type": 2, // 2 = leaf node
        "parent_key": "att_agent-actions",
        "functions": [{
            "function": "function as a string",
            "name": "function_name",
            "key": "a key assigned by the user",
            "description": "a description by the user",
            "owner": "the username of the owner of the function"
        }],
        "owner": "username of owner"
    }, {
        "create_date": <Date created in ms (UTC)>,
        "update_date": <Date created in ms (UTC)>,
        "label": "greeting",
        "key": "att_agent-actions.greeting",
        "description": "agent greets visitor.",
        "example": [],
        "type": 1, // 1 = parent node
        "parent_key": "att_agent-actions",
        "children": [
            ...
        ],
    }],
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
node_type | 0=root_node, 1=parent_node, 2=leaf_node
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
node | The lowest level node in a hierarchy this labeling function will resolve

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