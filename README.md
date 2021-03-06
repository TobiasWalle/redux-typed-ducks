# Redux Typed Ducks

[![Build Status](https://travis-ci.org/mhoyer/redux-typed-ducks.svg?branch=master)](https://travis-ci.org/mhoyer/redux-typed-ducks)
[![npm](https://img.shields.io/npm/v/redux-typed-ducks.svg)](https://www.npmjs.com/package/redux-typed-ducks)
[![Dependencies](https://david-dm.org/mhoyer/redux-typed-ducks.svg)](https://david-dm.org/mhoyer/redux-typed-ducks)
[![Development Dependencies](https://david-dm.org/mhoyer/redux-typed-ducks/dev-status.svg)](https://david-dm.org/mhoyer/redux-typed-ducks#info=devDependencies)

Inspired by https://github.com/erikras/ducks-modular-redux I wanted better
type support for projects written with typescript using the redux pattern. 
This project also tries to shrink the boilerplate code even further, e. g. 
no need for those switch-case statements to map the actions to its reducer 
functions.

Besides that, the goal was to stay independent of a redux implementation. 

## Install

```npm install redux-typed-ducks```

## API

### createDuck(actionType, payloadReducer)

##### Arguments

 - ```actionType``` - a string identifying the action that will be dispatched later
 - ```payloadReducer``` - a function that takes the previous state and an optional payload to create and return the next state.

##### Returns

The returned object (aka duck) is the actual action creator function, but
also has the two parameters assigned as properties (which are only relevant 
for internal use).

#### Example

```javascript
import {createDuck} from 'redux-typed-ducks';

const replaceDuck = createDuck('duck/REPLACE', replaceReducer);
replaceDuck.actionType;     // -> 'duck/REPLACE'
replaceDuck.payloadReducer; // -> replaceReducer()
replaceDuck('ribbit');      // -> { type:'duck/REPLACE', payload:'ribbit' } 
```

Interestingly, we do not have to fiddle with FSAs in our 
reducer implementations and can focus on the payload.

```javascript
function replaceReducer(state: string, payload: string) {
    const nextState = payload;
    return nextState;
};
```


### createReducer(ducks, [initialState])

At some point we need to create the actual reducer.

With ```createReducer``` we can convert the given ducks into a reducer function.

##### Arguments

 - ```ducks``` - an object literal holding the ducks created with ```createDuck```
 - ```initialState``` - (optional) will be ```{}``` by default

##### Returns

The reducer function, that will take care of extracting and mapping a 
dispatched action to the correct payload reducer.

#### Example 

```javascript
import {createDuck, createReducer} from 'redux-typed-ducks';

const replaceDuck = createDuck('duck/REPLACE', replaceReducer);

const initialState = 'initial string state';
const ducks = {
    replaceDuck,
    // ... add more ducks here
};
const reducer = createReducer(ducks, initialState);
```

Later on, [redux](https://github.com/reactjs/redux) or [@ngrx/store](https://github.com/ngrx/store) 
will execute the reducer like so:

```javascript
let prevState = 'previous state';
let dispatchedAction = { type: 'duck/REPLACE', payload: 'ribbit' };

const nextState = reducer(prevState, dispatchedAction); // -> 'ribbit'
```

##### Nested ducks

```createReducer()``` also supports object literals where ducks can be nested to 
help with structuring your ducks:

```javascript
import {createDuck, createReducer} from 'redux-typed-ducks';

const replaceDuck = createDuck('duck/REPLACE', replaceReducer);
const revertDuck = createDuck('duck/REVERT', revertReducer);

const ducks = {
    nested: {
        replace: replaceDuck,
        revert: revertDuck
    }
};
const reducer = createReducer(ducks, 'initial state');
```


### createDispatchedActions(ducks, store)

Additionally we can already wire up the ducks with the store 
to create self-dispating action functions.

##### Arguments

 - ```ducks``` - an object literal holding the ducks created with ```createDuck```
 - ```store``` - any object that has a ```dispatch()``` function

##### Returns

An object literal with the same keys as the given ```ducks``` object. Each
entry points to a self-dispatching action function. Hence, we can call it
directly from our components.

Exporting the actions object will help to prune boiler plate code 
in your actual application components.

#### Example

```javascript
import {createDuck, createDispatchedActions} from 'redux-typed-ducks';

const replaceDuck = createDuck('duck/REPLACE', replaceReducer);
const ducks = {
    replace: replaceDuck,
};
const fakeStore = {dispatch: (action) => { console.log(action); }};
const actions = createDispatchedActions(ducks, fakeStore);
actions.replace('next'); // -> dispatches the 'duck/REPLACE' action 
                         //    with 'next' as payload. It also returns
                         //    the generated action object.
```

##### Nested ducks

```createDispatchedActions()``` also supports object literals where ducks can be nested to 
help with structuring your actions:

```javascript
import {createDuck, createDispatchedActions} from 'redux-typed-ducks';

const replaceDuck = createDuck('duck/REPLACE', replaceReducer);
const revertDuck = createDuck('duck/REVERT', revertReducer);

const ducks = {
    nested: {
        replace: replaceDuck,
        revert: revertDuck
    }
};
const fakeStore = {dispatch: (action) => { console.log(action); }};
const actions = createDispatchedActions(ducks, fakeStore);
actions.nested.replace('next');
actions.nested.revert();
```

### Samples

- [Redux Typed Ducks - Counter sample](https://github.com/mhoyer/rtd-counter): A simple React sample that uses redux typed ducks.
- [ng2-kanban](https://github.com/mhoyer/ng2-kanban): A slightly more complex *Redux Typed Ducks* sample using [@angular](https://github.com/angular/angular) and [@ngrx/store](https://github.com/ngrx/store)

