# python-redux

A port of JavaScript's [redux](https://github.com/reactjs/redux) to Python, with some minor modifications, to make state management easier. The project's main goal is to provide:

- A predictable state container for a Python application, including and especially server-side
- Clean Pythonic interface
- First-class support for asynchronous action through asyncio

## I. Basics

At its bare minimum, a redux application has 4 components:

- **State**: the object that holds the application's state. There is only one state object in a redux application
- **Action**: data payload sent from the application to the store
- **Reducer**: pure function that takes the previous state and an action and returns the next state. Python-redux also offers an object-based reducer wrapped around aforementioned pure function just for syntastic sugar purposes.
- **Store**: think of this as manager object to bring action, state and reducer together

Some tips:
- There is always only one state object in the application
- I like to think of the idea behind redux as an one-liner as follow: 

```python 
new_state = reduce(lambda f: f(state), reducers, initial_state)
```

### 1. Actions

The base Action class is just a subclass of `namedtuple`, so you can create an action using the `namedtuple` syntax

```python
from redux.core import Action

AddSitups = Action('Add situps', ['num'])
AddPushups = Action('Add pushups', ['num'])
```

**Convention**: Action name should begin with an English verb

### 2. States

A state object could use any data structure to store its information. In the following example, the global state object of the application contains 2 lists.

```python
ReduxState = namedtuple('My Redux Gym Tracker', ['situps', 'pushups'])
ReduxState.__new__.__defaults__ = [], []
```

### 3. Reducers

Reducer takes the current state object, apply an action on it and return the new state.

```python
def gym_app(state: ReduxState, action: Action) -> ReduxState:
    return ReduxState(
        situps = situps(state.situps, action),
        pushups = pushups(state.pushups, action)
    )


def situps(state: list, action: Action) -> list:
    if action == AddSitups:
        state = state + [Pushup() for _ in range(action.num)]

def pushups(state: list, action: Action) -> list:
    if action == AddPushups:
        state = state + [Situp() for _ in range(action.num)]    
```
 
There is also an utility to combine reducers together

```python
from redux.utils import combine_reducers
gym_app = combine_reducers(situps, pushups)
```

Or you can use a BDD-style root reducer object to combine reducers together

```python
from redux.core import Reducer
gym_app = Reducer(initial_state = ReduxState())
gym_app
.when(AddSitups).then(situps)
.when(AddPushups).then(pushups)
```

Since Python doesn't have `switch` statement out of the box, this approach gives you the ability to splitting reducers into even smaller functions without sacrificing readability

```
gym_app
.when(AddSitups).then(process_new_situps)
.when(AddPushups).then(process_new_pushups)
```

### 4. Store

Store is the object that bring actions and reducers together.

- Create a store out of the `root_reducer`, i.e. the gymApp from the example above

```python
from redux.utils import create_store
store = create_store(root_reducer, initial_state=None)
```

- To access the current state tree

```python
store.getState()
```

- To dispatch an action object

```python
store.dispatch(action)
```

- Slightly different from the JavaScript implementation, you can dispatch an action creator `callable` out of the box without the need to enable extra middleware

```python
store.dispatch(lambda: action)
```

- After an action is dispatched, you can optionally invoke callback by registering it as a listener. This global listener will be triggered every time an action is dispatched. The state may or may not have been modified at that time.

```python
store.subscribe(listener)
```

- Python-redux also allows the possiblity to register a local listener for every action. 

```python
store.subscribe(log_pushups, to=AddPushups)
```

- There is also a low-level api to access and replace reducer

```python
store.get_reducer()
store.replace_reducer(next_reducer)
```

## II. Advanced concepts

### 1. Middleware

Middlewares sit between when an action is dispatched and when it reaches the reducers. To enable the use of middlewares:

```python
from redux.middlewares import logger, crash_reporter
create_store(root_reducer, middlewares=[logger, crash_reporter])
```

### 2. First class support for async actions with asyncio

TBU

### 3. Custom action scheduler

TBU 

## Alternatives

State management is hard, especially when asynchrony and parallelism are in the equation. Some other approaches to state management include:

- [State Machine](https://github.com/tyarkoni/transitions)
