# A Redux redux 
#### Reducers, Actions, Action Creators, Stores, Providers, Thunks, Sagas...

You're still pretty new to Javascript. But now the pressure is on to use a framework, so you've picked up React. You're starting to get the basic gist of it, there's still a lot of magic, especially when it comes to Webpack. You've been hearing about Redux for almost as long as you've been hearing about React. Everything you read tells you you'll know when you need it, don't rush it. But you know the truth is that every job description you see mentions React and Redux in the same breath. So you figure there's no time to waste. Even if your personal project that you're making to learn React is pretty simple, you still want to use the same tools the pros use. 

And then you hit the inevitable wall of jargon, split files, and boilerplate. 

Does it really take 5 files to deal with one variable?? 

The simple answer is no. It doesn't. Redux is an amazing tool in large part because it is so simple. That simplicity also allows a great deal of flexibility in how it's structured and used. That flexibility means a lot of time has gone into finding the best way among many of doing things (best practices). And those best practices means sometimes it can be very hard as a beginner to see the forest for the trees. Our goal today is to build a very simple version of redux in the most ELI5 way we can, one step at a time. 

Along the way we'll explore a lot of important Javascript concepts that might similarly feel a bit confusing as a beginner and, similarly, aren't really all that difficult at all. Large sections of this guide will look at these general concepts and talk with the expectation that even if you've learnt them, they may still feel a little unclear. 

By the end, we will hopefully have a clearer grasp of some of the most useful Javascript rulesand patterns and, at the same time, a better understanding of Redux (and why implementing it in small scale projects isn't entirely a waste of time and effort).

I should mention that we aren't really going to talk about React in this course. Redux is used with it most commonly, but Redux is really just a vanilla Javascript library and can be used anywhere. 'react-redux' is a library that is built to connect React components with the Redux library. after we have a strong command of Redux it will be a lot easier to see what is happening in react-redux, but we won't build that library from scratch as well, just cover the essentials to feel like we know what is going on underneath the hood. 

## Part 1: Creating a Redux store

### Chapter 0: We begin at the beginning

We're going to start by working with the simplest of functions and only a few pieces of state. I'm of the strong opinion that for novices learning best practices first is counterproductive. Hopefully the concepts you learn today will help you understand why the official Redux sourcecode was written the way it was as we slowly build towards it iteratively. 

We'll start simple and improve as we go. A few variables (our program's state) and a few functions that read and change them (our getters and setters respectively):

```javascript
let mood = 'Happy'
let children = ['Bobby', 'Billy', 'Betty', 'Bobo']

function showMood() {
  console.log(mood)
}

function changeMood(newMood) {
  mood = newMood
}

function showChildren() {
  console.log(children)
}

function addChild(newChild) {
  children.push(newChild)
}

```

And that's it. We have the fundamentals of a program. We can now do stuff:

```javascript
showMood() // 'Happy'
changeMood('Sad')
showMood() // 'Sad'
showChildren() // 4
addChild('Bozo')
showChildren() // 5
```

### Chapter 1: Gathering up state into one place

Not everything we are going to do as we build our simplified version of Redux is going to seem essential. But hopefully it always seems clear. For the next step, we are going to collect all of our state into an object. Like good housekeeping, we're going to hide all of those global variables so they don't create a mess (global just means anything can access them - it doesn't necessarily seem like such a bad idea, it's certainly more convenient. But what happens when more than one thing has the same name? Either your program starts doing weird stuff or telling bad puns. Or both):

```javascript
let ourState = {
  mood: 'Happy',
  children: ['Bobby', 'Billy', 'Betty', 'Bobo']
}
```

All of the functions we made earlier still work in principle. We just need to now point them towards the new state object we made earlier. For example:

```javascript
function showMood() {
  console.log(ourState.mood)
}

function addChild(newChild) {
  state.children.push(newChild)
}
```

### Chapter 2: Let's start making an interface

Now that we have our state (variables) in one place, we can get even fancier. Instead of directly changing (setting) our state, let's just tell our state to change itself. Why? Well, if for no other reason than to not worry about how it happens. Our functions right now are simple, but if we made it a bit more complicated, if we wanted to add in extra checks to make sure we couldn't do things like this:

```javascript
addChild(0.7734)
```

When we make the state object responsible for these things it keeps our other functions simple. THe idea being that if more than one function needs to use the same state, why repeat all that extra work? Let the state handle itself.

For example, let's say we want to make sure a newMood is a string. We would do something like this:

```javascript
function changeMood(newMood) {
  if(typeof newMood === 'string') {
    ourState.mood = newMood
  }
}
```
If we had to repeat that 'if' check every time we wanted to change the state it would get very tedious. Moving the check into the object itself where the state will be touched means we don't have to repeat ourselves.

NB: The key part of this chapter is remembering that objects in javascript can have all kinds of things in them. Most importantly, they can have functions in addition to variables. So that's how we'll make our built in ways to interact with it (interface/API), we'll add functions into our state object.

```javascript
let ourState = {
  mood: 'Happy',
  children: ['Bobby', 'Billy', 'Betty', 'Bobo'],
  changeMood: function(newMood) { 
    if(typeof newMood === 'string') {
      ourState.mood = newMood
    }
  },
  addChild: function(newChild) { this.children.push(newChild)} 
}
```

The 'this' keyword in Javascript is considered a tricky thing to understand. But for our purposes here, it should suffice to know that it is pointing at the 'ourState' object (which means using 'this.mood' is like saying 'ourState.mood' but from inside 'ourState').

So now, to change our state, we have to do things a bit differently:

Instead of directly changing the state:

```javascript
if(typeof newMood === 'string') {
  ourState.mood = newMood
}
```

