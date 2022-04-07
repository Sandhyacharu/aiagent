# Developing AI Agent with PEAS Description

## AIM

To find the PEAS description for the given AI problem and develop an AI agent.

## THEORY
A vacuum-cleaner world with just two locations.

Each location can be clean or dirty.

The agent can move left or right and can clean the square that it occupies.


## PEAS DESCRIPTION
| Agent Type  | Performance Measure  | Environment | Actuators | Sensors|
| :-------------: |:-------------: |:-------------: |:-------------: |:-------------: |
| Vaccum-Cleaner  | Cleanliness,Number of Movements  | Rooms | Wheels,suction tool | location,Cleanliness |

## DESIGN STEPS
### STEP 1:
Identifying the input:

### STEP 2:
Identifying the output:

### STEP 3:
Developing the PEAS description:
### STEP 4:
Implementing the AI agent

### STEP 5:
Measure the performance parameters

## PROGRAM
```Python3
import random
import time

class Thing:
    """
        This represents any physical object that can appear in an Environment.
    """

    def is_alive(self):
        """Things that are 'alive' should return true."""
        return hasattr(self, 'alive') and self.alive

    def show_state(self):
        """Display the agent's internal state. Subclasses should override."""
        print("I don't know how to show_state.")


class Agent(Thing):
    """
        An Agent is a subclass of Thing
    """

    def __init__(self, program=None):
        self.alive = True
        self.performance = 0
        self.program = program

    def can_grab(self, thing):
        """Return True if this agent can grab this thing.
        Override for appropriate subclasses of Agent and Thing."""
        return False

def TableDrivenAgentProgram(table):
    """
    [Figure 2.7]
    This agent selects an action based on the percept sequence.
    It is practical only for tiny domains.
    To customize it, provide as table a dictionary of all
    {percept_sequence:action} pairs.
    """
    percepts = []

    def program(percept):
        action =None
        percepts.append(percept)
        action=table.get(tuple(percept))
        return action

    return program

loc_A, loc_B,loc_C, loc_D, loc_E, loc_F, loc_G, loc_H, loc_I= (-1,-1),(0,-1),(1,-1),(1, 0) ,(0, 0),(-1,0),(-1,1),(0,1),(1,1)  # The two locations for the Vacuum world


def TableDrivenVacuumAgent():
    """
    Tabular approach towards vacuum world
    """
    table = {(loc_A, 'Clean'): 'Right',
             (loc_A, 'Dirty'): 'Suck',
             (loc_B, 'Clean'): 'Right',
             (loc_B, 'Dirty'): 'Suck',
             (loc_C, 'Clean'): 'Up',
             (loc_C, 'Dirty'): 'Suck',
             (loc_D, 'Clean'): 'Left',
             (loc_D, 'Dirty'): 'Suck',
             (loc_E, 'Clean'): 'Left',
             (loc_E, 'Dirty'): 'Suck',
             (loc_F, 'Clean'): 'Up',
             (loc_F, 'Dirty'): 'Suck',
             (loc_G, 'Clean'): 'Right',
             (loc_G, 'Dirty'): 'Suck',
             (loc_H, 'Clean'): 'Right',
             (loc_H, 'Dirty'): 'Suck',
             (loc_I, 'Clean'): 'Down',
             (loc_I, 'Dirty'): 'Suck',
            }
    return Agent(TableDrivenAgentProgram(table))


class Environment:
    """Abstract class representing an Environment. 'Real' Environment classes
    inherit from this. Your Environment will typically need to implement:
        percept:           Define the percept that an agent sees.
        execute_action:    Define the effects of executing an action.
                           Also update the agent.performance slot.
    The environment keeps a list of .things and .agents (which is a subset
    of .things). Each agent has a .performance slot, initialized to 0.
    Each thing has a .location slot, even though some environments may not
    need this."""

    def __init__(self):
        self.things = []
        self.agents = []

    def percept(self, agent):
        """Return the percept that the agent sees at this point. (Implement this.)"""
        raise NotImplementedError

    def execute_action(self, agent, action):
        """Change the world to reflect this action. (Implement this.)"""
        raise NotImplementedError

    def default_location(self, thing):
        """Default location to place a new thing with unspecified location."""
        return None

    def is_done(self):
        """By default, we're done when we can't find a live agent."""
        return not any(agent.is_alive() for agent in self.agents)

    def step(self):
        """Run the environment for one time step. If the
        actions and exogenous changes are independent, this method will
        do. If there are interactions between them, you'll need to
        override this method."""
        if not self.is_done():
            actions = []
            for agent in self.agents:
                if agent.alive:
                    actions.append(agent.program(self.percept(agent)))
                else:
                    actions.append("")
            for (agent, action) in zip(self.agents, actions):
                self.execute_action(agent, action)

    def run(self, steps=1000):
        """Run the Environment for given number of time steps."""
        for step in range(steps):
            if self.is_done():
                return
            self.step()

    def add_thing(self, thing, location=None):
        """Add a thing to the environment, setting its location. For
        convenience, if thing is an agent program we make a new agent
        for it. (Shouldn't need to override this.)"""
        if not isinstance(thing, Thing):
            thing = Agent(thing)
        if thing in self.things:
            print("Can't add the same thing twice")
        else:
            thing.location = location if location is not None else self.default_location(thing)
            self.things.append(thing)
            if isinstance(thing, Agent):
                thing.performance = 0
                self.agents.append(thing)

    def delete_thing(self, thing):
        """Remove a thing from the environment."""
        try:
            self.things.remove(thing)
        except ValueError as e:
            print(e)
            print("  in Environment delete_thing")
            print("  Thing to be removed: {} at {}".format(thing, thing.location))
            print("  from list: {}".format([(thing, thing.location) for thing in self.things]))
        if thing in self.agents:
            self.agents.remove(thing)


class TrivialVacuumEnvironment(Environment):
    """This environment has two locations, A and B. Each can be Dirty
    or Clean. The agent perceives its location and the location's
    status. This serves as an example of how to implement a simple
    Environment."""

    def __init__(self):
        super().__init__()
        self.status = {loc_A: random.choice(['Clean', 'Dirty']),
                       loc_B: random.choice(['Clean', 'Dirty']),
                       loc_C: random.choice(['Clean', 'Dirty']),
                       loc_D: random.choice(['Clean', 'Dirty']),
                       loc_E: random.choice(['Clean', 'Dirty']),
                       loc_F: random.choice(['Clean', 'Dirty']),
                       loc_G: random.choice(['Clean', 'Dirty']),
                       loc_H: random.choice(['Clean', 'Dirty']),
                       loc_I: random.choice(['Clean', 'Dirty'])}

    def thing_classes(self):
        return [ TableDrivenVacuumAgent]

    def percept(self, agent):
        """Returns the agent's location, and the location status (Dirty/Clean)."""
        return agent.location, self.status[agent.location]

    def execute_action(self, agent, action):
        """Change agent's location and/or location's status; track performance.
        Score 10 for each dirt cleaned; -1 for each move."""
        if action=='Right':
            agent.location = loc_B or loc_A or loc_G or loc_H
            agent.performance -=1
        elif action=='Left':
            agent.location = loc_D or loc_E 
            agent.performance -=1
        elif action=='Up':
            agent.location = loc_C or loc_F
            agent.performance -=1
        elif action=='Down':
            agent.location = loc_I
            agent.performance -=1
        elif action=='Suck':
            if self.status[agent.location]=='Dirty':
                agent.performance+=10
            self.status[agent.location]='Clean'

    def default_location(self, thing):
        """Agents start in either location at random."""
        return random.choice([loc_A,loc_B,loc_C,loc_D,loc_E,loc_F,loc_G,loc_H])


if __name__ == "__main__":
    agent = TableDrivenVacuumAgent()
    environment = TrivialVacuumEnvironment()
    environment.add_thing(agent)
    
    print(environment.status)
    print("AgentLocation : {0}".format(agent.location))
    print("Performance : {0}".format(agent.performance))
    time.sleep(5)

    for i in range(15):
        environment.run(steps=1)
        print(environment.status)
        print("AgentLocation : {0}".format(agent.location))
        print("Performance : {0}".format(agent.performance))
        time.sleep(10)
```
## OUTPUT

![WhatsApp Image 2022-04-07 at 9 33 06 PM](https://user-images.githubusercontent.com/75235167/162247429-0fb292bf-9ba6-43a3-bb6a-25c49b61b244.jpeg)


## RESULT
Thus, an AI agent was developed and PEAS description is given.

