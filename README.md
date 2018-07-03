# A Redux redux 
#### Reducers, Actions, Action Creators, Stores, Providers, Thunks, Sagas...

You're still pretty new to Javascript. But now the pressure is on to use a framework, so you've picked up React. You're starting to get the basic gist of it, there's still a lot of magic, especially when it comes to Webpack. You've been hearing about Redux for almost as long as you've been hearing about React. Everything you read tells you you'll know when you need it, don't rush it. But you know the truth is that every job description you see mentions React and Redux in the same breath. So you figure there's no time to waste. Even if your personal project that you're making to learn React is pretty simple, you still want to use the same tools the pros use. 

And then you hit the inevitable wall of jargon, endless files, and boilerplate. 

Does it really take 5 files to deal with one variable?? 

The simple answer is no. It doesn't. Redux is an amazing tool in large part because it is so simple. That simplicity also allows a great deal of flexibility in how it's structured and used. That flexibility means a lot of time has gone into finding the best way among many of doing things (best practices). And those best practices means sometimes it can be very hard as a beginner to see the forest for the trees. Our goal today is to rebuild the core functionality of redux in the most ELI5 way we can, one step at a time (iteratively).

Along the way we'll explore a lot of important Javascript concepts that might feel a bit confusing as a relative beginner and, similar to Redux, aren't really all that difficult at all. Large sections of this guide will look at these general concepts and talk with the expectation that even if you've learnt them, they may still feel a little unclear. Feel free to gloss over large portions if they feel too reductive.