We tell the state to change itself (but we don't worry about how):

```javascript
ourState.changeMood('Sad')
```

### Chapter 3: The plot thickens

If it feels like we're covering a lot of basic ground tediously, it's because we are. The idea is that by the end of the process it will seem downright simplistic. For the most part, the how of Redux is simplistic, it's the why and the syntax that tends to leave beginners a little unsure.

On a side-note, since we won't build anything that takes advantage of some of the coolest benefits of Redux, such as time-travel debugging, we should at least take a brief moment to mention about them. Time-travel debugging for instance sounds really crazy but it's really only time travel in the way that rewind and fast-forward on a VHS player are time travel. Sadly, you can put away your sports almanac now.

One of the biggest advantages of Redux is being able to see a perfect record of how changes are made to your state. Redux never actually changes anything directly, it gets instructions about what to change, and then creates new versions of our state based on those instructions. This gives us the potential to save a history of those instructions and versions of the state just like a web browser might when you are wasting too much time on Reddit. So we could browse through all the state changes in the order they occured when something goes wrong (time-travel debugging). 

Constantly creating new copies of the state and pretending they are the same thing also helps get us closer to avoiding another problem, race conditions. This is race in the sense of the tortoise and the hare, not in the sense of identity. The basic gist is that when you have two or more things going on in your program that will eventually have an effect on the same thing (such as a piece of state/variable) it can create unpredictability. In other words, when the order of things happening matters but you can't guarantee what order they will happen in you have race conditions. Sometimes it's not important and sometimes one instruction is 'jump out a plane' and the other is 'put on a parachute' and the order really, really matters!

When you have two actions in redux they never actually change the same thing, one action will create a new, altered copy of the state and the second action will create a copy of that state. It doesn't solve the problems with the order of actions, but it means two things aren't trying to change the same thing at the exact same time. And it means we can actually go back and see what the order is if we weren't sure (time travel debugging again). It's not as exciting as a real Delorean but it definitely can be helpful.

At this point in our program we have a state object that is a random assortment of variables and functions. It might make sense to bundle all that state together into it's own object just like we did before. 

```javascript
let ourState = {
  state: {
    mood: 'Happy',
    children: ['Bobby', 'Billy', 'Betty', 'Bobo']
  },
  changeMood: function(newMood) { 
    if(typeof newMood === 'string') {
      this.mood = newMood
    }
  },
  addChild: function(newChild) { 
    this.children.push(newChild)
  } 
}
```

Of course now it's a bit odd to call the outer object that holds everything 'state' and the inner object that just holds the state the same thing. So let's rename 'ourState' to 'ourStore' to reflect the fact that it holds stuff but also lets us add/remove/change stuff. Just like a real store. And we need to change our inner functions to reflect the changes.

```javascript
let ourStore = {
  state: {
    mood: 'Happy',
    children: ['Bobby', 'Billy', 'Betty', 'Bobo']
  },
  changeMood: function(newMood) { 
    if(typeof newMood === 'string') {
      this.state.mood = newMood
    }
  },
  showMood: function() {
    console.log(this.state.mood)
  },
  showChildren: function() {
    console.log(this.state.children)
  },
  addChild: function(newChild) { 
    this.state.children.push(newChild)
  } 
}
```

### Chapter 4: I see you but I can't reach you

One of the thing we are doing right now is console logging our state when we want to see it. That doesn't really make sense since we can't actually use it in our program. Instead, we should be returning it whenever its requested so that whoever requested it can use it as they see fit. In fact, instead of having to make separate functions to return individual variables, let's just return the whole state object and let whoever called it have access to whatever information they want. And since we are effectively getting the state for whoever requested it and returning it to them (a getter) we can call this getState():


```javascript
let ourStore = {
  state: {
    name: 'Brendan',
    mood: 'Hungry',
    children: []
  },
  getState: function() { return this.state }
}
```

Now, anytime we want to see what's happening we just ask for the current state and can read from or play with it as we see fit.

### Chapter 5: Setters aren't dogs, but they can breed like rabbits

You've probably heard a lot of chatter about something called DRY code. Man, do programmers like their acronyms and, at least in the case of WYSIWYG, have a pretty limited sense of irony. It just means try not to do the same thing over and over again in different places. If you find you are repeating yourself, take that thing you are repeating, separate it and then just reference it over and over again instead of rewriting it each time you need it.

LETS MOVE THIS TO LATER.BECAUSE IT STARTS LEADING INTO SEPARATE ACTIONS. START WITH READING.

In our case, it's not really a big deal, but imagine we had a lot of similar variables and a lot of similar functions to do stuff to them.

```javascript
let ourStore = {
  state: {
    name: 'Brendan',
    fruit: 'Passion fruit',
    color: 'Purple',
    age: 35,
    hair: false
  },
  getState: function() { return this.state },
  changeName: function(newName) { this.state.name = newName},
  changeFruit: function(newFruit) { this.state.fruit = newFruit},
  changeColor: function(newColor) { this.state.color = newColor}
  changeAge: function(newAge) { this.state.age = newAge}
}
```

Since all of these functions do the same thing, it's pretty easy to see how we can make this a bit more efficient. Instead of a separate function for changing every item, we pass in a second argument which tells us the item type and the newItem (key/value pair) and we can dynamically change our state. 

```javascript
let ourStore = {
  state: {
    name: 'Brendan',
    fruit: 'Passion fruit',
    color: 'Purple',
    age: 35,
    hair: false
  },
  getState: function() { return this.state },
  changeItem: function(itemType, newItem) {
    if(this.state.hasOwnProperty(itemType)) {
      this.state[itemType] = newItem
    }
  }
}
```
(NB: object.hasOwnProperty(keyName) let's us find out if a certain key/name already exists in an object. We'll use this in the following exercise to find out if something already exists before trying to add it or change it or delete it.)

Continuing we now have one function to change our state (setter), but there are other ways to change state, and that means we'll eventually need a lot of functions again. Deleting, adding, apppending... 


```javascript
let ourStore = {
  state: {
    name: 'Brendan',
    fruit: 'Passion fruit',
    color: 'Purple',
    age: 35,
    hair: false
  },
  getState: function() { return this.state },
  addItem: function(itemType, newItem) { 
    // We want to make sure something doesn't already exist before adding it
    if(!this.state.hasOwnProperty(itemType)) {
      this.state[itemType] = newItem
    }  },
  changeItem: function(itemType, newItem) {
    if(this.state.hasOwnProperty(itemType)) {
      this.state[itemType] = newItem
    }
  },
  deleteItem: function(itemType) {
    // We don't want to try and delete something that doesn't exist.
    // Note: We don't need to pass the item value. The delete command only needs to know the key name.
    if(this.state.hasOwnProperty(itemType)) {
      delete this.state[itemType]
    }
  }
}
```


Now, we should still strive to keep all these setter functions in one place but the information (item type, newItem) we're passing it just isn't enough, we need to give it options. So how about we pass it an option? We tell it what item we want to manipulate, what new value we want to use (when we need a newvalue, remember that delete doesn't need a value at all), and finally an action to perform.

The question is how do handle these three features? For example, we could pass in the key, value, and then an external function that does whatever we want. This would give us a lot of flexibility, we could pretty much do whatever we want to the state without having to hard code the functions ahead of time. But this approach seems pretty dangerous. We could pass ANY function. The whole idea of managing our state is having control. So it makes more sense to have the state control itself. That means keeping all the key editing functions internal and just letting our external request indicate which function they want to use, rather than passing in the function itself.

And what would be the best way to indicate the function we want it to perform? How about the name of the function itself? Now, we can't pass it the name as a variable (functionName) because that is just like passing the actual function, so we need to pass it as a string ('functionName') and let our state object look it up from there. 

Now, how should we store all these functions? We could use another object like we did with state and then look them up by key name. Or we could use a lot of if...else if...else statements. So let's try that.

```javascript
let ourStore = {
  state: {
    name: 'Brendan',
    fruit: 'Passion fruit',
    color: 'Purple',
    age: 35,
    hair: false
  },
  getState: function() { return this.state },
  changeState: function(actionType, itemType, itemValue) {
    if(actionType === 'add') {
      if(!this.state.hasOwnProperty(itemType)) {
        this.state[itemType] = newItem
      }   
    }
    else if (actionType === 'change') {
      if(this.state.hasOwnProperty(itemType)) {
        this.state[itemType] = newItem
      }
    }
    else if (actionType === 'delete') {
      if(this.state.hasOwnProperty(itemType)) {
        delete this.state[itemType]
      }
    }
    else {
      // We don't have any kind of default action yet for when an action type isn't specified
    }
  }
}
```

You might notice that I added actionType as the first argument instead of the third. That's because we don't always need an itemValue (for example when we delete something), and obviously our new changeState function needs an actionType to work correctly. Some actions might not even need a second argument (itemType) if, for instance, all they do is delete everything, or change everything etc. Each action might end up needing different types of keys and values. If we hardcode the arguments our changeState function needs, it might get very tricky. So let's make it more flexible. Instead of passing in separate arguments, we can wrap all of the options into an object. That means our changeState doesn't really need to care about the order of arguments or which ones we give it -- only the if{} block needs to care. Well, with one big exception. The whole thing breaks if we forget to specify the action type. So we should probably do a check to make sure our new action object has a type key first. We should also check if the action argument is an object, and if it exists in the first place.

We'll do one more thing to be dry. Instead of always looking up the action.type, we'll save it in a string (this makes things a bit clearer and saves a bit of typing when there are a lot of actions -- not so much in our example case).

```javascript
let ourStore = {
  state: {
    name: 'Brendan',
    fruit: 'Passion fruit',
    color: 'Purple',
    age: 35,
    hair: false
  },
  getState: function() { return this.state },
  changeState: function(action) {
    // Remember, return here will stop the rest of the function from running.

    if(!action) {
      return
    }
    if(typeof action !== 'object') {
      return
    }
    if(!action.hasOwnProperty('type')) {
      return
    }

    const type = action.type

    if(type === 'add') {
      if(!this.state.hasOwnProperty(action.itemType)) {
        this.state[action.itemType] = action.newItem
      }   
    }
    else if (type === 'change') {
      if(this.state.hasOwnProperty(action.itemType)) {
        this.state[action.itemType] = action.newItem
      }
    }
    else if (type === 'delete') {
      if(this.state.hasOwnProperty(action.itemType)) {
        delete this.state[action.itemType]
      }
    }
    else {
      // We don't have any kind of default action yet for when the action type isn't recognized
    }
  }
}
```

### Chapter 6: A new complication makes things easier

We talked earlier about race conditions. When we can't predict the order of actions. That means we also can't predict the state that the actions act on. So there are a few things that we should consider doing to make changeState() more predictable. 

Let's focus on all those action checks first. Right now we are receiving action objects directly into the same function that is changing the state. What if we created a separate function that was responsible for processing actions? Then we could handle all the validity checks and other stuff (like saving a log of actions for time travel debugging) in one place and just focus on changing state in another place.

```javascript
let ourStore = {
  state: {
    name: 'Brendan',
    fruit: 'Passion fruit',
    color: 'Purple',
    age: 35,
    hair: false
  },
  getState: function() { return this.state },
  receiveActionAndSendActionToChangeState: function(action) {
    if(!action) {
      return
    }
    if(typeof action !== 'object') {
      return
    }
    if(!action.hasOwnProperty('type')) {
      return
    }

    this.changeState(action)
  },
  changeState: function(action) {

    const type = action.type

    if(type === 'add') {
      if(!this.state.hasOwnProperty(action.itemType)) {
        this.state[action.itemType] = action.newItem
      }   
    }
    else if (type === 'change') {
      if(this.state.hasOwnProperty(action.itemType)) {
        this.state[action.itemType] = action.newItem
      }
    }
    else if (type === 'delete') {
      if(this.state.hasOwnProperty(action.itemType)) {
        delete this.state[action.itemType]
      }
    }
    else {
      // TODO
    }
  }
}
```

This does seem to complicate things unnecessarily. But setting up a gatekeeper like this, gives us a great place to do things related to actions that isn't just specifically changing the state. Later on we'll add in a log of actions. But for now we will focus on another problem. What happens if the changeState() is fired more than once from the same receiveActionAndSendActionToChangeState? Remember, we don't want to have that kind of unpredictability. What would cause that to happen? Well, one issue could be if someone tried to send another action from inside of the first one. So we should definitely make sure only one changeState() can fire at a time from one receiveActionAndSendActionToChangeState call. How do we do that though? If you're imagining it comes down to some fancy, advanced programming techniques, you might be surprised. Often the most important programming acronym of all is KISS, keep it simple, silly! We're just going to use a simple boolean, to tell us if the function is already running. Something like this:

```javascript
receiveActionAndSendActionToChangeState: function(action) {
  if(this.isChangingState) {
    // If there is already a changeState running we want to quit without trying to
    // change state again. 
    return
  }

  this.isChangingState = true
  this.changeState(action)
  this.isChangingState = false
}
```

Now you may notice two things about the above code. First of all, as long as changeState isn't resolved we won't be able to fire another one. That means we don't have to worry about someone nesting changeState calls. The second thing you might notice is that we referenced a variable isChangingState that we never declared. That doesn't work of course, but where should we declare it? Well, it can't be inside this function because then if the function gets called twice we also get two instances of isChangingState which negates the purpose of only having one. So we'll have to store it outside the function, so even if the function is called multiple times at once, each instance of the function points at the same isChangingState.

```javascript
{
  isChangingState: false,
  receiveActionAndSendActionToChangeState: function(action) {
    if(this.isChangingState) {
      // If there is already a changeState running we want to quit without trying to
      // change state again. 
      throw Error()
      return
    }

    this.isChangingState = true
    this.changeState(action)
    this.isChangingState = false
  }
}
```

There's another benefit to storing isChangingState externally, we can use it with other functions to see what is happening inside receiveActionAndSendActionToChangeState at any one time and make decisions based on that. Right now we don't want to have attempts to change state happen while the state is actively changing. It creates unpredictability that we are working hard to avoid. To that same end we should also prohibit users from trying to get state while the state is actively changing as well. So let's use the same isChangingState to make sure we can only getState when the coast is clear:

```javascript
getState: function() {
  if(this.isChangingState) {
    console.log('You can't call getState while the state is being changed')
    return
  }
  return this.state
}
```

### Chapter 7: Closures. Oh dear god, closures.

If you don't know what a closure is, then you've probably heard about it in hushed tones and fearful whispers. But, like many things in programming, the jargon makes it sound a lot more complicated than it is. Basically, it just means that if a function has access to some outside data when it is created in one place, it will continue to have access to that data, even if you call the function from other places or pass it around. Let's make a very simple example. We're going to use a container function that will be our pretend version of a program, module, library, etc. This container function will hold a single variable and another function inside. Our container will have one purpose, to return the function it contains. 

```javascript
function containerFunction() {
  let insideVariable = 'hello'

  function insideFunction() {
    console.log(insideVariable)
  }

  return insideFunction
}
```

So now if we call the container function it will return the inside function that we save to use later. The cool thing about this is, even though we don't explicitly return the inside variable at the same time, our inside function always has access to it, even when our inside function is called from another program or module. That a closure. That's it. As long as the inside function exists, javascript will keep a copy of the inside variable it has access to even if we can't directly access the inside variable, which, in this case, we can't.

```javascript
let savedVersionOfInsideFunction = containerFunction()
savedVersionOfInsideFunction() // 'hello'
```

So, why are we even talking about closures? Well, right now, the store that we have been using isn't really doing a good job of protecting it's inner state object. Even though we've been talking about getters and setters to read and change the inner state, the truth is, since our store is just a simple object, we can still access it directly. All of these would work:

```javascript
ourStore.state[itemType] = newItem
ourStore.state = {}
delete ourStore.state[itemType]
```

Now, that doesn't seem like very good housekeeping. So how can we protect our state from prying eyes and fingers? How can we still have access to the variables, but only using the functions we created (getState, changeState)? If you think the answer isn't a closure, than I probably need to work on my chapter titles a bit more! So, how can we do that? Let's refactor our store object into a store function (just like the container function in our example above). We don't want to return direct access to the state, but we do want to return direct access to the functions which outside programs will need access to in order to perform authorized (by us) actions on the state. Unlike the closure demo above, we will need to pass access to more than one function. So, instead of return an inside function from the container function, we'll return an object that contains multiple inside functions. Like so:

```javascript
function ourStore() {
  let state = {
    name: 'Brendan',
    fruit: 'Passion fruit',
    color: 'Purple',
    age: 35,
    hair: false    
  }
  let isChangingState = false

  function getState() {
    if(isChangingState) {
      console.log('You can't call getState while the state is being changed')
      return
    } 
    return state 
  }

  function receiveActionAndSendActionToChangeState(action) {
    if(!action) {
      return
    }

    if(typeof action !== 'object') {
      return
    }

    if(!action.hasOwnProperty('type')) {
      return
    }

    if(isChangingState) {
      return
    }

    isChangingState = true
    changeState(action)
    isChangingState = false
  }

  function changeState(action) {

    const type = action.type

    if(type === 'add') {
      if(!this.state.hasOwnProperty(action.itemType)) {
        this.state[action.itemType] = action.newItem
      }   
    }
    else if (type === 'change') {
      if(this.state.hasOwnProperty(action.itemType)) {
        this.state[action.itemType] = action.newItem
      }
    }
    else if (type === 'delete') {
      if(this.state.hasOwnProperty(action.itemType)) {
        delete this.state[action.itemType]
      }
    }
    else {
      // TODO
    }
  }

  return {
    getState,
    receiveActionAndSendActionToChangeState
  }
}
```

Now, if you try and run this, it won't work. Since our new store is a function and not an object, we don't need to refer to other parts of it with 'this', we can treat it like a mini program and just refer to other functions and variables in it directly (which will be saved along with whatever functions refer to them because of...closures!). 

While we're refactoring, maybe we should take a second to start thinking about the names of things again. 'receiveActionAndSendActionToChangeState' doesn't exactly roll off the tongue. So what is this functions main job? Well, it's job is to handle our messages (actions) and send them to our store like Fedex. It's a public method that lets us send messages to a private method (changeState), so in some senses it seems better to think of it as a sender rather than a receiver. So we could name it sendMessage or dispatchPackage or faxDocument or mailPostcard or... well, if you've ever tried using redux before, you know where I'm going with this. The benevolent gods who created Redux decided to call this function 'dispatch', so we're going to do the same thing. And while we're at it, since 'isChangingState' is really a measure of whether or not this 'dispatch' function is running, maybe we should change that to 'isDispatching'.

We should also rethink the name of our container function 'ourStore'. Because it's not a simple object anymore and instead a function that creates a new object (with our public methods), in theory we could create multiple stores now. We won't, but we could. So let's change this to 'createStore'. Here's what we've got so far:

```javascript
function createStore() {
  let state = {
    name: 'Brendan',
    fruit: 'Passion fruit',
    color: 'Purple',
    age: 35,
    hair: false    
  }
  let isDispatching = false

  function getState() {
    if(isDispatching) {
      console.log('You can't call getState while the state is being changed')
      return
    } 
    return state 
  }

  function dispatch(action) {
    if(!action) {
      return
    }

    if(typeof action !== 'object') {
      return
    }

    if(!action.hasOwnProperty('type')) {
      return
    }

    if(isDispatching) {
      return
    }

    isDispatching = true
    changeState(action)
    isDispatching = false
  }

  function changeState(action) {

    const type = action.type

    if(type === 'add') {
      if(!state.hasOwnProperty(action.itemType)) {
        state[action.itemType] = action.newItem
      }   
    }
    else if (type === 'change') {
      if(state.hasOwnProperty(action.itemType)) {
        state[action.itemType] = action.newItem
      }
    }
    else if (type === 'delete') {
      if(state.hasOwnProperty(action.itemType)) {
        delete state[action.itemType]
      }
    }
    else {
      // TODO
    }
  }

  return {
    getState,
    dispatch
  }
}
```

### Chapter 8: Teenage Mutant Ninja Functions!

Way back in chapter 3 we spent a few minutes talking about some of the features of Redux, like copying state rather than changing it directly and time-travel debugging, and then we completely ignored them until now. 

So let's talk about mutability. First of all, that doesn't mean being able to suddenly turn off the sound. Mutability is about whether something is able to change or not. Now this may seem confusing because it's rather obvious that things have to change inside of a program in order for anything to work. The real question is how to change things. Do we tinker around with the thing itself or do we create a copy of the thing, take it to the side, change it, and then replace the original thing with the changed copy? The first way is mutable, the second is what we call immutable. Instead of changing things we keep creating new copies of the things and replacing the old ones. That's pretty much it.

So why go through all the trouble instead of just changing something directly? Well, part of it is to be cool obviously. But it also helps us with another problem we touched on earlier, unpredictability and race conditions. Think about it like this. We have one piece of state and two functions. Each function does something to the state and both start running at the same time. But maybe one function changes the state in a way that breaks the other function. That's no good. Instead, we can create a separate copy of the state and pass it to each function. They can do whatever they want to the state without breaking the other function. At the end we get the changed copies back and replace the original with those. Now, you may notice it doesn't necessarily solve the problem of the order of replacing the original state at the end (this is where you start running into 'async', i.e. things that take an unpredictable amount of time to complete), but it does solve the problem of a function breaking halfway through running because some unknown outside function messed with the state it was working on.

So, how can we take this idea and use it in our own store? Well, first of all, we only have to worry about the function that actually changes the state. Our dispatch is only a messenger (and we've already made sure it can't have nested calls (dispatches inside of dispatches)). However, we haven't done anything to handle multiple dispatch calls from the outside at the same time. That means our changeState function might have the same problem we described in the paragraph above. Let's fix that. Instead of changeState directly changing (mutating) our state, let's give it a copy of the state and then receive a copy back and replace the original state. Now, where should we put all this logic? In our gatekeeping dispatch function of course!

To do this, we have to make a few changes. Dispatch will have to provide the state explicitly to our changeState, changeState will have to return new state instead of directly changing the original state object, and dispatch will have to receive that new state and replace the original state with it. We should also avoid some confusion resuing the word 'state' everywhere. Let's call our current state, before it goes through changeState, currentState instead.

There's one more thing we can do here. We've had an empty 'else' block for a long time. It stood for cases where nothing would change. So now, to have the same effect, we'll just return the original state unchanged (remember, our new changeState has to return something!):


```javascript
function createStore() {
  let currentState = {
    name: 'Brendan',
    fruit: 'Passion fruit',
    color: 'Purple',
    age: 35,
    hair: false    
  }
  let isDispatching = false

  function getState() {
    if(isDispatching) {
      console.log('You can't call getState while the state is being changed')
      return
    } 
    return currentState 
  }

  function dispatch(action) {
    if(!action) {
      return
    }

    if(typeof action !== 'object') {
      return
    }

    if(!action.hasOwnProperty('type')) {
      return
    }

    if(isDispatching) {
      return
    }

    isDispatching = true
    currentState = changeState(currentState, action)
    isDispatching = false
  }

  function changeState(state, action) {

    const type = action.type

    if(type === 'add') {
      if(!state.hasOwnProperty(action.itemType)) {
        state[action.itemType] = action.newItem
        return state
      }   
    }
    else if (type === 'change') {
      if(state.hasOwnProperty(action.itemType)) {
        state[action.itemType] = action.newItem
        return state
      }
    }
    else if (type === 'delete') {
      if(state.hasOwnProperty(action.itemType)) {
        delete state[action.itemType]
        return state
      }
    }
    else {
      return state
    }
  }

  return {
    getState,
    dispatch
  }
}
```

### Chapter 9: If it looks like a duck but barks, it's a dog in a disguise

Now, if you try and use the store we made above, it works. But, and this is a big but, it doesn't actually work the way we want it to.

Remember how we talked about creating a copy of the state that wouldn't be directly connected to the original state? Well, even though we thought we did, we actually were still changing the original state directly. This brings us into another fundamental programming concept, passing by value vs passing by reference. That basically means when you create a copy of something (either directly or by passing something into a function which automatically creates internal copies of whatever it gets), sometimes you get an entirely new copy and sometimes you actually just get a reference to the original thing (if you've ever heard of pointers and how scary they are and that's why you should stay away from the language 'C', well, funny thing, Javascript has pointers too. And they aren't scary at all).

How can you know if you're creating a new copy or just a new reference to the original thing? Well, it's pretty simple in Javascript. You can tell by what type of thing you are copying. Strings, numbers, booleans just get copied. Anything you do to the new copies doesn't affect the old ones. Objects {} and arrays [] are where it behaves differently. 

Unlike the other types of variables, objects and arrays are containers of variables. So when you create an object like this:

```javascript
let obj = {
  string: 'string',
  number: 1,
  boolean: true
}
```

The value of 'obj' can't be a simple value, it's a container that has to be able to access, change, etc. all of the stuff inside of it (like our store object was, it has lots of functions to do cool stuff with the data inside the container). So the value of 'obj' is actually a reference (or address or pointer) to the underlying container. When we create a copy of 'obj' we aren't copying all of the inside information, we're just copying the address. Like in the real world, if we keep following the same address, we'll keep ending up in the same place. So, now we know that copying objects and arrays the same way we copy simpler types of variables doesn't actually give us a new container, but how do we create an actualy distinct copy of the container and all it's contents? Well, thankfully, there are lots of cool ways to do that. Unfortunately, as learners, there are lots of ways to do that and we might not know which one is the right one to use. And they aren't always the same between arrays and objects. 

Here are three ways to create a new array that contains everything the old array it's copying does:

```javascript
let newArray = oldArray.slice()
let newArray = Array.from(oldArray)
let newArray = [...oldArray] //This is the hip new way. The '...' is called a 'spread operator'.
```

Those may seem confusing, but as long as you just replaces the newArray and oldArray variables and leave the rest alone, they should work fine.

Here are a couple ways to do the same thing with objects:

```javascript
let newObject = Object.assign({}, oldObject)
let newObject = {...oldObject}
```

Now, you may notice that the second way looks a lot like the 'spread operator' from the third array copy method, and it is. But there's a problem, it's very new, and that means not every browser knows what it means. It's basically a proposed way of doing something, but it's not a standard yet. To make sure it works everywhere it has to be translated back to standard Javascript. If you're trying to use Redux with React than you're probably already using Babel with Webpack to translate your awesome new Javascript into old boring Javascript that works everywhere (especially if you're using create-react-app). And when you look up other tutorials about Redux, you'll probably see this a lot:

```javascript
return {
  ...state,
  newKey: newValue
}
```

But we don't want to complicate things. And Redux doesn't need to be used with React. So we're going to Keep It Simple and use the first method instead.

```javascript
let newObject = Object.assign({}, oldObject)
```

Object.assign basically creates a new empty object {} and then goes through all the things inside of the old object one at a time, copying them over. Now, there are two very important things to understand about Object.assign(). One is very helpful, and one is super annoying. Let's start with the good stuff.

The good:

Not only can we can create a copy of an object, we can also change the old object at the same time by giving it a third argument which will be another object that contains only the changes we want to make. Whatever is new will be added along with the contents of the old object, whatever is a change will replace that part of the old object.

An example of adding:

```javascript
let oldObject = {name: 'Brendan'}
let newStuff = {age: 35}
let newObject = Object.assign({}, oldObject, newStuff)
console.log(newObject) // {name: 'Brendan', age: 35}
```

An example of changing:

```javascript
let oldObject = {name: 'Brendan', age: 35, hair: true}
let newStuff = {hair: false}
let newObject = Object.assign({}, oldObject, newStuff)
console.log(newObject) // {name: 'Brendan', age: 35, hair: false}
```

Remember to make sure to pass in an object where newStuff is, not just any type of variable.

The bad:

Here's the thing, remember how we said that copying objects simply doesn't actually copy them? Well, just because we are using Object.assign, we're not completely out of the woods. What happens when our object contains objects (or arrays)? Well, all magic that Object.assign performs only applies to the container object that we pass it. When it actually starts copying the stuff inside over it uses the simple method again. That means our outside container ight be unique, but any containers inside of it will point to the same place as the containers inside of other copies of the outside container. Ugh. Let's see what this means:

```javascript
let oldObject = {
  name: 'Brendan', 
  favorites: {
    food: 'Tacos',
    color: 'Purple'
  }
}

let copy1 = Object.assign({}, oldObject)
let copy2 = Object.assign({}, oldObject)
```

Here we have two copies of an object. If we change the name of one, the other won't be affected.

```javascript
copy1.name = 'Mr Brendan'
console.log(copy1.name) // 'Mr Brendan'
console.log(copy2.name) // 'Brendan'
```

And if we replace the favorites object of one. It won't change the other.

```javascript
copy1.favorites =  {superhero: 'Spider-Man'}
console.log(copy1.favorites) // {superhero: 'Spider-Man'}
console.log(copy2.favorites) // {food: 'Tacos', color: 'Purple'}
```

But if we try to just change what's inside the inner object, they both change:

```javascript
copy1.favorites.food =  'Cheesecake'
console.log(copy1.favorites) // {food: 'Cheesecake', color: 'Purple'}
console.log(copy2.favorites) // {food: 'Cheesecake', color: 'Purple'}
```

That's because both inner objects are just addresses pointing at a container holding their values, the same container. 


This isn't ideal. This is what shallow copying is. Everything at the top level is a unique copy (in this case our outer container object 'oldObject'), but everything deeper (nested) is treated the old way. So how do we handle that? Well, we have to use Object.assign inside of Object.assign for as many layers as we need. This is a horrible idea for lots of reasons, most importantly because it gets very confusing very quickly. Ideally, we don't nest anything. Instead we 'flatten' our state object. That means keeping it shallow, one or two levels deep at the most (flattening information like this to make it easier to work with is also called 'normalizing' data -- another big word for a simple idea). There are ways to deep copy objects (making sure every nested object is also a new copy not just a new reference to the same address), but they are either harder to write or require us using an outside library like lodash. And either way, we want to encourage clearer structure, which in our case means flatter.

So, in practice it looks like this:

```javascript
let oldObject = {
  name: 'Brendan', 
  favorites: {
    food: 'Tacos',
    color: 'Purple'
  }
}

let newFavorites = {food: 'Cheesecake'}

let copy1 = Object.assign({}, oldObject)
let copy2 = Object.assign({}, oldObject, Object.assign({}, oldObject.favorites, newFavorites))

console.log(copy1.favorites) // {food: 'Tacos', color: 'Purple'}
console.log(copy2.favorites) // {food: 'Cheesecake', color: 'Purple'}
```

It works, but could you imagine going much deeper? It would get very confusing and very fidgety very fast. So let's promise ourselves, even though this isn't strictly necessary, never to nest objects or arrays in our state more than one level deep!

This chapter definitely took us down a little rabbit hole of the specifics of how Javascript works so let's wrap this up by using our cool new Object.assign tool to make sure we are creating copies of our actual state and not just copies of the reference to it:

```javascript
function createStore() {
  let currentState = {
    name: 'Brendan',
    fruit: 'Passion fruit',
    color: 'Purple',
    age: 35,
    hair: false    
  }
  let isDispatching = false

  function getState() { 
    return currentState 
  }

  function dispatch(action) {
    if(!action) {
      return
    }

    if(typeof action !== 'object') {
      return
    }

    if(!action.hasOwnProperty('type')) {
      return
    }

    if(isDispatching) {
      return
    }

    isDispatching = true
    currentState = changeState(currentState, action)
    isDispatching = false
  }

  function changeState(state, action) {

    const type = action.type

    if(type === 'add') {
      if(!state.hasOwnProperty(action.itemType)) {
        return Object.assign({}, state, {[action.itemType]: action.newItem})
      }   
    }
    else if (type === 'change') {
      if(state.hasOwnProperty(action.itemType)) {
        return Object.assign({}, state, {[action.itemType]: action.newItem})
      }
    }
    else if (type === 'delete') {
      if(state.hasOwnProperty(action.itemType)) {
        delete state[action.itemType]
        return  Object.assign({}, state)
      }
    }
    else {
      return Object.assign({}, state)
    }
  }

  return {
    getState,
    dispatch
  }
}
```

GOTCHAS! YOu may notice something funny in our new code. Adding and changing don't do anything without first creating a copy of the state object. But deleting does. And that means we are actually deleting something on the original object before we make a copy of it. So we don't want to do that. There are ways to do this, but it actually addresses another issue of healthy practices. If we start adding and removing state fields, it's hard to build outide functions that rely on those pieces of state existing. So, from now on, we'll only add or delete values, not keys themselves. That way any function relying on a certain key will always find it, even if the value it points to is '', [], undefined,  or null. So, one more time for good luck:

```javascript
function createStore() {
  let currentState = {
    name: 'Brendan',
    fruit: 'Passion fruit',
    color: 'Purple',
    age: 35,
    hair: false    
  }
  let isDispatching = false

  function getState() { 
    return currentState 
  }

  function dispatch(action) {
    if(!action) {
      return
    }

    if(typeof action !== 'object') {
      return
    }

    if(!action.hasOwnProperty('type')) {
      return
    }

    if(isDispatching) {
      return
    }

    isDispatching = true
    currentState = changeState(currentState, action)
    isDispatching = false
  }

  function changeState(state, action) {

    const type = action.type

    if (type === 'change') {
      if(state.hasOwnProperty(action.itemType)) {
        return Object.assign({}, state, {[action.itemType]: action.newItem})
      }
    }

    else {
      return Object.assign({}, state)
    }
  }

  return {
    getState,
    dispatch
  }
}
```

### Chapter 10: Reduction is the name of the game

Now that we are using Object.assign, we can talk about what it does with a cool new word, 'reducing'. Reducing just means taking a lot of things and mashing them together into one. Isn't that just adding (or concatenating in programming terminology)? Yes, yes it is, thank you for asking. The difference is, when we reduce we also change the stuff we're mashing together so that it fits better. This usually means overriding repeated elements, which is how we are using it to update our state copies. So really our changeState is just a giant reduce function. It takes one object (state) and a second object (which we create from the information in our action message/object) and mashes them together to create a new, single object. So in honor of understanding what changeState really does, maybe we should give it a more appropriate name? After all, it isn't changing the state anymore so much as taking old state, new action and reducing that into new state. Something like this:

```javascript
function reducer(oldState, newStuff) {
  return Object.assign({}, oldState, newStuff)
}
```

And that's exactly what we are doing. We've made a reducer. So let's rename 'changeState' to 'reducer'. While we're at it, for good measure, maybe we should fill in some of those error checks in our dispatch function as well. Right now when something is wrong with the action message we just exit the function without trying to change the state, but maybe we should at least give some kind of heads up to let usknow what exactly the problem was. Typically we'd use Errors, just for the sake of simplicity, we'll stick to good ol' console.log:

```javascript
function createStore() {
  let currentState = {
    name: 'Brendan',
    fruit: 'Passion fruit',
    color: 'Purple',
    age: 35,
    hair: false    
  }
  let isDispatching = false

  function getState() { 
    return currentState 
  }

  function dispatch(action) {
    if(!action) {
      console.log('You forgot to dispatch an action!')
      return
    }

    if(typeof action !== 'object') {
      console.log('Actions need to be a simple object.')
      return
    }

    if(!action.hasOwnProperty('type')) {
      console.log("Actions need to have a 'type' key." )
      return
    }

    if(isDispatching) {
      console.log('You cannot dispatch an action from inside a reducer!')
      return
    }

    isDispatching = true
    currentState = reducer(currentState, action)
    isDispatching = false
  }

  function reducer(state, action) {
    const type = action.type
    if (type === 'change') {
      if(state.hasOwnProperty(action.itemType)) {
        return Object.assign({}, state, {[action.itemType]: action.newItem})
      }
    }

    else {
      return Object.assign({}, state)
    }
  }

  return {
    getState,
    dispatch
  }
}
```

While we're at it, I'd like to make one more change. This one isn't necessary, but it's a common practice for Redux, so we might as well try it out. Right now, our reducer is using if...else if...else to decide what to do when it gets an action.type. There's nothing wrong with that, but another way to do the same thing is using a 'switch' statement. At the top of our switch statement we declare what the value is that we use to choose what action to take and then we store each of those different actions in a 'case'. Just like our 'else' at the bottom, we'll use a 'default' to handle occasions when nothing needs to change.

So this: 

```javascript
function reducer(state, action) {
  const type = action.type
  if (type === 'change') {
    if(state.hasOwnProperty(action.itemType)) {
      return Object.assign({}, state, {[action.itemType]: action.newItem})
    }
  }

  else {
    return Object.assign({}, state)
  }
}
```

becomes this:

```javascript
function reducer(state, action) {
  switch(action.type) {
    case 'change':
      if(state.hasOwnProperty(action.itemType)) {
        return Object.assign({}, state, {[action.itemType]: action.newItem})
      }
      else {
        return Object.assign({}, state)
      }
    default:
      return Object.assign({}, state)
  }
}
```

A small note. Switch statement are a bit different in that you can activate multiple 'case' blocks with one value. It will keep going down the list until it hits a 'break' or a 'return' statement. That's why we had to add the 'else' into our 'change' case. Yes, we could have let it naturally hit the 'default' case and we'd have no problem. But as soon as we start adding new cases, it will run through those first.

Keep in mind, the above syntax isn't necessary. It's just what people usually use with Redux (myself included) and I wanted you to get comfortable with seeing it. I personally find it makes scanning a reducer a bit easier on the brain if nothing else.

### Chapter 11: Generics aren't just cheaper, they're better

If you look at our new createStore function, you might notice something. Depending on what kind of program we write, our reducer might have a different decision tree and that would be because our state had a different structure. But other parts, in this case our dispatch method, won't change based on different kinds of state. That means we could reuse our createStore function in other programs (or in other parts of the same program) to create different stores that held different states with different reducers, as long as we don't hard code the parts that might change into the parts that won't.

So how do we do this? How do we make createStore more dynamic? Right now, we aren't calling createStore with any arguments. That means we always return the same result. But if we pass in our reducer and our state, we can reuse createStore as much as we want. While we're at it, since we are now passing in a reducer, we should at least perform a simple check to make sure the reducer is a function otherwise we'll never have a working store to begin with.

Also, before we get started, we'll want to store a copy of both our reducer and state as soon as our createStore function is called. We'll call our state object initialState now, since it's the state we use to initialize our store. As a note, we're going to name our internal reducer variable as currentReducer. This won't really serve any purpose in our simplified version of Redux since we are never going to change the reducer once we create the store, but it will serve to remind us that it would be a simple thing to do if created a method for it (and returned it in the same object as dispatch and getState). So let's see what all our pieces look like now and how we would go about creating a store:

```javascript
function createStore(reducer, initialState) {
  if(typeof reducer !== 'function') {
    console.log('We expected the reducer to be a function. Try again!')
    return
  }

  let currentState = initialState
  let currentReducer = reducer
  let isDispatching = false

  function getState() { 
    return currentState 
  }

  function dispatch(action) {
    if(!action) {
      console.log('You forgot to dispatch an action!')
      return
    }

    if(typeof action !== 'object') {
      console.log('Actions need to be a simple object.')
      return
    }

    if(!action.hasOwnProperty('type')) {
      console.log("Actions need to have a 'type' key." )
      return
    }

    if(isDispatching) {
      console.log('You cannot dispatch an action from inside a reducer!')
      return
    }

    isDispatching = true
    currentState = currentReducer(currentState, action)
    isDispatching = false
  }

  return {
    getState,
    dispatch
  }
}

const initialState = {
  name: 'Brendan',
  fruit: 'Passion fruit',
  color: 'Purple',
  age: 35,
  hair: false    
}

function reducer(state, action) {
  switch(action.type) {
    case 'change':
      if(state.hasOwnProperty(action.itemType)) {
        return Object.assign({}, state, {[action.itemType]: action.newItem})
      }
      else {
        return Object.assign({}, state)
      }
    default:
      return Object.assign({}, state)
  }
}

const store = createStore(reducer, initialState)
```

Passing in arguments definitely makes it more reusable. Do you notice a possible problem though? Right now we are passing in the reducer and the initial state separately. That makes it possible for us to use whatever reducer and whatever state we want. Except, as you may have noticed, the reducer decision tree is completely dependant on what's inside the state object. That means we need to use our reducer only with the state it was designed for. The two pieces are inextricably linked and we should treat them as such. That means where one goes the other follows. So what's the best way to accomplish that?


### Chapter 12: Default is not a place where earthquakes happen

Right now a reducer takes in state and returns new state. We've already seen in our store above that we will keep storing that state in a currentState variable. Whenever dispatch is used, it grabs that currentState and throws it into the reducer and then takes the output and puts it back in currentState. But the very first time we run createStore nothing is inside the currentState. If we tried to use it, it would be 'undefined'. If we ran dispatch with it (AND an action -- remember, dispatch exits before completeing if it doesn't get an action with a 'type' key), the reducer would take the state we passed (which in this case is 'undefined') and skip to it's default case (unless we gave it a predefined action) which just returns that state to us unchanged (which would still be 'undefined'). How do we get our initialState into this process and still keep it linked to the reducer that acts on it?

We can use a neat little trick to always tie them together. Javascript lets us use something called default arguments. That means if an argument isn't provided when a function is called (which is the same as passing in 'undefined' or 'null' in this case), it will use the default instead. It looks like this:

```javascript
let defaultName = 'John Smith'

function sayName(name = defaultName) {
  console.log('Hello' + name)
}
```

So, when we call it with a name:

```javascript
sayName('Brendan Beltz') // 'Hello Brendan Beltz'
```

And when we call it without:

```javascript
sayName() // 'Hello John Smith'
```

So, how can we use this to tightly couple or state and reducer? We pass the state as a default:

```javascript
let currentState = {
  name: 'Brendan',
  fruit: 'Passion fruit',
  color: 'Purple',
  age: 35,
  hair: false    
}

function reducer(state = currentState, action) {
  switch(action.type) {
    case 'change':
      if(state.hasOwnProperty(action.itemType)) {
        return Object.assign({}, state, {[action.itemType]: action.newItem})
      }
      else {
        return Object.assign({}, state)
      }
    default:
      return Object.assign({}, state)
  }
}
```

So now when the reducer is called without a state argument (or with 'undefined'), it uses a default (or, in this case, initial) state. That means we don't have to pass in our reducer and initialState separately to the createStore function anymore. We can pass initialState to the reducer as a default and then pass the reducer to createStore.

What's the problem with this though? If we call dispatch as soon as we create the store, it will take the default initialState and save it to our currentState. But if we call getState before dispatch, all we'll get back is 'undefined' since nothing is inside currentState yet. That means we need to call an initial dispatch action as soon as we make our store, before a user has a chance to try and use getState on something that doesn't exist yet.

Another thing to bear in mind is that we what this initial dispatch action to just return the initialState, that means we want to make sure whatever the action type is, it doesn't match one of our reducer cases. We want to ensure it hits the default at the bottom and just returns the initialState. How can we do this? How can we know for sure what action type will never be created by a user of our Redux? Well, we can't. We can use an old trick in programming which is 'reserving' keywords (that's why you can't name your variables in javascript things like 'if' 'default' or 'class', they are reserved and confuse the compiler). So, let's do that. We'll create an INIT action since what we are doing is performing our initialization action.

```javascript
const initialAction = {
  type: 'INIT'
}
```

Now, let's be honest with ourselves. It would be pretty easy for someone to accidentally name one of their reducer cases INIT in the future. It's a pretty common word in programming and we don't have any easy ways of preventing it. So let's add on a whole bunch of random characters at the end just to make sure that kind of accident doesn't happen. Remember, it doesn't matter what the 'type' is as long as it doesn't match anything but the default case of a reducer.

We'll generate a long random number and then turn it into a string, truncate it to it's first 7 digits, stick a bunch of punctuation marks in between the numbers, and stick it to our 'INIT' string. We'll even add some redux specific text at the fron. The end result will look something like '@@redux@@reserved/INIT.3.6.8.4.2.9.6.2.5.6'. If someone else ever accidentally creates a reducer with a case that matches that, well, clearly they have a million monkeys and a million typewriters and a lot of time on their hands!

```javascript
const initialAction = {
  type: '@@redux@@reserved/INIT' + Math.random().toString().substring(7).split('').join('.')
}
```

(Note: Redux uses toString(36) which gives back a mixture of letters and numbers, but it doesn't really matter. We're just trying to make a unique string.)

So, let's put all the pieces together:

```javascript
function createStore(reducer, initialState) {
  if(typeof reducer !== 'function') {
    console.log('We expected the reducer to be a function. Try again!')
    return
  }

  const initialAction = {
    type: '@@redux@@reserved/INIT' + Math.random().toString().substring(7).split('').join('.')
  }
  let currentState = initialState
  let currentReducer = reducer
  let isDispatching = false

  function getState() { 
    return currentState 
  }

  function dispatch(action) {
    if(!action) {
      console.log('You forgot to dispatch an action!')
      return
    }

    if(typeof action !== 'object') {
      console.log('Actions need to be a simple object.')
      return
    }

    if(!action.hasOwnProperty('type')) {
      console.log("Actions need to have a 'type' key." )
      return
    }

    if(isDispatching) {
      console.log('You cannot dispatch an action from inside a reducer!')
      return
    }

    isDispatching = true
    currentState = currentReducer(currentState, action)
    isDispatching = false
  }

  // When the store is first created, we want to dispatch an initial action that will
  // make sure our reducer returns it's initial state object and populates our
  // currentState variable before getState can ever be called.
  dispatch(initialActon)

  return {
    getState,
    dispatch
  }
}

const initialState = {
  name: 'Brendan',
  fruit: 'Passion fruit',
  color: 'Purple',
  age: 35,
  hair: false    
}

function reducer(state = initialState, action) {
  switch(action.type) {
    case 'change':
      if(state.hasOwnProperty(action.itemType)) {
        return Object.assign({}, state, {[action.itemType]: action.newItem})
      }
      else {
        return Object.assign({}, state)
      }
    default:
      return Object.assign({}, state)
  }
}

const store = createStore(reducer)
```

And there we have it, a fully functioning store that accepts different reducers and state objects!


## Part 2: Using best practices to manage Actions

### Chapter 13: Lights, Camera, ACTION!

We have, for all intents and purposes, a functioning state management device. Yes, there are some things missing and we'll address some of those later. But first let's look into another aspect of what he have already that we haven't really explored much. Actions. If they don't seem that complicated, it's because they aren't. All an action is is an object with a 'type' key. This object can come from anywhere and have anything other data inside it as well. So what is there left to really say? 

If you've tried using the real Redux library before, and your sense of confusion led you here, there's a decent chance a small part of that confusion arose out of the many different files that Redux seems to be split into. Reducers, Actions, Action Creators, Action Types... (Not to mention all the extra stuff that comes with using Redux in React (Providers, connect, mapStateToProps, mapDispatchToProps...)) 

The reason for this crazy file splitting isn't because it's necessary, but because it's useful. Let's consider why that is now. While actions seem pretty straightforward on their own, we need to consider them as being part of a greater system. And to that extent we should see pretty obviously that they share a strong connection to our reducer. After all, the 'type' value of an action needs to match a 'case' in our reducer in order for the reducer to do anything (except of course when we specifically want to just return the initialState in our INIT action). Following that reasoning, we realize there are two limitations we should be placing on ourselves in order to avoid actions not matching the reducer correctly. 

First of all, the string they both use needs to be the exact same, so why not just use the same string to begin with? That's what an Action Type is. It's just us storing a string in separate location so when we refer to it in multiple places in our program, it's always guaranteed to be the same. That's why it's in it's own file typically, and reducers and actions will just import the string variables they need insted of us having to hard code that string in two separate places to match. It looks something like this:


```javascript
const CHANGE = 'CHANGE'
const DO_SOMETHING = 'DO SOMETHING'
```

Why is it uppercase now? Well, that's in line with a convention in programming that says variables that will never change should be written in all caps. It doesn't do anything, it just lets people reading your code feel secure that the variable will never change (using const instead of let helps with that too of course!)

We're going to see how sharing action types looks by making a new reducer and new actions to play with, but before we do that, it would be a good idea to review a bit of modern Javascript syntax we'll want to start using.


### Chapter 14: Hoist the main sail, ES6 ahoy!


We're going to start using some ES6 (the new, cool Javascript) style functions. If they are new to you it might seem a bit strange at first, especially because we won't use the function keyword anymore. That isn't completely crazy. After all, we already can declare functions as variables if we do them like this:

```javascript
const sayHello = function(name) {
  console.log('Hello ' + name)
}
```

Which is almost the exact same as this:

```javascript
function sayHello(name) {
  console.log('Hello ' + name)
}
```

There is one big difference between the two, something called hoisting. To hoist means to lift up and, in the same sense, when something is hoisted in Javascript it's lifted up to the top of the program. Huh? Typically when you run a program it starts at the top and runs to the bottom, line by line. It has no idea of what is coming until it gets there. But Javascript isn't really just running from start to finish. It scans the whole program before it actually starts running so it cheats and knows what's coming. Confusingly, sometimes it acts like it does and sometimes it pretends it doesn't. This is a pretty important distinction and it's worth reading more about it if you still feel confused (the keyword 'var' is famous for being problematic because of hoisting and that's why it's abandoned now in favor of 'let' and 'const' (when you know the variable won't change again)). However, for the sake of our exercise I'll just make a simple distinction. 

