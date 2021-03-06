# transitions

A lightweight, object-oriented state machine implementation in Python.

## Installation

    pip install transitions

...or clone the repo from GitHub and then:

    python setup.py install

## Quickstart

They say [a good example is worth](https://www.google.com/webhp?sourceid=chrome-instant&ion=1&espv=2&ie=UTF-8#q=%22a+good+example+is+worth%22&start=20) 100 pages of API documentation, a million directives, or a thousand words. Well, "they" probably lie... but here's an example anyway:

```python
from transitions import Machine
import random

class NarcolepticSuperhero(object):

    # Define some states. Most of the time, narcoleptic superheroes are just like
    # everyone else. Except for...
    states = ['asleep', 'hanging out', 'hungry', 'sweaty', 'saving the world']

    def __init__(self, name):
    
        # No anonymous superheroes on my watch! Every narcoleptic superhero gets 
        # a name. Any name at all. SleepyMan. SlumberGirl. You get the idea.
        self.name = name

        # What have we accomplished today?
        self.kittens_rescued = 0
        
        # Initialize the state machine
        self.machine = Machine(model=self, states=NarcolepticSuperhero.states, initial='asleep')
        
        # add some transitions. We could also define these using a static list of 
        # dictionaries, as we did with states above, and then pass the list to 
        # the Machine initializer as the transitions= argument.
        
        # At some point, every superhero must rise and shine.
        self.machine.add_transition('wake_up', 'asleep', 'hanging out')
        
        # Superheroes need to keep in shape.
        self.machine.add_transition(trigger='work_out', source='hanging out', dest='hungry')
        
        # Those calories won't replenish themselves!
        self.machine.add_transition('eat', 'hungry', 'hanging out')

        # Superheroes are always on call. ALWAYS. But they're not always 
        # dressed in work-appropriate clothing.
        self.machine.add_transition('distress_call', '*', 'saving the world', 
                         before='change_into_super_secret_costume')
        
        # When they get off work, they're all sweaty and disgusting. But before
        # they do anything else, they have to meticulously log their latest 
        # escapades. Because the legal department says so.
        self.machine.add_transition('complete_mission', 'saving the world', 'sweaty', 
                         after='update_journal')
        
        # Sweat is a disorder that can be remedied with water.
        # Unless you've had a particularly long day, in which case... bed time!
        self.machine.add_transition('clean_up', 'sweaty', 'asleep', conditions=['is_exhausted'])
        self.machine.add_transition('clean_up', 'sweaty', 'hanging out')
        
        # Our NarcolepticSuperhero can fall asleep at pretty much any time.
        self.machine.add_transition('nap', '*', 'asleep')

    def update_journal(self):
        """ Dear Diary, today I saved Mr. Whiskers. Again. """
        self.kittens_rescued += 1
        
    def is_exhausted(self):
        """ Basically a coin toss. """
        return random.random() < 0.5

    def change_into_super_secret_costume(self):
        print "Beauty, eh?"
```

There, now we've baked a state machine into our NarcolepticSuperhero. Let's take him/her/it out for a spin...

```python
>>> batman = NarcolepticSuperhero("Batman")
>>> batman.state
'asleep'

>>> batman.wake_up()
>>> batman.state
'hanging out'

>>> batman.nap()
>>> batman.state
'asleep'

>>> batman.clean_up()
MachineError: "Can't trigger event clean_up from state asleep!"

>>> batman.wake_up()
>>> batman.work_out()
>>> batman.state
'hungry'

# Batman still hasn't done anything useful...
>>> batman.kittens_rescued
0

# We now take you live to the scene of a horrific kitten entreement...
>>> batman.distress_call()
'Beauty, eh?'
>>> batman.state
'saving the world'

# Back to the crib. 
>>> batman.complete_mission()
>>> batman.state
'sweaty'

>>> batman.clean_up()
>>> batman.state
'asleep'   # Too tired to shower!

# Another productive day, Alfred.
>>> batman.kittens_rescued
1
```

## The non-quickstart

### Basic initialization

Getting our state machine up and running is pretty simple. Let's say we have an object called _lump_--an instance of class Matter--whose states we want to manage:

```python
class Matter(object):
    pass
    
lump = Matter()
```

We can initialize a minimal working state machine bound to _lump_ like so:

```python
from transitions import Machine
machine = Machine(model=lump, states=['solid', 'liquid', 'gas', 'plasma'], initial='solid')

# Lump now has state!
lump.state
>>> 'solid'
```

I say "minimal", because while this state machine is technically operational, it doesn't actually _do_ anything. It'll start its life in the 'solid' state, but won't ever move into any other state, because we haven't defined any transitions yet. So let's try again.

```python
# The states
states=['solid', 'liquid', 'gas', 'plasma']

# And some transitions between states. We're lazy, so we'll leave out 
# the inverse phase transitions (freezing, condensation, etc.).
transitions = [
    { 'trigger': 'melt', 'source': 'solid', 'dest': 'liquid' },
    { 'trigger': 'evaporate', 'source': 'liquid', 'dest': 'gas' },
    { 'trigger': 'sublimate', 'source': 'solid', 'dest': 'gas' },
    { 'trigger': 'ionize', 'source': 'gas', 'dest': 'plasma' }
]

# Initialize
machine = Machine(lump, states=states, transitions=transitions, initial='liquid')

# Now lump maintains state...
lump.state
>>> 'liquid'

# And that state can change...
lump.evaporate()
lump.state
>>> 'gas'
lump.ionize()
lump.state
>>> 'plasma'
```

Notice the shiny new methods attached to our Matter instance (evaporate(), ionize(), etc.), each of which triggers the corresponding transition. We don't have to explicitly define these methods anywhere; the name of each transition is bound to the model we passed to the Machine initializer (in this case, _lump_).

### States

The soul of any good state machine (and of many bad ones, no doubt) is a set of states. Above, we defined the valid model states by giving the Machine initializer a list of strings. But internally, states are actually represented as State objects. You can initialize and modify States in a number of ways. Specifically, you can (a) pass a string to the Machine initializer giving the name(s) of the state(s); (b) directly initialize each new State object; or (c) pass a dictionary that contains initialization arguments. The following snippets illustrate several ways to achieve the same goal:

```python
# Create a list of 3 states to pass to the Machine
# initializer. We can mix types; in this case, we 
# pass one State, one string, and one dict.
states = [
    State(name='StateA'),
    'StateB',
    { 'name': 'StateC'}
    ]
machine = Machine(lump, states)

# This alternative example illustrates more explicit 
# addition of states and state callbacks, but the net
# result is identical to the above.
machine = Machine(lump)
state_a = State('StateA')
state_b = State('StateB')
state_c = State('StateC')
machine.add_states([state_a, state_b, state_c])

```

#### Callbacks
A State can also be associated with a list of _enter_ and _exit_ callbacks, which will be triggered whenever the state machine enters or leaves that state. Callbacks can be specified during initialization or added later. For convenience, whenever a new State is added to a Machine, dynamic on\_enter\_{state name} and on\_exit\_{state name} methods are created.

```python
# Our old Matter class, now with  a couple of new methods we 
# can trigger when entering or exit states.
class Matter(object):
    def say_hello(self): print "hello, new state!"
    def say_goodbye(self): print "goodbye, old state!"
    
lump = Matter()

# Same states as above, but now we give StateA an exit callback
states = [
    State(name='StateA', on_exit=['say_goodbye']),
    'StateB',
    { 'name': 'StateC' }
    ]
    
machine = Machine(lump, states=states)
machine.add_transition('sublimate', 'solid', 'gas')
    
# Callbacks can also be added after initialization using 
# the dynamically added on_enter_ and on_exist_ methods.
lump.on_enter_StateC('say_hello')

# Test out the callbacks...
lump.set_state('solid')
lump.sublimate()
>>> 'goodbye, old state!'
>>> 'hello, new state!'
```

#### Checking state
We can always check the current state of the model by inspecting the .state attribute. We can also check whether the model is in a specific state using is\_{state name}(). Lastly, if we want to retrieve the actual State object for the current state, we can do that through the Machine instance's get\_state() method.

```python
lump.state
>>> 'solid'
lump.is_gas()
>>> False
lump.is_solid()
>>> True
machine.get_state().name
>>> 'solid'
```

### Transitions
Some of the above examples already illustrate the use of transitions in passing, but here we'll explore them in more detail. As with states, each transition is represented internally as its own object--an instance of class Transition. The quickest way to initialize a set of transitions is to pass a dictionary, or list of dictionaries, to the Machine initializer. We already saw this above:

```python
transitions = [
    { 'trigger': 'melt', 'source': 'solid', 'dest': 'liquid' },
    { 'trigger': 'evaporate', 'source': 'liquid', 'dest': 'gas' },
    { 'trigger': 'sublimate', 'source': 'solid', 'dest': 'gas' },
    { 'trigger': 'ionize', 'source': 'gas', 'dest': 'plasma' }
]
machine = Machine(model=Matter(), states=states, transitions=transitions)
```

Alternatively, we can add transitions to a Machine after initialization:

```python
machine = Machine(model=lump, states=states, initial='solid')
machine.add_transition('melt', source='solid', dest='liquid')
```

The trigger argument defines the name of the new triggering method that gets attached to the base model. When this method is called, it will try to execute the transition:

```python
lump.melt()
lump.state
>>> 'liquid'
```

#### Transitioning from multiple states
A given trigger can be attached to multiple transitions, some of which can potentially begin or end in the same state. For example:

```python
machine.add_transition('transmogrify', ['solid','liquid', 'gas'], 'plasma')
machine.add_transition('transmogrify', 'plasma', 'solid')
# This next transition will never execute
machine.add_transition('transmogrify', 'plasma', 'gas')
```

In this case, calling transmogrify() will set the model's state to solid if it's currently plasma, and set it to plasma otherwise. Note that only the first matching transition will execute--hence, the transition defined in the last line above won't do anything.

We can also indicate that a trigger should cause a transition from _all_ states to a particular destination by using the wildcard:

```python
machine.add_transition('to_liquid', '*', 'liquid')
```

#### Conditional transitions
Sometimes we only want a particular transition to execute if some specific condition obtains. We can ensure that happens by passing a method or list of methods in the conditions argument:

```python
# Our Matter class, now with a bunch of methods that return booleans.
class Matter(object):
    def is_flammable(self): return False
    def is_really_hot(self): return True

machine.add_transition('heat', 'solid', 'gas', conditions='is_flammable')
machine.add_transition('heat', 'solid', 'liquid', conditions=['is_really_hot'])
```

In this somewhat contrived example, calling heat() when the model is in state 'solid' will transition to state 'gas' if is\_flammable returns True. Otherwise, it will transition to state 'liquid' if is\_really\_hot returns True.

#### Callbacks
As with states, we can attach callbacks to transitions. Every transition has 'before' and 'after' attributes that contain a list of methods to call before and after the transition executes:

```python
class Matter(object):
    def make_hissing_noises(self): print "HISSSSSSSSSSSSSSSS"
    def disappear: print "where'd all the liquid go?"

transitions = [
    { 'trigger': 'melt', 'source': 'solid', 'dest': 'liquid', 'before': 'make_hissing_noises'},
    { 'trigger': 'evaporate', 'source': 'liquid', 'dest': 'gas', 'after': 'disappear' }
]

lump = Matter()
machine = Machine(lump, states, transitions=transitions, initial='solid')
lump.melt()
>>> "HISSSSSSSSSSSSSSSS"
lump.evaporate()
>>> "where'd all the liquid go?"
```

### Passing data
Sometimes you need to pass the callback functions registered at machine initialization some data that reflects the model's current state. Transitions allows you to do this in two different ways. First (and by default), you can pass any positional or keyword arguments you like directly to the trigger methods added when transitions were registered:

```python
class Matter(object):
    def __init__(self): self.set_environment()
    def set_environment(self, temp=0, pressure=101.325):
        self.temp = temp
        self.pressure = pressure
    def print_temperature(self): print("Current temperature is %d degrees celsius." % self.temp)
    def print_pressure(self): print("Current pressure is %.2f kPa." % self.pressure)

lump = Matter()
machine = Machine(lump, ['solid', 'liquid'], initial='solid')
machine.add_transition('melt', 'solid', 'liquid', before='set_environment')

lump.melt(45)  # positional arg
lump.print_temperature()
> 'Current temperature is 45 degrees celsius.'

machine.set_state('solid')  # reset state so we can melt again
lump.melt(pressure=300.23)  # keyword args also work
lump.print_pressure()
> 'Current pressure is 300.23 kPa.'

```

You can pass any number of arguments you like to the trigger.

There is one important limitation to this approach: every callback function triggered by the state transition must be able to handle _all_ of the arguments. This may cause problems if you have multiple callbacks that each expects somewhat different data. To get around this, Transitions supports an alternate method for sending data. If you set send_events=True at Machine initialization, all arguments to the triggers will be wrapped in an EventData instance and passed on to every callback. (The EventData object also maintains internal references to the source state, model, machine, and trigger associated with the transition, in case you need to access these for anything.)

```python
class Matter(object):

    def __init__(self):
        self.temp = 0
        self.pressure = 101.325

    # Note that sole argument is now the EventData instance
    def set_environment(self, event):
        self.temp = event.kwargs.get('temp', 0)
        self.pressure = event.kwargs.get('pressure', 101.325)

    def print_pressure(self): print("Current pressure is %.2f kPa." % self.pressure)

lump = Matter()
machine = Machine(lump, ['solid', 'liquid'], send_event=True, initial='solid')
machine.add_transition('melt', 'solid', 'liquid', before='set_environment')

lump.melt(temp=45, pressure=1853.68)  # positional arg
lump.print_pressure()
> 'Current pressure is 1853.68 kPa.'

```


### Alternative initialization patterns

In all of the examples so far, we've attached a new Machine instance to a separate model--specifically, to _lump_ (an instance of class Matter). While this separation keeps things tidy--because we don't have to monkey patch a whole bunch of new methods into our Matter class--it can also get annoying, since it requires us to keep track of which methods get called on our state machine, and which ones get called on the model the state machine is bound to (e.g., lump.on_enter_StateA() vs. machine.add_transition()). Fortunately, Transitions is flexible, and supports two other initialization patterns. First, we can create a standalone state machine that doesn't require another model at all. All we have to do is omit the model argument during initialization:

```python
machine = Machine(state=states, transition=transitions, initial='solid')
machine.melt()
machine.state
>>> 'liquid'
```

If we initialize our machine this way, we will then attach all triggering events (like evaporate(), sublimate(), etc.) and all callback functions directly to the Machine instance. This approach has the benefit of consolidating all of our state machine functionality in one place, but can feel a little bit unnatural if you think state logic should be contained within the model itself rather than in a separate controller.

An alternative and potentially better approach is to have the model inherit from the Machine class. Transitions is designed to support inheritance seamlessly. Just be sure to override class Machine's init method:

```python
class Matter(Machine):
    def say_hello(self): print "hello, new state!"
    def say_goodbye(self): print "goodbye, old state!"

    def __init__(self):
        states = ['solid', 'liquid', 'gas']
        Machine.__init__(self, states, initial='solid')
        self.add_transition('melt', 'solid', 'liquid')
    
lump = Matter()
lump.state
>>> 'solid'
lump.melt()
lump.state
>>> 'liquid'
```

Here we get to consolidate all state machine functionality in our existing model, which is often a more natural way to think about things than sticking all of the functionality we want in a separate standalone Machine instance.

## I have a [bug report/issue/question]...
For bug reports and other issues, please open an issue on GitHub. For any other questions, solicitations, or large unrestricted monetary gifts, email [Tal Yarkoni](mailto:tyarkoni@gmail.com).