By the end, we will hopefully have a clearer grasp of some of the most useful Javascript rules and patterns and, at the same time, a better understanding of Redux (and why implementing it in small scale projects isn't entirely a waste of time and effort).

I should mention that this is not a tutorial that will spend much time talking about React. Redux is used with it most commonly, but Redux is really just a vanilla Javascript library and can be used anywhere. React-Redux, however, is a library that is built to mae using Redux in React a lot easier. After we have a strong command of Redux it will be a lot easier to see what is happening in React-Redux, but we won't build that library from scratch as well, we'll just touch on it at the end to feel like we know what is going on underneath the hood (spolier alert: it's mostly about React Context (which has gotten a lot easier to work with since React 16.3)).

Finally, I would to also say, in no uncertain terms, that this guide is by no means written to cast aspersions on the documentation that surrounds Redux itself. Not only is the Redux source code clear and elegant, the creators have done an amazing job explaining and documenting their work. There is an excellent series on Egghead by Dan Abramov that introduces the library by rebuilding it from scratch, much like we are doing today. This guide is just a bit more geared towars people who may be still coming to grips with the Javascript language itself. Exposure to Redux is happening sooner and sooner for new developers and this is written with those coders in mind.

Without further ado, let's get started! 

## Part 1: Creating a Redux store

### Chapter 0: We begin at the beginning

We're going to start by working with the simplest of functions and only a few pieces of state. I'm of the strong opinion that for novices learning best practices first is counterproductive. It's impossible to understand why an approach is 'best' until you've seen a version that clearly has deficiencies. Hopefully the concepts you learn today will help you understand why the official Redux sourcecode was written the way it was as we slowly build towards it.

We'll start simple and improve as we go, just a few variables (our program's state) and a few functions that read and change them (our getters and setters respectively):

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

Not everything we are going to do as we build our simplified version of Redux is going to seem essential. But hopefully it always seems clear. For the next step, we are going to collect all of our state into an object. Like good housekeeping, we're going to hide all of those global variables so they don't create a mess (global just means anything, anywhere in the program can access them - it doesn't necessarily seem like such a bad idea, it's certainly more convenient. But what happens when more than one thing has the same name? Either your program starts doing weird stuff or telling bad puns. Or both):

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

Now that we have our state (variables) in one place, why not do the same with the functions we use to interact with it? By moving our getters and setters inside the the state we now have everything in one place.

```javascript
let ourState = {
  mood: 'Happy',
  children: ['Bobby', 'Billy', 'Betty', 'Bobo'],
  showMood: function() {
    console.log(this.mood)
  },
  changeMood: function(newMood){
    this.mood = newMood;
  },
  addChild: function(newChild) {
    this.children.push(newChild)
  }
}
```

You'll notice that we've just introduced the dreaded javascript 'this'. 'this' is notorious for tripping up coders and introducing all kinds of bugs. For our sake, what's important to understand is that when 'this' is used inside a function, it points to the object/{} that the function is inside of. so, showMood is a function inside of the ourState object. That means 'this' in showMood is the same as saying 'ourState', so this.mood === ourState.mood.

So now, to change our state, we do things a bit differently:

Instead of directly changing the state:

```javascript
ourState.mood = 'Sad'
```

We tell the state to change itself (but we don't worry about how):

```javascript
ourState.changeMood('Sad')
```

This doesn't seem like a big change, but it introduces encapsulation or separation of concerns. The larger program no longer has to worry about what's happening inside of the state object which, as we progress, will more and more come to resemble it's own little mini-program (i.e. a library). What's great about this is, the outer program can always use the same interface (group of methods like getters and setters) to talk to the inner program (this is what's called an API) even if the inner program gets a lot more complex. So, let's take advantage of this and add in our first bit of complexity to the setter function changeMood: type validation. We'll keep it simple, we know that a mood should be a word which is a string in programming terms. We wouldn't want this:

```javascript
changeMood(0.7734)
```
We won't worry beyond making sure the new mood the setter is being asked to use is in fact a string. Like so: 

```javascript
function changeMood(newMood) {
  if(typeof newMood === 'string') {
    ourState.mood = newMood
  }
}
```

From now on, we can handle all the validating (checking to make sure stuff is what it's supposed to be) inside of the state library, which means our outer program is free to focus on it's own problems (and if it's anything like most programs, there will be plenty and half of them will involve missing commas). So now we have:

```javascript
let ourState = {
  mood: 'Happy',
  children: ['Bobby', 'Billy', 'Betty', 'Bobo'],
  showMood: function() {
    console.log(this.mood)
  },
  changeMood: function(newMood) { 
    if(typeof newMood === 'string') {
      ourState.mood = newMood
    }
  },
  addChild: function(newChild) { 
    this.children.push(newChild)
  } 
}
```

### Chapter 3: The plot thickens

If it feels like we're covering a lot of basic ground tediously, it's because we are. The idea is that by the end of the process it will seem downright simplistic. For the most part, the how of Redux is simplistic, it's the why and the syntax that tends to leave beginners confused.

> On a side-note, since we won't build anything that takes advantage of some of the coolest benefits of Redux, such as time-travel debugging, we should at least take a brief moment to mention about them. Time-travel debugging for instance sounds really crazy but it's really only time travel in the way that rewind and fast-forward on a VHS player are time travel. Sadly, you can put away your sports almanacs now.
>
> One of the biggest advantages of Redux is being able to see a perfect record of how changes are made to your state. Redux never actually changes anything directly, it gets instructions about what to change, and then creates new versions of our state based on those instructions. This gives us the potential to save a history of those instructions and versions of the state. So we could browse through all the state changes in the order they occured when something goes wrong (time-travel debugging). 
>
> Constantly creating new copies of the state and pretending they are the same thing also helps get us closer to avoiding another problem, race conditions. Note, this is race in the sense of the tortoise and the hare. The basic gist is that when you have two or more things going on in your program that will eventually have an effect on the same thing (such as a piece of state/variable) it can create unpredictability if you don't know which will finish first. In other words, when the order of things happening matters but you can't guarantee what order they will happen in you have race conditions. Sometimes it's not important and sometimes one instruction is 'jump out a plane' and the other is 'put on a parachute' and the order really, really matters!
>
> When you have two actions in redux they never actually change the same thing, one action will create a new, altered copy of the state and the second action will create a copy of that state. It doesn't solve the problems with the order of actions, but it means two things aren't trying to change the same thing at the exact same time. And it means we can actually go back and see what the order is if we weren't sure (time travel debugging again). It's not as exciting as a real Delorean but it definitely can be helpful.

At this point in our program we have a state object that is a random assortment of variables and functions. It might make sense to bundle all that state together into it's own object just like we did before.

From:

```javascript
let ourState = {
  mood: 'Happy',
  children: ['Bobby', 'Billy', 'Betty', 'Bobo'],
  ...
}
```

To:
```javascript
let ourState = {
  state: {
    mood: 'Happy',
    children: ['Bobby', 'Billy', 'Betty', 'Bobo']
  },
  ...
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
(NB: object.hasOwnProperty(key) let's us find out if a certain key already exists in an object. We'll use this in the following exercise to find out if something already exists before trying to add it or change it or delete it.)

Continuing on, we now have one function to change our state (setter). However, there are other ways to change state and that means we'll eventually need a lot of functions again. Deleting, adding, apppending... 


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

You might notice that I added actionType as the first argument instead of the third. That's because we don't always need an itemValue (when we delete something, for example), and obviously our new changeState function needs an actionType to work correctly. Some actions might not even need a second argument (itemType) if, for instance, all they do is delete everything, or change everything etc. Each action might end up needing different types of keys and values. If we hardcode the arguments our changeState function needs, it might get very tricky. So let's make it more flexible. Instead of passing in separate arguments, we can wrap all of the options into an object. That means our changeState doesn't really need to care about the order of arguments or which ones we give it -- only the IF block needs to care. Well...with one big exception: The whole thing breaks if we forget to specify the action type. We should probably do a check to make sure our new action object has a type key first. We should also check if the action argument is an object (and if it even exists in the first place!)

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
    // Remember, 'return' in one of these IF blocks will stop the rest of the function from running. These are called guard clauses because they allow for an early exit and protect the rest of the function from running with incompatible data.

    if(typeof action !== 'object') {
      return
    }
    if(typeof action.type === 'undefined') {
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

> Before we go any further, if you are asking yourselves how we validate new input (like making sure a new mood is a string), then you have been paying attention! As it stands, we no longer do that in the API since it no longer knows what types of state it is adding/replacing/deleting - it is now an agnostic API. When we have built it up a bit further we'll start to see likely places to put this kind of validation again. If you wanted to stop here and use this store object in a real program, you'd simply have to move that validation back into the outer program before it called ourStore.changeState().

### Chapter 6: A new complication makes things easier

Earlier we touched briefly on race conditions. When problems are caused because we can't predict the order of actions (which means we also can't predict the state that the actions act on). So there are a few things that we should consider doing to make changeState() more predictable. 

Let's focus on all those action checks first. Right now we are receiving action objects directly into the same function that is changing the state. What if we created a separate function that was responsible for processing actions? Then we could handle all the action validition checks and other stuff (like saving a log of actions for time travel debugging) in one place and just focus on changing state in another place.

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
    if(typeof action !== 'object') {
      return
    }
    if(typeof action.type === 'undefined') {
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

At first glance this does seem to complicate things unnecessarily. But setting up a gatekeeper like this, gives us a great place to do things related to actions that isn't just specifically changing the state. Later on we'll add in a log of actions. But for now we will focus on another problem. What happens if the changeState() is fired more than once from the same receiveActionAndSendActionToChangeState? Remember, we don't want to have that kind of unpredictability. What would cause that to happen? Well, one issue could be if someone tried to send another action from inside of the first one. So we should definitely make sure only one changeState() can fire at a time from one receiveActionAndSendActionToChangeState call. How do we do that though? If you're imagining it comes down to some fancy, advanced programming techniques, you might be surprised. Often the most important programming acronym of all is KISS, keep it simple, ~~stupid~~ silly! We're just going to use a simple boolean, to tell us if the function is already running. Something like this:

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

Now you may notice two things about the above code. First of all, as long as changeState hasn't resolved (finished running) we won't be able to fire another one. That means we don't have to worry about someone nesting changeState calls. The second thing you might notice is that we referenced a variable isChangingState that we never declared. That doesn't work of course, but where should we declare it? Well, it can't be inside this function because then if the function gets called twice we also get two instances of isChangingState which negates the purpose of only having one. So we'll have to store it outside the function, so even if the function is called multiple times, each instance of the function points at the same isChangingState instead of it's own local version.

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

There's another benefit to storing isChangingState externally, we can use it with other functions to see what is happening inside receiveActionAndSendActionToChangeState at any time and make decisions based on that. Right now we don't want to have attempts to change state happen while the state is actively changing. It creates unpredictability that we are working hard to avoid. To that same end we should also prohibit users from trying to get state while the state is actively changing as well. So let's use the same isChangingState to make sure we can only getState when the coast is clear:

```javascript
getState: function() {
  if(this.isChangingState) {
    console.log('You can\'t call getState while the state is being changed')
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

Now, that doesn't seem like very good housekeeping. So how can we protect our state from prying eyes and fingers? How can we still have access to the variables, but only using the functions we created (getState, changeState)? If you don't think the answer is a closure, than I probably need to work on my chapter titles a bit more! So, how can we do that? Let's refactor our store object into a store function (just like the container function in our example above). We don't want to return direct access to the state, but we do want to return direct access to the functions which outside programs will need access to in order to perform authorized (by us) actions on the state (this is the API). Unlike the closure demo above, we will need to pass access to more than one function. So, instead of returning a single inside function from the container function, we'll return an object that contains multiple inside functions. Like so:

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
      console.log('You can\'t call getState while the state is being changed')
      return
    } 
    return state 
  }

  function receiveActionAndSendActionToChangeState(action) {

    if(typeof action !== 'object') {
      return
    }

    if(typeof action.type === 'undefined') {
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
      console.log('You can\'t call getState while the state is being changed')
      return
    } 
    return state 
  }

  function dispatch(action) {

    if(typeof action !== 'object') {
      return
    }

    if(typeof action.type === 'undefined') {
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

It's starting to look a lot like Redux!

### Chapter 8: Teenage Mutant Ninja Functions!

Way back in chapter 3 we spent a few minutes talking about some of the features of Redux, like copying state rather than changing it directly and time-travel debugging, and then we completely ignored them until now. 

So let's take a moment to talk about mutability. First of all, that doesn't mean being able to suddenly kill the sound. Mutability is about whether something is able to change or not (in the sense of mutate, not mute). Now this may seem confusing to care about because it's rather obvious that things have to change inside of a program in order for anything to work. But the real question is how to change things. Do we tinker around with the thing itself or do we create a copy of the thing, take it to the side, change it, and then replace the original thing with the changed copy? The first way is mutable, the second is what we call immutable. Instead of changing things we keep creating new copies of the things and replacing the old ones. That's pretty much it.

So why go through all that trouble instead of just changing something directly? Well, part of it is to be cool obviously (we are in the era of bespoke everything). But it also helps us with another problem we touched on earlier, unpredictability. If one part of the program is looking at the current state of the store while another part is changing it then all kinds of craziness can happen.

So, how can we take this idea of immutability and use it in our own store? Instead of changeState directly changing (mutating) our state, let's give it a copy of the state and then receive a copy back and replace the original state. Now, where should we put all this logic? In our gatekeeping dispatch function of course!

To do this, we have to make a few changes. Dispatch will have to provide the state explicitly to our changeState, changeState will have to return new state instead of directly changing the original state object, and dispatch will have to receive that new state and replace the original state with it. We should also avoid some confusion resuing the word 'state' everywhere. Let's call our current state, before it goes through changeState, 'currentState' instead.

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
      console.log('You can\'t call getState while the state is being changed')
      return
    } 
    return currentState 
  }

  function dispatch(action) {

    if(typeof action !== 'object') {
      return
    }

    if(typeof action.type === 'undefined') {
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

Remember how we talked about creating a copy of the state that wouldn't be directly connected to the original state? Well, even though we thought we did, we actually were still changing the original state directly. This brings us into another fundamental programming concept, passing by value vs passing by reference. That basically means when you create a copy of something (either directly or by passing something into a function which automatically creates internal copies of whatever it gets), sometimes you get an entirely new copy and sometimes you actually just get a reference to the original thing (if you've ever heard of pointers and how scary they are and that's why you should stay away from the language 'C', well, funny thing, this is very similar (pointers just point at a location where some value is instead of actually representing the value itself like a normal variable would - this is very similar to a reference in Javascript)).

How can you know if you're creating a new copy or just a new reference to the original thing? Well, it's pretty simple in Javascript. You can tell by what type of thing you are copying. Strings, numbers, booleans just get copied. Anything you do to the new copies doesn't affect the old ones. Objects {} and arrays [] (and functions()) are where it behaves differently. (Side note to confuse you, in Javascript arrays and functions are also objects!)

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

Now, you may notice that the second way looks a lot like the 'spread operator' from the third array copy method, and it is. But there's a problem, it's very new, and that means not every browser knows what it means. It's basically a proposed way of doing something, but it's not a standard yet. To make sure it works everywhere it has to be translated (transpiled) back to standard Javascript. If you're trying to use Redux with React than you're probably already using Babel with Webpack to transpile your awesome new Javascript into old boring Javascript that works everywhere (especially if you're using create-react-app). And when you look up other tutorials about Redux, you'll probably see this a lot:

```javascript
return {
  ...state,
  newKey: newValue
}
```

But we don't want to complicate things. And Redux doesn't need to be used with React. So we're going to KISS and use the first method instead  which has wider browser support.

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

Remember to make sure to pass in an object where the argument 'newStuff' is above, not just any type of variable.

The bad:

Here's the thing, remember how we said that copying objects simply doesn't actually copy them? Well, just because we are using Object.assign, we're not completely out of the woods. What happens when our object contains objects (or arrays)? Well, all the magic that Object.assign performs only applies to the container object that we pass it. When it actually starts copying the stuff inside over it uses the simple method again. That means our outside container might be unique, but any containers inside of it will point to the same place as the containers inside of other copies of the outside container. Ugh. Let's see how this looks in practice:

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


This isn't ideal. This is what shallow copying is. Everything at the top level is a unique copy (in this case our outer container object 'oldObject'), but everything deeper (nested) is treated the old way. So how do we handle that? Well, we have to use Object.assign inside of Object.assign for as many layers as we need. This is a horrible idea for lots of reasons, most importantly because it gets very confusing very quickly. Ideally, we don't nest anything. Instead we 'flatten' our state object. That means keeping it shallow, one or two levels deep at the most (flattening information like this to make it easier to work with is also called 'normalizing' data -- another big word for a simple idea). There are ways to deep copy objects (making sure every nested object is also a new copy not just a new reference to the same address), but they are either harder to write (using loops or recursion) or require us to use an outside library like lodash or underscore. And either way, we want to encourage clearer structure, which in our case means flatter.

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

> Normalizing isn't a rule, just an example of a best practice.

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
    if(isDispatching) {
      console.log('You can\'t call getState while the state is being changed')
      return
    } 
    return currentState   
  }

  function dispatch(action) {

    if(typeof action !== 'object') {
      return
    }

    if(typeof action.type === 'undefined') {
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

GOTCHAS! YOu may notice something problematic in our code. Adding and changing don't do anything without first creating a copy of the state object. But deleting does. And that means we are actually deleting something on the original object before we make a copy of it. So we don't want to do that. There are ways to do this, but it actually addresses another issue of healthy practices. If we start adding and removing state fields, it's hard to build outside functions that rely on those pieces of state existing. So, from now on, we'll only add or delete values, not keys themselves. That way any function relying on a certain key will always find it, even if the value it points to is '', [], undefined, or null. When we deal with different types of state more specifically we can specify what kind of empty state we want to use but for now we'll simply use the catch-all null:

```javascript
function createStore() {
  ...
  function changeState(state, action) {
    ...
    else if (type === 'delete') {
      if(state.hasOwnProperty(action.itemType)) {
        return  Object.assign({}, state, {[action.itemType]: null})
      }
    }
    ...
  }
  ...
}
```

### Chapter 10: Reduction is the name of the game

Now that we are using Object.assign, we can talk about what it does with a cool new word, 'reducing'. Reducing just means taking a lot of things and mashing them together into one. Isn't that just adding (or concatenating in programming terminology)? Yes, yes it is, thank you for asking. The difference is, when we reduce we also change the stuff we're mashing together so that it fits better. This often means overriding repeated elements, which is how we are using it to update our state copies. So really our changeState is just a giant reduce function. It takes one object (state) and a second object (which we create from a combination of the information in our action message/object and the relevant piece of the old (current) state) and mashes them together to create a new, single object. So in honor of understanding what changeState really does, maybe we should give it a more appropriate name? After all, it isn't changing the state directly anymore so much as taking old state and a new action and subsequently 'reducing' that into new state. Something like this:

```javascript
function reducer(oldState, newStuff) {
  return Object.assign({}, oldState, newStuff)
}
```

And that's exactly what we are doing. We've made a reducer. So let's rename 'changeState' to 'reducer'. While we're at it, for good measure, maybe we should fill in some of those error checks in our dispatch function as well. Right now when something is wrong with the action message we just exit the function without trying to change the state, but maybe we should at least give some kind of heads up to let us know what exactly the problem was. Typically we'd use Errors, just for the sake of simplicity, we'll stick to good ol' console.log:

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
      console.log('You can\'t call getState while the state is being changed')
      return
    } 
    return currentState     }

  function dispatch(action) {
    if(!action) {
      console.log('You forgot to dispatch an action!')
      return
    }

    if(typeof action !== 'object') {
      console.log('Actions need to be a simple object.')
      return
    }

    if(typeof action.type === 'undefined') {
      console.log("Actions need to have a 'type' key with a value that is not undefined" )
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

> A small note. Switch statements are a bit different from if...else in that you can activate multiple 'case' blocks with one value. It will keep going down the list of cases until it hits a 'break' or a 'return' statement. That's why we had to add the 'else' into our 'change' case. Yes, we could have let it naturally hit the 'default' case and we'd have no problem. But as soon as we start adding new cases, it will run through those first.

Keep in mind, a switch statement isn't necessary. It's just what people usually use with Redux (myself included) and I wanted you to get comfortable with seeing it. I personally find it makes scanning a reducer a bit easier on the brain if nothing else.

### Chapter 11: Generics aren't just cheaper, they're better

If you look at our new createStore function, you might notice something. Depending on what kind of program we write, our reducer might have a different decision tree if our program's state has a different structure. But other parts of our store, like our dispatch method, won't change based on different kinds of state. That means we could reuse our createStore function in other programs (or in other parts of the same program) to create different stores that held different states with different reducers, as long as we don't hard code the parts that might need to change (this is what it means to make code extensible (i.e. reusable)).

So how do we do this? How do we make createStore more dynamic? Right now, we aren't calling createStore with any arguments. That means we always return the same result. But if we pass in our reducer and our state, we can reuse createStore as much as we want with any state shape we need. (While we're at it, since we are now passing in a reducer, we should at least perform a simple check to make sure the reducer is a function otherwise we'll never have a working store to begin with.)

Also, before we get started, we'll want to store a copy of both our reducer and state as soon as our createStore function is called. We'll call our state object initialState now, since it's the state we use to initialize our store. 

> As a note, we're going to name our internal reducer variable as currentReducer. This won't really serve any purpose in our simplified version of Redux since we are never going to change the reducer once we create the store, but it will serve to remind us that it would be a simple thing to do if we created a method for doing it (and returned it in the same object as dispatch and getState (redux has a replaceReducer method that does this)). 

So let's see what all our pieces look like now and how we would go about creating a store:

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
    if(isDispatching) {
      console.log('You can\'t call getState while the state is being changed')
      return
    } 
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

    if(typeof action.type === 'undefined') {
      console.log("Actions need to have a 'type' key with a value that is not undefined" )
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

Passing in arguments definitely makes it more reusable. Do you notice a possible problem though? Right now we are passing in the reducer and the initial state separately. That makes it possible for us to use whatever reducer and whatever state we want. Except, as you may have noticed, the reducer decision tree is completely dependent on what's inside the state object. That means we need to use our reducer only with the state it was designed for. The two pieces are inextricably linked and we should treat them as such. That means where one goes the other follows. So what's the best way to accomplish that?


### Chapter 12: Default is not a place where earthquakes happen

Right now a reducer takes in state and returns new state. We've already seen in our store above that we will keep storing that state in a currentState variable. Whenever dispatch is used, it grabs that currentState and throws it into the reducer and then takes the output and puts it back in currentState. But the very first time we run createStore nothing is inside the currentState. If we tried to use it, it would be 'undefined'. If we ran dispatch with it (AND an action -- remember, dispatch exits before completing if it doesn't get an action with a 'type' key), the reducer would take the state we passed (which in this case is 'undefined') and skip to it's default case (unless we gave it a predefined action) which just returns that state to us unchanged (which would still be 'undefined'). How do we get our initialState into this process and still keep it linked to the reducer that acts on it?

We can use a neat little trick to always tie them together. Javascript lets us use something called default arguments. That means if an argument isn't provided when a function is called (or when passing in 'undefined' or 'null' because of Javascript truthiness fun), it will use the default instead. It looks like this:

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

Another thing to bear in mind is that we what this initial dispatch action to just return the initialState, that means we want to make sure whatever the action type is, it doesn't match one of our reducer cases. We want to ensure it hits the default at the bottom of our switch statement and just returns the initialState. How can we do this? How can we know for sure what action type will never be created by a user of our Redux? Well, we can't. We can use an old trick in programming which is 'reserving' keywords (that's why you can't name your variables in javascript things like 'if' 'default' or 'class', they are reserved and confuse the compiler). So, let's do that. We'll create an INIT action since what we are doing is performing our initialization action.

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
    if(isDispatching) {
      console.log('You can\'t call getState while the state is being changed')
      return
    } 
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

    if(typeof action.type === 'undefined') {
      console.log("Actions need to have a 'type' key with a value that is not undefined" )
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
  dispatch(initialAction)

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

And there we have it, a fully functioning store that accepts different reducers and state objects! Because the first time dispatch calls the reducer it uses currentState which is initially undefined, the reducer will return it's (now tightly coupled) initial state.


## Part 2: Using best practices to manage Actions

### Chapter 13: Lights, Camera, ACTION!

We have, for all intents and purposes, a functioning state management device. Yes, there are some things missing and we'll address some of those later. But first let's look into another aspect of what he have already that we haven't really explored much. Actions. If they don't seem that complicated, it's because they aren't. All an action is is an object with a 'type' key. This object can come from anywhere and have any kind of data inside it. So what is there left to really say? 

If you've tried using the real Redux library before, and your sense of confusion led you here, there's a decent chance a small part of that confusion arose out of the many different files that Redux seems to be split into. Reducers, Actions, Action Creators, Action Types... (Not to mention all the extra stuff that comes with using Redux in React (Providers, connect, mapStateToProps, mapDispatchToProps...)) 

The reason for this crazy file splitting isn't because it's necessary, but because it's useful. Let's consider why that is now. While actions seem pretty straightforward on their own, we need to consider them as being part of a greater system. And to that extent it should be pretty obvious that they share a strong connection to our reducer. After all, the 'type' value of an action needs to match a 'case' in our reducer in order for the reducer to do anything (except of course when we specifically want to just return the initialState in our INIT action). Following that reasoning, we realize there are two limitations we should be placing on ourselves in order to avoid actions not matching the reducer correctly. 

First of all, the string they both use needs to be the exact same, so why not just use the same string to begin with? That's what an Action Type is. It's just us storing a string in a separate location so when we refer to it in multiple places in our program, it's always guaranteed to be the same. That's why it's in it's own file typically, and reducers and actions will just import the string variables they need instead of us having to hard code that string in two separate places to match. The other benefit is that if you type a variable name incorrectly your app will throw an error because it isn't defined, but if you misspell a string it will silently cause all sorts of problems. It looks something like this:


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

There is one big difference between the two, something called hoisting. To hoist means to lift up and, in the same sense, when something is hoisted in Javascript it's lifted up to the top of the program. Huh? Typically when you run a program it starts at the top and runs to the bottom, line by line. It has no idea of what is coming until it gets there. But Javascript isn't really just running from start to finish. It scans the whole program before it actually starts running so it cheats and knows what's coming. Sometimes, which is the confusing bit. This is a pretty important distinction and it's worth reading more about it if you still feel confused (the keyword 'var' is famous for being problematic because of hoisting and that's why it's abandoned now in favor of 'let' and 'const' (when you know the variable won't change again)). However, for the sake of our exercise I'll just make a simple distinction. 

When you use the 'function' keyword, that function is immediately available anywhere in the program, even if you put the function at the very bottom. That's hoisting. But if you create a function by assigning it to a const like we did in the first example above, it's only available to the parts of the program that come after it is created. So from now on, the order of declarations is even more important to consider. 

The newer type of functions we will use from now on won't be hoisted because they also use a const declaration when they are created. These are called arrow functions and they use the => fat arrow operator. Let's look at some examples of similarities between traditional functions (but declared instead of using the 'function' keyword) and arrow functions:

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
// That floating comma just confuses our compiler, so we need to wrap multiple arguments in () braces.
```

There's a second, and much more confusing aspect to arrow functions, something called implicit return. It's really just a quirk of syntax though and not a new concept. Implicit return means that if you have a one line arrow function, you don't need the curly braces and you don't need to use the 'return' keyword, the compiler automatically assumes that's what you meant:

```javascript
const add = (a, b) => a + b
```

Is exactly the same as the version with 'return' above. 

But what about if you want to return an object? It uses the same curly braces as the area where we'd normally put lines of code to run and confuses the compiler. So we just wrap it in our () braces first and the compiler knows it's like using the one line implicit return and it's actually an object and not a block of code (we also do this when we want to implicitly return something that is multiple lines of code as you may see in examples of React functional components). 

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

I will admit to sometimes having trouble wrapping my head around the more confusing lengthy strings of implicit return you see like this. What helps me is converting that back into a traditional non-implicit 'function' style in my head until I feel comfortable with the logic:

```javascript
function fullName(firstName) {
  return function(lastName) {
    return firstName + ' ' + lastName
  }
}
```

This kind of returning functions gets into more advanced topics like currying and composition which we won't be covering in detail, but I was hoping to at least dispell some of the mystery of lines like this:

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

The first time we call it it returns the next function deeper, and when we call that new function, it returns the next one deeper, and so on. So, if we want to get the word 'scary' back, we have to call it, save the new function, call that, and so on 4 times. 

```javascript
const firstInnerFunction = iAmAScaryFunction(); // returns const f = () => () => () => 'Scary'
const secondInnerFunction = firstInnerFunction(); // returns const f = () => () => 'Scary'
const thirdInnerFunction = secondInnerFunction(); // returns const f = () => 'Scary'
const innerString = thirdInnerFunction(); // returns 'Scary'
```

Or we can do this: function()()()(). This has the same effect without us having to save the new function each time it gets returned (we just run it immediately instead).

```javascript
iAmAScaryFunction()()()() // returns 'Scary'
```

> Redux uses this technique but thankfully never goes as many levels deep in nested function returns as we did.

So, now that we're comfortable with arrow function syntax let's start using them in our codebase when we write reducers and other external functions (we'll leave the createStore stuff as is). Not because we really need to, but to get comfortable with how people typically write React and Redux programs.

*Back to the main action!*

> NOTE: If you're curious as to what details about arrow functions we skipped, they mostly have to do with the 'this' keyword. It's important to understand in some cases, but it doesn't make a difference to us at the moment. But do keep in mind that arrow functions are not hoisted. I.e. we can't use them until after we declare (create) them.


### Chapter 15: Where do actions come from? Storks?

In order to explore actions a bit more clearly, let's create a new reducer/initialState/actionType to work with.

We'll use a simple reducer 'case' that doesn't need any value except a 'type' and it automatically knows to add 1 to our state value (or in computer parlance, increment the state by 1). 

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

So now we have a simple function that creates (and implicitly returns) an action object. If we give that to our dispatch function it will dispatch the action object to the store. A second benefit is that this is DRY, we don't have to create a new object manually each time we want to dispatch an action we just call the creator with the data that is most important. So now instead of creating an object directly we just call the function that does it for us:

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

There is a secondary benefit to using action creator functions instead of making our own action objects, it means we can't create new types of actions willy nilly inside of a component. Which makes sense of course because actions need to match the reducer cases. So the difference between sending an action object that the reducer can't understand and trying to use an actionCreator that doesn't exist, is that you'll get better error reporting while you are building your program. The former might return state unchanged accidentally, but the latter will crash your program and force you to fix the issue before continuing (this is a good thing!).

So to recap, as a matter of good practice, not because it's essential, we'll share action types between our reducer and actions. Furthermore, we'll use action creator functions to create those actions and share them between different parts of a program, which in turn prevents anybody from sending their own, non-sanctioned actions to our store. A security guard if you will.

## Section 3: Subscriptions

### Chapter 16: Hello? What's going on in there?

At this point we've done most of the heavy lifting for our tutorial. We've built our own simplified version of Redux, at least as far as changing and viewing state goes. And we've created a very clear set of guidelines for how to send messages to change the state inside of our store. Hopefully by now the core concepts behind Redux feel like second nature to you. We're going to focus now on extending our Redux to make it more useful by adding more public functions to the ones we already have (dispatch, getState). 

We've spent a lot of time thinking about the best way to change (dispatch) state. But what about getting it back? All we can do right now is manually ask for the state with getState. But how do we know when to ask? How do we know when the state has changed? If we call getState right after dispatch, what happens if dispatch takes longer than getState to complete? We won't see the new state at all.

We're going to take advantage of another useful pattern that is used in all kinds of languages and all kinds of programs, not just Redux. The Publisher-Subscriber pattern, Pub-Sub for short.

Right now, we have a few things that happen when we change the store's state. We have a dispatch function that takes a message and then sends it to the reducer function. The reducer takes the action and gives back to the dispatch function a new version of the store's state. The dispatch function then saves that new state in a variable (currentState) that is available to the rest of the store (like getState). And that's pretty much it. 

Where do subscriptions come into this? Well, if we want other parts of the program using this store to get notified whenever the store's state changes, we need to have a way for them to subscribe (listen) to the store. That means a subscribe function (and unsubscribe, natch, if they decide they don't need to listen anymore). It also means we need to keep a list of all of our subscribers/listeners (there could be more than one place in a big program that would want to receive notifications of store changes). And at some point after the state has been changed we need to go through that list of listeners and notify them of the change (whether or not we send them a copy of the new state as well is a question for the developer, in Redux's case it is not sent). The most obvious location for that would be right after dispatch updates currentState. So why not just stick the notification logic right inside dispatch? Good idea, that's exactly what we'll do! 

However, before we do that, we'll need to figure out where to keep that list of listeners. We've already figured out that more than one of our store functions will need access to it: subscribe needs to add things to it, unsubscribe needs to remove things from it, and dispatch will need to read it to know where to send updates. So that means we should put it somewhere everyone can see it. How about right next to our other store variables? Good idea again, you're really on fire today! As a last note, since this is a list, we'll just use a simple array to store our listeners.

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
    if(isDispatching) {
      console.log('You can\'t call getState while the state is being changed')
      return
    } 
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

    if(typeof action.type === 'undefined') {
      console.log("Actions need to have a 'type' key with a value that is not undefined" )
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

  dispatch(initialAction)

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

How about if the external programs sent us a function that handled notifications? A listener function. We'd never have to know what's in it, just that we have to call it every time the state changes. We would even pass the new state object to the listener function, but after that our Redux store doesn't care what happens. That listener could do nothing, it could just record that changes happened but not what those changes were, it could just record the changes (like a time-travel debugger program would need to do), or it could take those changes and use them to change something inside of itself (updating based on state changes -- by far the most common use). It doesn't matter to our Redux what happens after we call the listeners with the new state. That makes it a lot simpler for us!

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

And that's it. The listener is a callback, but what aProgram does inside the callback listener isn't something our store has any clue about, all it knows is that it needs to call it after the state is updated and pass in the new state. However, if you remember, the store that we've been building all this time has a getState function already. Right now we still require external callback listeners to receive the new state directly, that's not particularly flexible. What if they don't want it? Well, instead, a listener can just act whenever something changes, and if wants to get the state it can, if it doesn't, it can do something else. Let's add that into our toy example above:

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

### Chapter 18: Papa can you hear me?

The biggest difference between our example subscription in the last chapter and what we want to have in our actual store is that we'll be using an array of listeners rather than a single one. Listeners also won't be passed in immediately when the store is created. We will use a subscribe function to add them later (as often as we want). So instead of just calling the one listener at the end of the dispatch() function we need to iterate through the array of listeners and call each one in turn:

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

### Chapter 19: Subscriptions are rising

We need to create a new public function that takes in a callback from an external source and adds it to our currentListeners array. Something like this:

```javascript
let currentListeners = []

function subscribe(listener) {
  currentListeners.push(listener)
}
```

Hold on a second, did I say 'something like that'? I meant that exactly. Yeah, that's it. Sure, we should do some error checking, especially to make sure the listener is actually a function, but otherwise, that's pretty much how subscribing works. 

There is a problem with this approach though and that is we are directly changing (mutating) our array. If we've followed one core tenet throughout this guide it's that we should avoid mutations whenever possible. For example, if one part of our program tries to unsubscribe while another part is calling dispatch which will also be using that array at the same time. How do we avoid that? Well, we create a new copy to change instead of changing the old one directly. Dispatch will use the array of current listeners, while any changes we want to make we'll make to a new array of listeners (this will be the array of listeners we use in future dispatch calls so we can call it nextListeners).

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

### Chapter 20: I want to get off this crazy ride!

Now that we can subscribe and call our listeners, what exactly does unsubscribing entail? In some ways it's pretty much the opposite of subscribe. For example instead of adding to the listeners array we need to remove the listener from the array. But that's where it starts to get a bit tricker. In order to remove it from the array, we need to find it, in order to find it we need to have some way to identify it. 

But before we even begin to deal with how to identify it (and that will be the harder part), let's do a quick refresher on the general mechanics of removing something from an array.

Removing an element from an array is not as easy or straightforward as adding (pushing) it unless it's at the beginning (array.shift()) or the end (array.pop()) of the array. Here are two ways we can do it when the location of the element is unknown but the element itself is known:

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

> It's worth noting that, like many questions in programming, there isn't one right answer. For example, the most obvious solution to you might be to add an unsubscribe method to the store object we return at the end of createStore. Then the external program can call that with the listener it wants to unsubscribe from at a later time. But that means the external program has to keep track of the listener callback just for the purposes of calling unsubscribe later (it needs the listener to be able to find it from the array of nextListeners in order to remove it). 

We're going to use a simple method. One that brings us back to a concept we covered much earlier, closures. Remember, a function always stays connected to any variables that it references at the time of its creation, even if it subsequently gets passed back and forth all over the program. And we already have a reference to the appropriate listener inside of the subscribe function. So if we create the unsubscribe function inside of subscribe, it will always be attached to the appropriate listener. Of course we need to make sure we return that function at the same time, otherwise there won't be any way to access it later. 

> Just for good measure, we should also prevent unsubscribe from being called inside an action. We'll use isDispatching once again to make sure of that. 

When we put everything together it should look like this:

```javascript
function subscribe(listener) {
  // Error checking is temporarily removed for clarity

  ensureCanMutateNextListeners()
  nextListeners.push(listener)

  return function unsubscribe() {
    if(isDispatching) {
      console.log('You can\'t call unsubscribe while the store is dispatching an action')
      return
    }

    ensureCanMutateNextListeners()
    const index = nextListeners.indexOf(listener)
    nextListeners.splice(index, 1)  
  }
}
```

So, now we can do this:

```javascript
const callback = () => {}  // Empty, placeholder callback
const unsubscribe = subscribe(callback) // We simultaneously subscribe and save the returned unsubscribe function
unsubscribe() // We unsubscribe and wash our hands of this terrible business
```

Voila. A working subscribe/unsubscribe system.

### Chapter 21: Things can always get worse

So we have a working unsubscribe function and that's awesome but we should take a minute to also consider how it might be possible to mess things up. For example, what happens when these functions are called more than once? 

If we call subscribe multiple times with the different callback listeners, that's fine, that's actually the intended use case. If we call it multiple times with the same listener? Well, things could get weird. But, there isn't a perfect solution. Hopefully, if the listener is doing the right thing (like updating local state), at worst its just going to make the program less efficient with redundant calls. 

Where things can really go off the rails is when we call unsubscribe multiple times. Why is that? It should only work if it finds the listener it's looking for right? Well, no. Array.indexOf(element) normally returns the index of an element in an array. But if it can't find the element it returns the number -1. This can be useful, it's a common tool used when trying to find out if something is in an array at all:

```javascript
if(array.indexOf(element) !== -1) {
  //We know the element is in the array and we can do something with it
  console.log(array[array.indexOf(element)])
}
```

But the problem is we are also using Array.splice(). When we give splice a negative index it still works, it just starts counting from the end of the array instead of the beginning:

```javascript
let array = [1,2,3,4,5]
array.splice(-1, 1)
console.log(array) // [1,2,3,4]
```

Clearly that's not good. One subscriber could call unsubscribe multiple times and wipe out listeners from other subscribers. So how can we insure that unsubscribe can only be called once per listener? How about a simple boolean? We can check it before we run unsubscribe, something that tells us whether the listener is subscribed. It should start off as true when we run the subscribed function. When we run the unsubscribe function we can set it to false. If we check to make sure it's true at the top of unsubscribe, any further calls won't do anything. Remember, closures mean unsubscribe and subscribe will be able to share access to the same variables (just like they do with the listener callback). Like this:

```javascript
function subscribe(listener) {
  // Error checking is temporarily removed for clarity

  let isSubscribed = true
  ensureCanMutateNextListeners()
  nextListeners.push(listener)

  return function unsubscribe() {
    if(!isSubscribed) {
      console.log('This listener is already unsubscribed. You can\'t unsubscribe again')
      return
    }

    if(isDispatching) {
      console.log('You can\'t call unsubscribe while the store is dispatching an action')
      return
    }

    isSubscribed = false
    ensureCanMutateNextListeners()
    const index = nextListeners.indexOf(listener)
    nextListeners.splice(index, 1)  
  }
}
```

At this point we won't be adding any more functionality to our store. We'll cleanup a little bit, put all the pieces together and see what we've made in all it's glory.

## Section 4: Cleanup and error handling 

### Chapter 22: Error, error does not compute.

Up to now, we've been using console.log to notify when an error occurs. That can be helpful for a programmer doing some debugging but what happens if our external program wants to know when something goes wrong? Nothing. The program that is calling functions on the store has no way to know when an error occurs. We need to find a better way to handle errors. That's where the 'throw' statement comes in.

Throwing is pretty straightforward, but there are still a few particulars we should review. 

> Note: we can actually throw anything we want, it doesn't have to be an Error object. However, we will stick to Errors for this guide. 

A basic example of throwing an Error message looks like this:

```javascript
throw new Error('I am an error message')
```

Let's go over how throw behaves in different situations. First of all, if you call throw in the normal operation of a program it will cause that program to immediately terminate, no questions asked! However, in many cases, we want to be able to detect and deal with errors but not actually cause a program to terminate completely. In order to do that, we use another tool, try...catch blocks.

Basically, we put whatever we want to do inside the 'try' block, but because we are planning ahead for problems, we put any error handling in the 'catch' block. If anything happens inside the 'try' block that causes an error, the program will stop executing the rest of the code in the 'try' and skip down to the 'catch' area and execute that code instead (catch takes an argument which is the error that caused it to be triggered). Let's see how that works:

```javascript
try {
  console.log('Now you see me')
  throw new Error('Something went wrong')
  console.log('Now you don\'t')
}
catch (err) {
  console.log(err)
}
```

If we run the above code we should see 'Now you see me' followed by 'Error: Something went wrong' in the console. So whatever we throw will get caught by any try...catch block that it is contained in. Importantly, if there are multiple nested try...catch blocks, the first one that is triggered by the 'throw' is the only one that will see it. So if we wrap anything in try...catch inside our Redux than external programs using the store will never see the Errors either.

The onus will be on the external program to handle errors, we just want to make sure they get there. So we're going to start replacing our console.log error handling with throwing errors.

For example, something like this:

```javascript
if(typeof action !== 'object') {
  console.log('Actions need to be a simple object.')
  return
}
```

becomes this:

```javascript
if(typeof action !== 'object') {
  throw new Error('Actions need to be a simple object.')
}
```

You might notice we got rid of the return statement as well. Since 'throw' causes the function to stop running at the point it is called, it functions just like using 'return' to exit a function prematurely.

There is one other kind of error handling we should consider. Right now, in our dispatch function, we are doing this:

```javascript
isDispatching = true
currentState = currentReducer(currentState, action)
isDispatching = false
```

This piece of code seems pretty straightforward right? Nothing can go wrong with switching a boolean value, but what about the call to our reducer? At this point in a dispatch call we've already checked the action for errors. But maybe something goes wrong inside the reducer? We didn't create it after all, who knows what's in there. The difference here is that we aren't going to throw an error to the external function if something happens. Why not? Well, if we stop executing the dispatch call before we reset isDispatching back to false, the rest of the store becomes unusable until a new dispatch is called. We still want to have an error trigger for an external program to handle, but we want to make sure it doesn't cause us to terminate dispatch until after we've reset the boolean.

We're going to use a slightly different version of try..catch to do that. There's also a third block, 'finally'. It doesn't matter what happens in 'try' or 'catch', what's in 'finally' will always run afterwards. Eg:

```javascript
try {
  console.log('Now you see me') // 'Now you see me'
  throw new Error('Something went wrong')
  console.log('Now you don\'t')
}
catch (err) {
  console.log(err) // 'Something went wrong'
}
finally {
  console.log('You\'ll always see me!') // 'You'll always see me!'
}
```

Now, once again, we aren't trying to catch errors internally in our store. We still want to leave that to whatever external program is calling the store. So instead of try...catch...finally we'll drop the 'catch' and just use try...finally. Like so:

```javascript
try {
  isDispatching = true
  currentState = currentReducer(currentState, action)
}
finally {
  isDispatching = false
}
```

So, if there is an error, we'll never get to the part in dispatch where the listeners are called. But we'll at least reset isDispatching before the dispatch call terminates (because 'finally' always runs, even if the function is about to terminate because of a 'throw' error). That means it will still be possible to call other functions (subscribe, getState, unsubscribe) before our next dispatch call resets the boolean anyway.

### Chapter 23: And on the seventh line, he rested

We've come a long way since the last time we looked at our createStore in total. In that time we've added in subscriptions, mutation checkers, and better error handling. Let's put all these new elements together and see our final product!

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
  let nextListeners = currentListeners 
  let isDispatching = false

  // This is a private function, meaning we won't export (expose) it
  function ensureCanMutateNextListeners() {
    if(currentListeners === nextListeners) {
      nextListeners = currentListeners.slice() 
    }
  }

  function getState() { 
    if(isDispatching) {
      throw new Error('You can\'t call getState while the state is being changed')
    } 
    return currentState
  }

  function dispatch(action) {
    if(typeof action !== 'object') {
      throw new Error('Actions need to be a simple object.')
    }

    if(typeof action.type === 'undefined') {
      throw new Error("Actions need to have a 'type' key with a value that is not undefined" )
    }

    if(isDispatching) {
      throw new Error('You cannot dispatch an action from inside a reducer!')
    }

    try {
      isDispatching = true
      currentState = currentReducer(currentState, action)
    }
    finally {
      isDispatching = false
    }
    
    const listeners = currentListeners = nextListeners
    for(let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }
  }

  function subscribe(listener) {
    if(typeof listener !== 'function') {
      throw new Error('Error. Expected listener to be a function')
    }

    if(isDispatching) {
      throw new Error('You may not subscribe while an action is being dispatched to the reducer')
    }

    let isSubscribed = true
    ensureCanMutateNextListeners()
    nextListeners.push(listener)

    return function unsubscribe() {
      if(!isSubscribed) {
        return
      }

      if(isDispatching) {
        throw new Error('You can\'t call unsubscribe while the store is dispatching an action')
      }

      isSubscribed = false
      ensureCanMutateNextListeners()
      const index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1)  
    }
  }

  dispatch(initialAction)

  return {
    getState,
    dispatch,
    subscribe
  }
}
```

You may wonder why we aren't throwing an error in unsubscribe() when the listener is already unsubscribed. After all, we replaced all the other console.log/return instances with throw. If you think about it though, there's nothing wrong with calling this function more than once, we just want to prevent it from having any effect after the first call.

Ok. That's it, we're done. We didn't rebuild Redux completely, but we will take a moment to discuss a few of the features we haven't covered. First though, let's create a dummy program that let's us do something with our Redux redux so we can better see how it behaves. We want this to be as simple as possible, so it will just be a counter. Since we want to keep this simple, we won't split reducers, action types, and action creators into separate files. We're going to put everything in one.

First we need some action types:

```javascript
const INCREMENT = 'INCREMENT'
const DECREMENT = 'DECREMENT'
const RESET = 'RESET'
```

Then we need an initial state:

```javascript
const initialState = {
  counter: 0
}
```

Then we need a reducer to handle the different action types (you may think that since we only have a single state value to hold, it seems like overkill to use an object and you are right! State can be a single variable, an array, or an object. If the state holds more than one value it will almost certainly be an object though, so we are overbuilding here to reflect typical use cases):

```javascript
const counterReducer = (state = initialState, action) => {
  switch(action.type) {
    case INCREMENT:
      return Object.assign({}, state, {counter: state.counter + 1})
    case DECREMENT:
      return Object.assign({}, state, {counter: state.counter - 1})
    case RESET:
      return initialState;
    default:
      return state
  }
}
```

Now we need to make action creators:

```javascript
const incrementCounter = () => ({
  type: INCREMENT
})
const decrementCounter = () => ({
  type: DECREMENT
})
const resetCounter = () => ({
  type: RESET
})
```

Now we need a store:

```javascript
const store = createStore(counterReducer)
```

Ok, we're ready to roll. Oh wait...we need a program to use this with! Let's make the absolute bare minimum (sorry CSS) program. We want it to have one display (the counter value) and three buttons (one for each action). We aren't going to use any fancy framework, we won't even create DOM nodes dynamically. We're just going to use Javascript to hook on to an existing element and innerHTML to rewrite it's value every time we get an update from the store. Like this:

```html
<div id="counter">0</div>
```

```javascript
const counter = document.getElementById('counter')

const updateCounter = () => {
  const counterValue = store.getState().counter
  counter.innerHTML = counterValue
}

store.subscribe(updateCounter)
```

We'll also need to make buttons that trigger those action creators we made earlier. Remember we need to pass the action creators to the dispatch function and call them to retrieve the action object:

```html
<button onClick="store.dispatch(incrementCounter())">Increment</button>
<button onClick="store.dispatch(decrementCounter())">Decrement</button>
<button onClick="store.dispatch(resetCounter())">Reset</button>
```

Let's put that all together (you may notice I use multiple <script></script> tags, it doesn't change anything about what we're doing, but we'll use that to represent what would normally be separate files):

```html
<!DOCTYPE html>
<html>
  <head>
    <title>A Basic counter using Redux</title>
  </head>
  <body>
    <div id="counter">0</div>
    <button onClick="store.dispatch(incrementCounter())">Increment</button>
    <button onClick="store.dispatch(decrementCounter())">Decrement</button>
    <button onClick="store.dispatch(resetCounter())">Reset</button>

    <script>
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
        let nextListeners = currentListeners 
        let isDispatching = false

        function ensureCanMutateNextListeners() {
          if(currentListeners === nextListeners) {
            nextListeners = currentListeners.slice() 
          }
        }

        function getState() { 
          if(isDispatching) {
            throw new Error('You can\'t call getState while the state is being changed')
          } 
          return currentState
        }

        function dispatch(action) {
          if(typeof action !== 'object') {
            throw new Error('Actions need to be a simple object.')
          }

          if(typeof action.type === 'undefined') {
            throw new Error("Actions need to have a 'type' key with a value that is not undefined" )
          }

          if(isDispatching) {
            throw new Error('You cannot dispatch an action from inside a reducer!')
          }

          try {
            isDispatching = true
            currentState = currentReducer(currentState, action)
          }
          finally {
            isDispatching = false
          }
          
          const listeners = currentListeners = nextListeners
          for(let i = 0; i < listeners.length; i++) {
            const listener = listeners[i]
            listener()
          }
        }

        function subscribe(listener) {
          if(typeof listener !== 'function') {
            throw new Error('Error. Expected listener to be a function')
          }

          if(isDispatching) {
            throw new Error('You may not subscribe while an action is being dispatched to the reducer')
          }

          let isSubscribed = true
          ensureCanMutateNextListeners()
          nextListeners.push(listener)

          return function unsubscribe() {
            if(!isSubscribed) {
              return
            }

            if(isDispatching) {
              throw new Error('You can\'t call unsubscribe while the store is dispatching an action')
            }

            isSubscribed = false
            ensureCanMutateNextListeners()
            const index = nextListeners.indexOf(listener)
            nextListeners.splice(index, 1)  
          }
        }

        dispatch(initialAction)

        return {
          getState,
          dispatch,
          subscribe
        }
      }    
    </script>
    <script>
      const INCREMENT = 'INCREMENT'
      const DECREMENT = 'DECREMENT'
      const RESET = 'RESET'
    </script>
    <script>
      const initialState = {
        counter: 0
      }

      const counterReducer = (state = initialState, action) => {
        switch(action.type) {
          case INCREMENT:
            return Object.assign({}, state, {counter: state.counter + 1})
          case DECREMENT:
            return Object.assign({}, state, {counter: state.counter - 1})
          case RESET:
            return Object.assign({}, state, {counter: 0})
          default:
            return state
        }
      }
    </script>
    <script>
      const incrementCounter = () => ({
        type: INCREMENT
      })
      const decrementCounter = () => ({
        type: DECREMENT
      })
      const resetCounter = () => ({
        type: RESET
      })    
    </script>
    <script>
      const store = createStore(counterReducer)
    </script>
    <script>
      const counter = document.getElementById('counter')

      const updateCounter = () => {
        const counterValue = store.getState().counter
        counter.innerHTML = counterValue
      }

      store.subscribe(updateCounter)    
    </script>
  </body>
</html>
```

Voila! If you copy the above code into a separate HTML file and run it, you should have a working program that uses our Redux redux for state management.

> An important note to make is that our small example program is simply a demonstration of how to integrate Redux into a real project. We haven't done anything that takes advantage of Redux's particular strengths. For example, we could use our browser's local storage to store a copy of the state and pass that to createStore on future visits to save changes. We could also store a copy of each change to the reducer in an array and use that to 'time-travel' through our state by adding Fast Forward and Rewind buttons.

### Chapter 24: Enhancer, ApplyMiddleware, and Compose (Higher Order Functions, Currying, and Function Composition)

If you take a look at the real Redux source code now, especially the createStore.js file, you might be happily surprised to realize it looks very, very familiar. In fact, hopefully, by this point the documentation has stopped feeling overwhelming and started feeling incredibly obvious and sensible. However, you will still run into small differences between what we did and what Redux does. On thing you may notice is that dispatch() in Redux returns the original action it was passed. We never added that feature. It's not essential, but having functions return something on execution can offer more flexibility in how the external program that calls them is structured.

You might also notice that the official Redux createStore has two more functions: replaceReducer and observable. ReplaceReducer does exactly what you might think, allowing us to swap out reducers after the store has been created. Observables are another type of programming pattern, similar to pub-sub, that some frameworks favor. If you looks closely though in Redux's case it is just a wrapper over the subscribe functionality we've already built.

The much bigger change you may notice is that the Redux createStore handles a third argument, enhancer. This is one of the most important features of Redux that we have so far completely ignored and it's the last one that we'll spend significant time with. You'll also notice a bindActionCreators and a combineReducers module which we will touch on as well -- again, at this point, much of what's left to say is well covered in the source code and this guide may be a bit redundant.


The underlying value of the enhancer argument is that it lets you pass in custom functions which extend Redux (give it new, cool features). We touched on what this constitutes very briefly in chapter 14 when we mentioned composition and currying. 

Composition is when you combine multiple functions together (effecively nesting them). Function a(), b(), and c() become a(b(c())). The result of c() is passed to b(), the result of which is passed to a(). 

Currying is when you have a process that takes multiple arguments, but take them piece by piece. Instead of Taking all the arguments at once, you take some arguments and then return another function that takes the rest and returns a result. This means you can partially fill out the arguments and wait for more input before returning the result. 

```javascript
const add = (a,b) => a + b
```

becomes:

```javascript
const curriedAdd = a => b => a + b
```

That may look a little confusing, so let's go back to the old way of writing functions:

```javascript
function add(a,b) {
  return a + b
}
```

becomes 

```javascript
function curriedAdd(a) {
  return function(b) {
    return a + b
  }
}
```

To use the traditional add function we do:

```javascript
const result = add(10, 1)
console.log(result) // 11
```

To use the curried version:

```javascript
const partiallyFilled = curriedAdd(10)
console.log(partiallyFilled) // ƒ (b) { return a + b }
const result = partiallyFilled(1)
console.log(result) // 11
```

There's one more concept concept worth mentioning, Higher Order Functions. HOFs take a function and return a new function that extends the functionality of the original function. This is of course what we said the Redux enhancer did at the beginning of this chapter. Let's make a simple example. We'll take a basic add function which returns a result. But we'll 'wrap' it in a higher order function that gives us the result in the console as well (the wrapper is a higher order function):

```javascript
function add(a,b) {
  const result = a + b
  return result
}

function wrapper(func) {
  return function(a, b) {
    const result = func(a,b)
    console.log(result)
    return result
  }
}
```

To use that we do:

```javascript
const wrappedAdd = wrapper(add)
```

And now we can use wrappedAdd to both get the results of the original function as originally intended as well as log it to the console at the same time.

You may notice our wrapper function is built to expect the function it is wrapping to have two arguments, 'a' and 'b'. This makes it pretty inflexible. Instead we can capture all of the arguments (no matter how many there are) and pass them to our wrapped function without ever having to know how many or what they are. There are a few ways to do this in Javascript. Typically you either see people use the 'arguments' keyword which is a secret feature of any function. It is like an array (but not quite) that holds all the arguments passed to a function, you don't have to make it yourself. 

The other way that is more common these days is to use a spread operator. We saw this much earlier as one way of cloning arrays and objects. Here it will first gather all the arguments a function gets into an array, and then when we use it again it will spread those back out, just like we found them, in the new array we return:

```javascript
function wrapper(func) {
  return function(...args) {
    const result = func(...args)
    console.log(result)
    return result
  }
}
```

And now we have a generic higher order function that can take any function that returns something and log it to the console. Pretty cool, right!

That's what the enhancer argument is. It's an unknown higher order function. We give it our createStore function. It does something to make it even more awesome and then passes it back. We immediately call the new and improved createStore function with the original reducer and state arguments it was first called with. That's what this line means:

```javascript
return enhancer(createStore)(reducer, preloadedState)
```

The problem with this is that it only let's use use one higher order function. That's not so awesome. There are all kinds of cool enhancements we could add in the middle of receiving the first arguments and returning the final result. Like so originalFunction -> enhancer -> enhancer -> enhancer -> finalModifiedFunction. Those middle enhancers are called 'middleware' in programming. Functions or programs that take an input and pass on the modified result to the next function in a chain. 

Because createStore can only accept one enhancer, in order to have multiple higher order functions we need to somehow combine them into one. Remember function composition? That's what applyMiddleware is. It's a higher order function that takes any number of other HOFs and returns a new higher order function that takes createStore as an argument. That new function is the enhancer function we already looked at above. The new enhancer function is run and calls createStore to create a store as normal. However, instead of just giving it back to us like usual, first it creates a new version of the dispatch function (this is the only function that Redux allows middleware to be applied to), that new version of dispatch is then wrapped with all the middleware functions, one after the other (this is composition and it's what the Redux 'comopose' function is for). Finally, the original store object (with getState, subscribe, etc.) is returned with the new modified dispatch function.

Middleware is powerful and let's us add all kinds of new functionality to our store. There are all kinds of enhancements we could use but by far the most common type is something to handle asynchronous behavior. Let's take a look.

## Appendices

IMPORTANT! From here on out we will be using the real Redux library and it's version of createStore. The original scope of the tutorial didn't cover reimplementing a few pieces of functionality that we will need for exploring middleware, namely applyMiddleware. Because our version of Redux mirrored the original so closely, we don't have to change the way we interact with the store or the structure of our reducers, action creators etc. 

Also good to note is that in practice, Redux is almost always used in programs that use modules (when the program is broken up into multiple files that are then recombined using require/import statements) and so you would typically need to do something like this at the top of your program: 

```javascript
import { createStore, combineReducers, applyMiddware } from 'redux';
```

If you want to keep using the single file we've been building in without imports, you can use a CDN (a website that keeps files online for us to access without having to keep our own copy). At the time of writing, this works https://cdnjs.cloudflare.com/ajax/libs/redux/4.0.0/redux.js. Now we will have access to redux methods in our program on the global scope (we can access it anywhere) using the Redux namespace (the methods are all in an object called Redux). Like so:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/redux/4.0.0/redux.js"></script>
<script>
  const store = Redux.createStore(/* our reducer goes here same as before */)
</script>
```

We can also make it easier to access the methods by doing this by destructuring the methods to the global scope first. And then our code will look like it would in a program using modules with import statements:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/redux/4.0.0/redux.js"></script>
<script>
  const { createStore, applyMiddleware, combineReducers } = Redux;
  const store = createStore(/* our reducer goes here same as before */)
</script>
```

For the rest of appendices, I will assume the above process has been done so that we can use the more common shorthand.

### Appendix 1 - Middleware

The modern Javascript world is full of choices. If you got this far in my guide then you've probably already made a few of your own. You chose a DIY approach over a CMS like Wordpress, React over Angular, and Redux over MobX. You would think that would be enough but no, now you are faced with the most impenetrable choice yet, Thunks or Sagas? I mean, really, what on Earth is a thunk? What is a saga? When will the choices stop!

In short, Thunks and Sagas (and Redux-Observable if you've seen that one too) are different approaches to handling complicated processes in a React/Redux app, often asynchronous, that take advantage of the ability to extend the dispatch function with applyMiddleware. 

> We won't talk about Sagas here since that is a fully featured library (using more new paradigms like generator functions) and we are focused on understanding the core Redux functionality. 

Despite the bizarre name and the fact that thunks are another library you have to install, they are in fact one of the easiest parts of Redux to understand. But before we dive into thunks, let's get a better understanding of what Redux middleware looks like and how it works.

### Appendix 1.1: The basic structure of middleware 

We already know that dispatch takes an action and passes it to a reducer before capturing the result. We also saw that middleware are functions that take that action, possibly do something with it and then pass it on (forming a chain between the initial call of the modified dispatch between and the final process of calling the reducer with the action (the original dispatch method)). Because of the way dispatch is constructed, middleware needs to have a certain common structure. It's going to look complicated (it uses a lot of currying) at first but fear not! The best part is that the pattern never really changes, anytime you want to make a new middleware, you can simply copy the skeleton over. So what does it look like?

```javascript
const middleware = store => next => action => {
  // Your code here
}
```

Which is the same as:

```javascript
function middleware(store){
  return function(next){
    return function(action){
      // Your code here
    }
  }
}
```

You should also note that store represents an object with access to the getState and dispatch methods of our store so that we can use them in our middleware (that's what makes middleware so powerful), and you'll often see it destructured in the arguments directly (one or both methods), like so:

```javascript
const middleware = ({ dispatch, getState }) => next => action => {
  // Your code here
}
```

Sometimes you don't need either (like a logging middleware) and you can just use:

```javascript
const middleware = () => next => action => {
  // Your code here
}
```

That's it! Well, sort of. The middleware above won't be very useful. In fact, it will effectively break our store. You'll remember I said the middleware acts like a chain of sorts to pass the action along (something like this: modifiedDispatch -> middleware1 -> middleware2 -> originalDispatch -> reducer). Well, you'll notice we haven't passed anything along, so any action that gets passed to our middleware will get stuck and never make it to our reducer to update the state. How do we fix that? Well, in this pattern 'next' represents the unknown function that represents the next link in the chain (either another middleware or, eventually, the final link in the chain, the original dispatch method that calls the reducer with the action). So in order to pass the action to the next function, we simple pass it to next. Like so:

```javascript
const middleware = () => next => action => {
  next(action);
}
```

That is a fully functioning piece of middleware. Well, except it doesn't do anything. So let's add a tiny bit of functionality, we'll log the action to the console so we can see the actions being called under the hood while our program runs.

```javascript
const loggerMiddlware = () => next => action => {
  console.log(action);
  next(action);
}
```

### Appendix 1.2: Applying middleware

Before we go any deeper, let's quickly look at how to add this middleware to your Redux store. Remember, 'createStore' and 'applyMiddleware' come from Redux.

```javascript
// This is mock reducer (remember the pattern, a reducer is a function that takes current state and an action and returns new state)
const uselessReducer = (state = {}, action) => state;
const uselessMiddleware = store => next => action => { next(action) };
const store = createStore(uselessReducer, applyMiddleware(uselessMiddleware));
```

That's all there is to it. You can pass more middleware as more arguments to applyMiddleware. I.e. applyMiddleware(middleware1, middleware2, middleware3). Middlewares will be executed in the order in which they are passed. I.e. applyMiddleware(middleware1, middleware2, middleware3) will lead to a dispatch that looks like enhancedDispatch -> middleware1 -> middleware2 -> middleware3 -> originalDispatch -> reducer.

Since we didn't fully replicate this functionality in our version of createStore, I want to briefly mention whats going on under the hood here. The real createStore takes up to three arguments: a single reducer, preloadedState, and a single enhancer. If you have multiple reducers (and you will), you use 'combineReducers' to glue them all together before passing them as the first parameter. If you have previously saved the state of the store and want to restore it on load (I use this for things like saving the state to sessionStorage so the app doesn't restart from scratch when someone refreshes the page). If you want to add middleware you need to combine (compose) them with applyMiddleware first. Redux uses a little trick to let you pass in an enhancer even without previous state so when you pass only two arguments the second argument can be either state or an enhancer (createStore checks if the second argument is a function or not to decide what to treat it as). Like so:

```javascript
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState;
    preloadedState = undefined;
  }
```

If there is an enhancer passed (and it's a function), instead of running the rest of the createStore function, instead the function will be passed to the enhancer along with the other arguments (the reducer and the preloadedState). Like so:

```javascript
  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.');
    }

    return enhancer(createStore)(reducer, preloadedState);
  }
```

This gives the enhancer the chance to modify (enhance) createStore (really only the dispatch method if you recall from earlier), after which it will run the enhanced version with the other original arguments.

> It is possible to pass other enhancers that do more than just add middleware to the dispatch chain, but we will consider that out of scope for now. This requires using a compose function we haven't covered (but applyMiddleware uses under the hood), and following a certain pattern (applyMiddleware needs to be passed first).

### Appendix 1.3: Thunks

We've finally made it. All that build up and now we can finally discuss the widely feared THUNKS of lore. Be prepared for a bit of a surprise. 

First of all, what does thunk mean? Is it just the sound of your head hitting the table in frustration? Possibly. But it does have another more useful meaning (it has a few but we'll only focus on the relevant one). In essence a thunk is a function that wraps another function in order to delay it's execution until a later time. Let's say we had a function that returned some words:

```javascript
const getHello = () => 'Hello'
```

And another function to 'say' those words (this is a higher order function (a function that wraps another function)):

```javascript
const talk = thingToSay => { console.log(thingToSay()) }
```

So, we can do this:

```javascript
talk(getHello) // 'Hello'
```

By passing the word function into the talk function, we are able to say something. Now, let's make it more complicated. if instead of talk directly calling console.log, what if it instead used another method to do that. (We might as well change thingToSay to getWords, since that's what it's really doing anyway.)

```javascript
const talk = getWords => {
  const sayWords = words => { console.log(words) }

  const words = getWords();
  sayWords(words);
}

// NOTE: The way we call it hasn't changed
talk(getHello) // 'Hello'
```

 Right now 'talk' expects the 'getWords' function to return a string immediately (and it doesn't expect a promise so we can't use that trick) which it then immediately passes to 'sayWords' to log to the console. But what if the 'getHello' function needed some extra time to get the words it needs (like from a database) instead of returning them immediately? There are different approaches to handling this issue, but the one we will look at is passing 'sayWords' to 'getHello' so that 'getHello' can use decide when to call 'sayWords' instead of 'talk'. 

 ```javascript
 const getHello = sayWords => {
   sayWords('Hello');
 }
 ```

 Now, if there's a delay in retrieving the word 'hello', there isn't a problem because 'getHello' has access to 'sayWords' and can call it when it's ready.

 ```javascript
 const getHello = sayWords => {
   fetch('http://www.placewithwords.com')
    .then(response => {
      sayWords(response)
    })
 }
 ```

Now, if you've been paying attention, you might be wondering why 'talk' couldn't just expect a delay if it knew 'getWords' was an async function. Something like:

```javascript
const talk = getWords => {
  const sayWords = words => { console.log(words) }
  
  getWords()
    .then(response => sayWords(response))
}
```

That works, as long as getWords returns a promise. Something like:

```javascript
const getHello = () => fetch('http://www.placewithwords.com')
```

But now we have two functions that are tightly coupled (very interdependent). There isn't a lot of flexibility for different kinds of getWords functions. What if a new 'getName' function didn't use fetch but instead looked up a file on the local directory? How would 'talk' know how to handle the results? It wouldn't, it's expecting a certain type of response to then pass to 'sayWords'. It could try and handle different types of responses, but this would be difficult to say the least. So, instead of having 'talk' be responsible for deciding how to handle different kinds of 'getWords' functions before calling 'sayWords', we just have 'talk' pass 'sayWords' to them and let them handle it. Now 'talk' becomes an agnostic way to provide functions with a method to 'sayWords'. As long as the functions expect to be passed 'sayWords' as an argument, everything works.

```javascript
const getHello = sayWords => {
  fetch('http://www.placewithwords.com')
    .then(response => {
      sayWords(response)
    })
}

const talk = getWords => {
  const sayWords = words => { console.log(words) };
  getWords(sayWords)
}

talk(getHello); // 'Hello'
```

So, now 'getHello' is a thunk. It takes a function and then delays calling it until it's ready. And 'talk' is like a thunk helper, it passes the functions that the thunk needs to it. Nothing else. We could even do a safety check for the hell of it. If 'talk' receives something that isn't a function it will throw an error, so we can do a simple check.

```javascript
const talk = getWords => {
  const sayWords = words => { console.log(words) };
  if(typeof getWords === 'function'){
    getWords(sayWords);
  }
}
```

Now it only tries to run the thunk when it's sure it's a function. Cool.

So, how does this longwinded explanation of a pretty simple process apply to redux-thunk? Let's make a new middleware to see. Like before, we use the same scaffolding:

```javascript
const middleware = ({ dispatch, getState }) => next => action => {
  next(action);
}
```

We want this new middleware to do something a little different. Instead of logging the action object to the console or something, instead we want to see if the action isn't actually an action, but instead a thunk. How do we do that? Well, we can check to see if the action is a thunk, just like before. 

```javascript
const middleware = ({ dispatch, getState }) => next => action => {
  if(typeof action === 'function'){
    // Do something
    return;
  }
  next(action);
}
```
Why do we return? It's a guard clause like we saw way back in chapter 5, because we don't want to pass the thunk action to another middleware in the chain. At this point, the thunk is now in control of it's own destiny (and no other middleware will know how to handle a thunk instead of an action object that they are built to expect). If the action is not a thunk, then we simply ignore it and pass it to the next middleware in the chain. So, what can we put in place of 'Do something'? Like we saw in our 'talk' example above, we want this middleware to be agnostic. It's only job is to provide the thunk with control of it's own destiny. In the case of an action, control means having direct access to dispatch (and getState). So:

```javascript
const middleware = ({ dispatch, getState }) => next => action => {
  if(typeof action === 'function'){
    return action(dispatch, getState);
    // We can combine the thunk call and return statement into one line
  }
  return next(action);
}
```

That's it. No joke. That is redux-thunk. You've seen dozens of tutorials explaining it, and there it is, in all it's naked glory. If an action is an object, call the action with dispatch and getstate, otherwise ignore the action. 

Now, I would never lie to you, if you look at the source code for redux-thunk, you'll see a few more lines of code (and only a few). All they are is a wrapper that allows you to pass an extra argument to your thunk action creators in addition to getState and dispatch. So, what does a thunk look like in redux? You might be happily surprised to see that it looks almost exactly like 'getWords'.

```javascript
const exampleThunkToFetchData = () => (dispatch, getState) => {
  fetch('http://www.placewithdata.com')
    .then(response => {
      dispatch({
        type: 'SAVE',
        payload: response
      });
    })
}
```
Or, using an action creator:

```javascript
const save = data => ({
  type: 'SAVE',
  payload: data
})

const exampleThunkToFetchData = () => (dispatch, getState) => {
  fetch('http://www.placewithdata.com')
    .then(response => {
      dispatch(save(response));
    })
}
```

Why is our thunk returning a function that take dispatch and getState instead of taking them directly like 'getWords' did? That's because the first time this function is called is when it is called by dispatch somewhere in our program. At that point the inner function that takes dispatch and getState travels down the middleware chain until our thunk middleware passes in those arguments while calling it again. This means the first time it's called it can take other arguments from the program (like user input, a url etc.) Like so:

```javascript
const thunkMiddleware = ({ dispatch, getState }) => next => action => {
  if(typeof action === 'function'){
    return action(dispatch, getState);
  }
  next(action);
}

const reducer = (state = {}, action) => state;

// Action Creator
const saveData = data => ({
  type: 'SAVE',
  payload: data
})

// Thunk
const getData = url => (dispatch, getState) => {
  fetch(url)
    .then(response => {
      dispatch(saveData(response));
    })
}

const store = createStore(reducer, applyMiddleware(thunkMiddleware))

// This represents our main program.
const main = () => {
  // Do Stuff Here (like a React component)
  const url = Math.random() > .5 ? 'http://www.cutekittens.com' : 'http://www.evilgremlins.com';
  store.dispatch(getData(url))
}
```

And that's how thunks work. Like I mentioned, redux-thunk does provide for the ability to take an extra argument (something that you might want your thunk to have access to that wouldn't be available otherwise), but we won't explore this since it should be pretty clear by now after all the practice currying we've done.

### Appendix 1.4: The power of dispatch

We've already talked about how applyMiddleware creates a chain that actions flow through on their way to a reducer. While we didn't cover how applyMiddleware works in detail, there is an important part of how it affects middleware is worth mentioning. In your middleware, if you call next(action), the action will be passed down the chain (and in one case next is the original dispatch function itself), but if you call dispatch(action) in a middleware, the action will be passed back to the start of the chain and begin the journey to the reducer all over again. This is immensely useful! It means we no longer have to care about sequence as much. One middleware can pass an action to another whether or not it's already been passed on the chain. A middleware can even pass an action to itself again. In fact, that last piece is very important for redux-thunk. It means that thunks can call thunks. If you dispatch a thunk from a thunk, the new thunk will hit the thunkMiddleware again. If you dispatch an action, it will be ignored by thunkMiddleware and finally make it's way to the reducer to update the state. This helps up break up a thunk into smaller pieces of logic (which is good for reuse and for testing, and for just being able to understand later).

### Appendix 1.5: What to do with thunks

I just want to provide a little food for thought here. There are plenty of tutorials exploring the power of thunks, but I at least wanted to start the ball rolling. The power (and peril) of thunks is that they give us our own private garden in which to run all kinds of code that does all kinds of things before finally updating the state. In most React apps that means almost all the business logic can go in thunks and the components can concern themselves with just worrying about rendering and handling events. Once you start handling your async logic in action creators, your React components will feel massively cleaner!



///////// IN PROGRESS NOTES

-Talk about how reducers are actually not separate.
-We can make the second phase about how it works with react (Provider/connect)