When you use the 'function' keyword, that function is immediately available anywhere in the program, even if you put the function at the very bottom. That's hoisting. But if you create a function by giving it to a variable like we did in the first example above, it's only available to the parts of the program that come after it is created. So from now on, the order of declarations is even more important to consider. 

The newer type of functions we will use from now on won't be hoisted because they also use a variable declaration when they are created. These are called arrow functions and they use the => fat arrow operator. Let's look at some examples of similarities between traditional functions (but declared instead of using the 'function' keyword) and arrow functions:

```javascript
const sayHello = function() {
  console.log('Hello')
}

const sayHello = () => {
  console.log('Hello')
}
```

Not so different, right? Those empty () braces are where any arguments would go. We can't leave them out completely because '= =>' would confuse the hell out of our compiler. 

There is another syntax quirk to remember as well. If you have only one argument, you don't need the braces, but if you have more than one, or none, you do. For example, these are all correct:

```javascript
const sayHello = () => {
  console.log('Hello')
}

const sayHello = name => {
  console.log('Hello ' + name)
}

const sayHello = (name) => {
  console.log('Hello ' + name)
}

const add = (a, b) => {
  return a + b
}
```

This is not:

```javascript
const add = a, b => {
  return a + b
}

// ERROR!
```

That floating comma just confuses our compiler, so we need to wrap multiple arguments in () braces.

