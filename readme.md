# easy-pipeline
> Framework for building pipelines in node.

## Install
```sh
npm install easy-pipeline
```

## Features
- Create pipeline stages as functions.
- Safer execution environment which prevents the output of function being modified.
- Extensible logging framework to capture the input and output of each function.

## Stages
Stages are the fundamental building blocks of a pipeline. 
It's defined by a function that receives a single argument - context and 
returns an arbitary object representing the output.

```javascript
// Simple stage that returns an object with a property called foo.
const stageFoo = context = ( { foo: 'a' });

// Another stage that return an object with a property called bar.
const stageBar = context = ( { bar: 'b' });
```

Once we have the stages, we can call ```createPipeline``` function to bind them 
together to create a pipeline. 

```javascript
const createPipeline = require('easy-pipeline');

const pipeline = createPipeline(stageFoo, stageBar);
```

Finally the pipeline can be invoked by providing the initial input (optional).

```javascript
pipeline();
```

Result of the pipline would be the combination of return values from stageFoo 
and stageBar.

```javascript
{ foo: 'a', bar: 'b' }
```

### Accessing the results of a stage from another
Our pipeline would be little useful if we could not access the result of 
one stage from another. ```Context``` argument passed into each stage provides
access various useful services including the results of previous stages.

Let's modified our ```barStage``` to return a value based on the previous stage.

```javascript
const barStage = context = ({ bar: `${context.props.foo}-b`});
```

This time around, the result of our pipeline would be:
```
{ foo: 'a', bar: 'a-b' }
```

### Safety of props
```context.props``` received by a stage is an immutable object. If a stage
accidently attempts to modify the output from another, it will result in an error.

```javascript
const barStage = context => {
  context.props.foo = 'c'; // Receives a type error.
};
```

### Stage name
Each stage has a name. This is used within various logging operations 
(discussed below). By default the stage name is same as the function name. 
We can modify this by attaching a config property to our stage function.

```javascript
const barStage = context => ( { bar: 'b' });
barStage.config = { name: 'my-bar-stage' };
```

This feature can be useful if we want to have more elaborate names for our 
stages or if we are running on node 6.4.0 where function name property is 
not available for certain types of expressions (http://node.green/#ES2015-built-in-extensions-function--name--property). 