There's a second, and much more confusing aspect to arrow functions, something called implicit return. It's really just a quirk of syntax though and not a new concept. Implicit return means that if you have a one line arrow function, you don't need the curly braces and you don't need to use the 'return' keyword, the compiler automatically assumes that's what you meant:

```javascript
const add = (a, b) => a + b
```

Is exactly the same as the version with 'return' above. 

But what about if you want to return an object? It uses the same curly braces as the area where we'd normally put lines of code to run and confuses the compiler. So we just wrap it in our () braces first and the compiler knows it's like using the one line implicit return and it's actually an object not a block of code (we also do this when we want to implicitly return something that is multiple lines of code as you may see in examples of React functional components). 

These are the same:

```javascript
const makeAction = () => {
  return {
    type: 'ACTION'
  }
}

const makeAction = () => ({
  type: 'ACTION'
})
```

Where implicit return gets really head-scratching is when we start returning functions from functions. That's when you start seeing things like this:

```javascript
const fullName = firstName => lastName => firstName + ' ' + lastName
```

I will admit to sometimes having trouble wrapping my head around the more confusing strings of implicit return you see like this. What helps me is converting that back into a traditional non-implicit 'function' style in my head until I feel comfortable with the logic:

```javascript
function fullName(firstName) {
  return function(lastName) {
    return firstName + ' ' + lastName
  }
}
```

This kind of returning functions gets into more advanced topics like currying and composition which we won't be covering, but I was hoping to at least dispell some of the mystery of lines like this:

```javascript
const iAmAScaryFunction = () => () => () => () => 'Scary'
```

That's just a lot of functions hiding inside each other like a Matrushka doll. The above is the same as this:

```javascript
function iAmAScaryFunction() {
  return function() {
    return function() {
      return function() {
        return 'Scary'
      }
    }
  }
} 
```

The first time we call it it returns the next function deeper, and when we call that new function, it returns the next one deeper, and so on. So, if we want to get the word 'scary' back, we have to call it, save the new function, call that, and so on 4 times. Or we can do this: function()()()(). This has the same effect without us having to save the new function each time it gets returned (we just run it immediately instead).

```javascript
iAmAScaryFunction()()()() // 'Scary'
```

Redux uses this technique but thankfully never goes as many levels deep in nested function returns as we did. 

So, now that we're comfortable with arrow function syntax let's start using them in our codebase when we write reducers and other external functions (we'll leave the createStore stuff as is). Not because we really need to, but to get comfortable with how people typically write React and Redux programs.

Now, back to the main action!

NOTE: If you're curious as to what details about arrow functions we skipped, they mostly have to do with the 'this' keyword. It's important to understand in some cases, but it doesn't make a difference to us at the moment. But do keep in mind that arrow functions are not hoisted. I.e. we can't use them until after we declare (create) them.


### Chapter 15: Where do actions come from? Storks?

In order to explore actions a bit more clearly, let's create a new reducer/initialState/actionType to work with.

We'll use a simple reducer 'case' that doesn't need any value except a 'type' and it automatically knows to add one to our state value (or in computer parlance, increment the state by one). 

```javascript
// Action Type
const INCREMENT = 'INCREMENT' 

// Initial state object
const initialState = {
  count: 0
}

// Reducer (in cool, shiny ES6)
const reducer = (state = initialState, action) => {
  switch(action.type) {
    case INCREMENT:
      return Object.assign({}, state, {count: state.count + 1})
    default:
      return state
  }
}
```

Now, we also need to create a store that gives us a function (dispatch) that we can use to send actions to the reducer. Our createStore hasn't changed since the end of chapter 12. So we'll create a store using that same function:

```javascript
const store = createStore(reducer)
store.getState() // {count: 0}
```

Now, let's send it an action using the same action type that the reducer uses:

```javascript
store.dispatch({type: INCREMENT})
```

...and see what happens:

```javascript
store.getState() // {count: 1}
```

Yay! Everything works. But let me ask you a question, what stops someone else from dispatching a different action that our reducer isn't prepared for? Or sending information in an action type that the reducer doesn't know how to use? Nothing really. That's where Actions and Action Creators come in. The idea is that we create a gatekeeper. If a person wants to change the state in the store (ie dispatch an action to the store) than instead of directly creating the action, they use a small function that creates the action for them (an Action Creator). Since the Action Creator is a function that creates the function, if it gets arguments it doesn't like it can just ignore them:

```javascript
const actionCreator = () => ({
  type: INCREMENT
})
```

So now we have a simple function that creates (and implicitly returns) an action object. If we give that to our dispatch function it will dispatch the action object to the store. So now instead of creating an object directly we just call the function that does it for us:

```javascript
store.dispatch(actionCreator())
```

So, if a user tried to add extra information to the action, our action creator will disregard it since it doesn't care about any arguments. The above example sends the exact same action to the store as the following:

```javascript
store.dispatch(actionCreator('asfsdfjsdjkf', 34535345, ['Hello', 'can', 'anybody', 'hear', 'me', '?']))
```

This is really helpful when we do want extra arguments. Now we can do all the error and type checking in our action creator instead of in our main program. We can make sure strings are strings, numbers are the right size, arrays aren't objects... For example, we'll use an argument:

```javascript
// Action Type 
const ADD_PERSON = 'ADD PERSON'

// Action Creator
const addPerson = (name, age) => {
  if(typeof name !== 'string' || typeof age !== 'number') {
    console.log('Please provide a name and age, a string and number respectively')
    return
  }

  if(age < 0 || age > 150) {
    console.log('Please provide a realistic age. We don\'t believe you')
    return
  }

  if(name === 'Brendan Beltz') {
    console.log('Please provide a name that doesn\'t sound so silly')
    return
  }

  return {
    type: ADD_PERSON,
    name: name,
    age: age
  }
}
```

By the way, there is another cool, but occasionally confusing piece of Javascript syntax that we should cover. If you look at the above action object we return from the action creator, you may notice that we are creating keys in the object and giving them values that match variables with the same name as the keys. There is a shortcut (remember, it does the exact same thing, just a shorthand version). If we just pass the name of the variable as the name of the key, without any colons or values, it will automatically create a key with the name of the variable and then give it the value inside the variable. For example:

```javascript
const name = 'Brendan'
const age = 35

const objectOne = {
  name: name,
  age: age
}

const objectTwo = {
  name,
  age
}

console.log(objectOne) // {name: 'Brendan', age: 35}
console.log(objectTwo) // {name: 'Brendan', age: 35}
```

You can even mix and match in the same object, sometimes giving keys values, and sometimes using shorthand properties. This is fine:

```javascript
const name = 'Brendan'
const age = 35

const objectOne = {
  favoriteColor: 'purple',
  name,
  age,
  mood: 'hungry'
}
```

There is a secondary benefit to using action creator functions instead of making our own action objects, it means we can't create new types of actions. Which makes sense of course because actions need to match the reducer cases. So the difference between sending an action object that the reducer can't understand and trying to use an actionCreator that doesn't exist, is that you'll get better error reporting while you are building your program. The former might return state unchanged accidentally, but the latter will crash your program and force you to fix the issue before continuing (this is a good thing!).

So to recap, as a matter of good practice, not because it's essential, we'll share action types between our reducer and actions. Furthermore, we'll use action creator functions to create those actions and prevent anybody from sending their own, non-sanctioned actions to our store. A security guard if you will.

## Section 3: Subscriptions

### Chapter 16: Hello? What's going on in there?

At this point we've done most of the heavy lifting for our tutorial. We've built our own simplified version of Redux, at least as far as changing and viewing state goes. And we've created a very clear set of guidelines for how to send messages to change the state inside of our store. Hopefully by now the core concepts behind Redux feel like second nature to you. We're going to focus now on extending our Redux to make it more useful by adding more public functions to the ones we already have (dispatch, getState). 

We've spent a lot of time thinking about the best way to change (dispatch) state. But what about getting it back? All we can do right now is manually ask for the state with getState. But how do we know when to ask? How do we know when the state has changed? If we call getState right after dispatch, what happens if dispatch takes longer than getState to complete? We won't see the new state at all.

We're going to take advantage of another useful pattern that is used in all kinds of languages and all kinds of programs, not just Redux. The Publisher-Subscriber pattern, Pub-Sub for short. (You may have run across something called the Observer pattern as well. The two aren't conceptually different, the difference lies in how you implement the concept.)

Pub-sub is pretty much what is sounds like (the long version that is, we're not talking about bar sandwiches after all), a way for things to subscribe to something and receive something back when it is published. Kind of like when you set up an automated google search alert on your name. 

Right now, we have a few things that happen when we change the store's state. We have a dispatch function that takes a message and then sends it to the reducer function. The reducer takes the action and gives back to the dispatch function a new version of the store's state. The dispatch function then saves that new state in a variable (currentState) that is available to the rest of the store (like getState). And that's pretty much it. 

Where do subscriptions come into this? Well, if we want other programs using this store to get notified whenever the store's state changes, we need to have a way for them to subscribe (listen) to the store. That means a subscribe function (and unsubscribe, natch). It also means we need to keep a list of all of our subscribers/listeners (there could be more than one place in a big program that would want to receive notifications of store changes). And at some point after the state has been changed we need to go through that list of listeners and notify them of the change (ie send them a copy of the new state). The most obvious location for that would be right after dispatch updates currentState. So why not just stick the notification logic right inside dispatch? Good idea, that's exactly what we'll do! 

However, before we do that, we'll need to figure out where to keep that list. We've already figured out that more than one of our store functions will need access to it: subscribe needs to add things to it, unsubscribe needs to remove things from it, and dispatch will need to read it to know where to send updates. So that means we should put it somewhere everyone can see it. How about right next to our other store variables? Good idea again, you're really on fire today! As a last note, since this is a list, we'll just use a simple array to store our listeners.

Here's our new store with a few placeholders that we'll start filling in:

```javascript
function createStore(reducer, initialState) {
  if(typeof reducer !== 'function') {
    console.log('We expected the reducer to be a function. Try again!')
    return
  }

  const initialAction = {
    type: '@@redux@@reserved/INIT' + Math.random().toString().substring(7).split('').join('.')
  }
  let currentState = initialState
  let currentReducer = reducer
  let currentListeners = []
  let isDispatching = false

  function getState() { 
    return currentState 
  }

  function dispatch(action) {
    if(!action) {
      console.log('You forgot to dispatch an action!')
      return
    }

    if(typeof action !== 'object') {
      console.log('Actions need to be a simple object.')
      return
    }

    if(!action.hasOwnProperty('type')) {
      console.log("Actions need to have a 'type' key." )
      return
    }

    if(isDispatching) {
      console.log('You cannot dispatch an action from inside a reducer!')
      return
    }

    isDispatching = true
    currentState = currentReducer(currentState, action)
    isDispatching = false

    //TODO: Notify our subscribers of changes (publish new state)
  }

  function subscribe() {
    // TODO
  }

  function unsubscribe() {
    // TODO
  }

  dispatch(initialActon)

  return {
    getState,
    dispatch
  }
}
```

Ok. So now we have a framework for what we need to handle listeners and a pub-sub pattern. How do we actually go about sending notifications?

### Chapter 17: Call me back maybe

In order for two separate programs (in this case Redux and whatever program is using Redux to manage state) to communicate, they need to have some kind of connection, right? We could of course build this in manually, a way for Redux to send and a way for the external program to receive notifications. The problem is that this means the program that uses Redux is effectively an extension of Redux because it has to build the receiver in a specific way to match what Redux does. That's not very flexible. 

Instead of hard coding a receiving protocol, we should leave that to the external program to figure out. It should be free to do whatever it wants with our notifications. We just need a way to get them there in the first place. 

How about if the external programs sent us a function that handled notifications? A listener function. We'd never have to know what's in it, just that we have to call it every time the state changes. We would always pass the new state object to the listener function, but after that our Redux store doesn't care what happens. That listener could do nothing, it could just record that changes happened but not what those changes were, it could just record the changes (like a time-travel debugger program would need to do), or it could take those changes and use them to change something inside of itself (updating based on state changes -- by far the most common use). It doesn't matter to our Redux what happens after we call the listeners with the new state. That makes it a lot simpler for us!

So a listener function will be a function that comes from the external program and does somethign to the external program based on the new state that we pass it (or just based on the fact that it is being called at all). This listener function is an example of a callback function. A callback function is a function that is meant to be executed after something else happens. It's how we handle doing things in order when we don't know how long each thing will take to do. The external program has no way of predicting when it will receive updates from the store, but everytime that happens the callback will execute and handle those changes. This means the external program is free to go about it's business as usual instead of pausing everything to wait for updates.

Let's make a very simplified version of this. First of all, we create a basic function that receives a callback function as an argument and then does some stuff and afterwards runs the callback function with state (this is basically passing it back to the external function we haven't written yet):

```javascript 
function aStore(callback) {
  let storeState = 0
  for(let i = 0; i < 1000000000; i++) {
    storeState++
  }

  callback(storeState)
}
```

Now, let's create another function that calls redux (this would be equivalent to creating a store and then subscribing to it with a listener) and passes it an internal function that changes it's internal state. We'll also add an interval that keeps displaying the program's state so we can tell when it changes:

```javascript
function aProgram() {
  let programState = 42

  function showState() {
    console.log(programState)
  }
  
  function callback(storeState) {
    programState = storeState
    showState()
  }

  showState()
  aStore(callback)
}
```

Now when we run aProgram(), it will display it's state (42) and then send a listener to redux which will count up to a billion before calling that listener with it's own internal state.

If you run that, it will display the initial program state once and then when the redux loop is finished, it will cause the program to update and show 1000000000. But remember, this is a listener, not an RSVP. As long as redux keeps doing stuff we want it to keep calling that callback/listener, not just the one time. Let's make a very simple, hardcoded listener that is automatically subscribed to the store.

```javascript
function createStore(callback) {
  let storeState = 0
  let listener = callback
  
  function dispatch(newState) {
    storeState = newState
    listener(newState)
  }

  return {
    dispatch
  }
}

function aProgram() {
  let programState = 42

  function showState() {
    console.log(programState)
  }
  
  function callback(storeState) {
    programState = storeState
    showState()
  }

  showState() // 42
  const store = createStore(callback) // Now we can do store.dispatch()
  store.dispatch(58)
  store.dispatch(100)
  showState() // 100
}
```

The output from running aProgram() should be: 42, 58, 100, 100. We are directly calling showstate on the first and last calls, the two in the middle are done by our store when it finishes updating.

And that's it. The listener is a callback, but what aProgram does inside the callback listener isn't something our store has any clue about, all it knows is that it needs to call it after the state is updated and pass in the new state. However, if you remember, the store that we've been building all this time has a getState function already. Right now we still require external callback listeners to receive the new state directly, that's not particularly flexible. What if they don't want it? Well, instead, a listener can just act whenever something changes, and if wants to get the state it can, if it doesn't, it can do something else. It's more flexible that way. Let's add that into our toy example above:

```javascript
function createStore(callback) {
  let storeState = 0
  let listener = callback
  
  function dispatch(newState) {
    storeState = newState
    listener()
  }

  function getState() {
    return storeState
  }

  return {
    dispatch,
    getState
  }
}

function aProgram() {
  let programState = 42

  function showState() {
    console.log(programState)
  }
  
  function callback() {
    programState = store.getState()
    showState()
  }

  showState() // 42
  const store = createStore(callback) // Now we can do store.dispatch()
  store.dispatch(58)
  store.dispatch(100)
  showState() // 100
}
```

If you run aProgram() again, you'll see the exact same results, but without having to pass anything to the listener at all. We just call it.

Let's implement this in our real store now.

Chapter 18: Papa can you hear me?

The biggest difference between our example subscription in the last chapter and what we want to have in our actual store is that we'll be using an array of listeners rather than a single one. Listeners also won't be passed in immediately when the store is created. We will use a subscribe function to add them later (as often as we want). So instead of calling the listener at the end of dispatch() we need to iterate through the array of listeners and call each one in turn:

```javascript
// Don't try and run this code directly since it's part of the larger createStore function
// We'll refactor our full createStore function after we finish creating the internal subscription methods
let currentState = initialState
let currentReducer = reducer
let currentListeners = []
let isDispatching = false

function dispatch(action) {
  // We'll cut out the error checking to make it easier to focus on the functionality

  isDispatching = true
  currentState = currentReducer(currentState, action)
  isDispatching = false

  for(let i = 0; i < currentListeners.length; i++) {
    const listener = currentListeners[i]
    listener()
  }
}
```

And that's it for notifying listeners that something has changed. Of course, we don't have any listeners right now so it's pretty meaningless. Let's figure out how to do that next.

Chapter 19: Subscriptions are rising

We need to create a new public function that takes in a callback from an external source and adds it to our currentListeners array. Something like this:

```javascript
let currentListeners = []

function subscribe(listener) {
  currentListeners.push(listener)
}
```

Hold on a second, did I say something like that? I meant that exactly. Yeah, that's it. Sure, we should do some error checking, especially to make sure the listener is actually a function, but otherwise, that's pretty much how subscribing works. 

There is a problem with this approach though and that is we are directly changing (mutating) our array. If we've followed one core tenet throughout this guide it's that we should avoid mutations whereever possible. For example, if one part of our program tries to unsubscribe while another part is calling dispatch which will also be using that array at the same time. How do we avoid that? Well, we create a new copy to change instead of changing the old one directly. Dispatch will use the array of current listeners, while any changes we want to make we'll make to a new array of listeners (this will be the array of listeners we use in future dispatch calls so we can call it nextListeners).

The nextListeners array will start as a simple copy of the currentListeners but before we make changes to it (like adding new subscribers) we'll check to see if it is actually the same array reference as currentListeners. If it is we'll need to create a fresh copy for it to make changes to. Since the idea is to make sure we can change (mutate) our nextListeners array without affecting the currentListeners array (which now becomes a historical record of previous listeners), we can create and call a helper function ensureCanMutateNextListeners(). It really rolls off the tongue, right! Here's what it should look like:

```javascript
let currentListeners = []
let nextListeners = currentListeners 
// Both variables currently point to the same array container when our store is created

function ensureCanMutateNextListeners() {
  if(currentListeners === nextListeners) {
    nextListeners = currentListeners.slice() 
    // Remember, this gives us a fresh copy, not just a reference to the same array
  }
}
```

That's all there is to it. Whenever we run that function we can be sure that our current array of listeners won't be affected by any changes we make. So we'll run this function before we add listeners and before we remove them:

```javascript
let currentListeners = []
let nextListeners = currentListeners 
// Both variables currently point to the same array container when our store is created

function ensureCanMutateNextListeners() {
  if(currentListeners === nextListeners) {
    nextListeners = currentListeners.slice() 
    // Remember, this gives us a fresh copy, not just a reference to the same array
  }
}

function subscribe(listener) {
  ensureCanMutateNextListeners()
  nextListeners.push(listener)
}
```

Let's also add some typical checks. We want to make sure the listener is a function and we want to make sure subscribe isn't called from inside an action (isDispatching):

```javascript
function subscribe(listener) {
  if(typeof listener !== 'function') {
    console.log('Error. Expected listener to be a function')
    return
  }

  if(isDispatching) {
    console.log('You may not subscribe while an action is being dispatched to the reducer')
    return
  }

  ensureCanMutateNextListeners()
  nextListeners.push(listener)
}
```

But when do we update currentListeners? It doesn't matter how many new subscriptions happen or how many listeners unsubscribe until it comes time to actually call those listeners. Since the only time we really care about what listeners are currently subscribed is right before we actually call them, we should worry about this where that happens, at the end of our dispatch function.  So let's change our dispatch a little bit more.

```javascript
let currentState = initialState
let currentReducer = reducer
let currentListeners = []
let isDispatching = false

function dispatch(action) {

  isDispatching = true
  currentState = currentReducer(currentState, action)
  isDispatching = false

  currentListeners = nextListeners 
  for(let i = 0; i < currentListeners.length; i++) {
    const listener = currentListeners[i]
    listener()
  }
}
```

Just to be even more clear about what we are doing, let's create a new reference to the currentListeners and just call it listeners:

```javascript
  currentListeners = nextListeners
  const listeners = currentListeners
  for(let i = 0; i < listeners.length; i++) {
    const listener = listeners[i]
    listener()
  }
```

And now, since we are taking the nextListeners reference and passing it to currentListeners and then taking that and passing it into our new listeners variable, we can just use this shorthand:

```javascript
  const listeners = currentListeners = nextListeners
  for(let i = 0; i < listeners.length; i++) {
    const listener = listeners[i]
    listener()
  }
```

Now, you may be wondering why we didn't also create a fresh copy (slice) of the nextListeners to give to currentListeners, something like ensureCanMutateCurrentListeners(). Well, we don't have to. We're not actually going to change anything, we're just going through the array of functions and calling each one. 

Chapter 21: I want to get off this crazy ride!

Now that we can subscribe and call our listeners, what exactly does unsubscribing entail? In some ways it's pretty much the opposite of subscribe. For example instead of adding to the listeners array we need to remove the listener from the array. But that's where it starts to get a bit tricker. In order to remove it from the array, we need to find it, in order to find it we need to have some way to identify it. 

But before we even begin to deal with how to identify it (and that will be the harder part), let's do a quick refresher on the general mechanics of removing something from an array.

Removing an element from an array is not as easy or straightforward as adding (pushing) it (unless it's at the beginning (array.shift()) or the end (array.pop()) of the array). Here are two ways we can do it when the location of the element is unknown but the element itself is known:

```javascript
const el1 = 'Hello'
const el2 = 'Goodbye'
const el3 = 'nanana'
let array = [el1, el2, el3]

console.log(array) // ['Hello', 'Goodbye', 'nanana']

// 1. Using splice 
// First we need the index
const indexOfElToDelete = array.indexOf(el1)
// The second argument tells splice how many elements to remove starting from the index we give it
// We just want to delete one thing, so it's 1
array.splice(indexOfElToDelete, 1)

console.log(array) // ['Goodbye', 'nanana']

// 2. Using filter
// This method does not change the original array, to save our changes we either need to save the result
// to a new array or overwrite the old array with the results. We'll do the latter (if we used a const
// declaration for our array we wouldn't be able to replace it like this)
// This method will look at each elementInArray and if it doesn't match the thing we want to remove (el2) 
// it will return it unharmed.
array = array.filter(elementInArray => elementInArray !== el2)

console.log(array) // ['nanana']
```
The real Redux favors the splice method. So we will use that too. The only problem with using splice is that we are directly changing (mutating) our original array. Thankfully, we've already created a handy method, ensureCanMutateNextListeners(), that will solve that problem for us. We can do something like this:

```javascript
function unsubscribe(listener) {
  ensureCanMutateNextListeners()
  const index = nextListeners.indexOf(listener)
  nextListeners.splice(index, 1)
}
```

Ok, that's all well and good, but the big question is how does our external program get access to this unsubscribe function, and how does the function get access to the listener so that it can find it and remove it from our array of listeners?

It's worth noting that, like many questions in programming, there isn't one right answer. For example, the most obvious solution might be to add an unsubscribe method to the store object we return at the end of createStore. Then the external program can call that with the listener it wants to unsubscribe from at a later time. But that means the external program has to keep track of the listener callback just for the purposes of calling unsubscribe later. Instead, we're going to use a method that brings us back to a concept we covered much earlier, closures.











<!-- Now, we've talked a lot about time travel debugging as one of the reasons Redux is so popular (it's also the reason it was originally created) but we haven't really seen what that looks like. We're not going to build that functionality out, but if you want to think about what the simplest version would entail, you might realize it's pretty simple. Every time we dispatch an action, we get back the new state. So far we are replacing the same variable over and over again which is erasing our history. But what if we stored those copies in an array as well? Then we could go back and forth all day loading different copies into currentState. Unfortunately, that's out of scope for us now. Instead we're going to move on to part two. Actions. -->











/////////NOTES

--use strict and global at the very end
-Talk about separation for testing.
-Talk about how reducers are actually not separate.
-We can make the second phase about how it works with react (Provider/connect)