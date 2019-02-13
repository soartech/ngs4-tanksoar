# TankSoar using NGS-4

The following instructions are a guide to setting up and using NGS-4 with TankSoar. We have two examples of agents you can walk through. One simply shows some of the syntax, and the other uses the NGS design philosophy to create an agent.

In this tutorial we will be using:
- TankSoar and Soar 9.4.0 provided by the Soar Tutorial [https://soar.eecs.umich.edu/downloads/Archives/9.4.0/SoarSuite/](https://soar.eecs.umich.edu/downloads/Archives/9.4.0/SoarSuite/). Running on a MacBook Pro running OS X Mojave, it was difficult to get tcl processing to work correctly with the latest version of Soar (9.6.0 loads tcl differently).
- NGS-4 [https://github.com/soartech/new-goal-system-4](https://github.com/soartech/new-goal-system-4) checked out at commit 3fae9e7. This is the last version of NGS-4 compatible with 9.4.0.
- Eclipse IDE [https://www.eclipse.org/](https://www.eclipse.org/)
- SoarIDE [https://github.com/soartech/soaride](https://github.com/soartech/soaride)

In the folder where you have the TankSoar launch file (TankSoar.command in OS X, TankSoar.bat in Windows), add a file called `settings.soar` with the line

```
cli tcl on
```
    
On OS X Mojave, System Integrity Protection needed to be disabled.

# Setting up your environment

Install Eclipse and the Soar IDE.

In Eclipse create a new project for your agents.
Right click in Package Explorer and choose File from the drop down menu. Then choose New -> Project… -> General -> Project. 

Give your project a name (TankSoar-NGS) and choose a location for it (or use the default location). You will need to use this location later when you tell TankSoar where your agent is.

Click Finish

In Package Explorer, right click on your project and choose “Include Soar Project Support.”

It’s useful to switch to the Soar Perspective. From the drop down menu select Window -> Perspective -> Open Perspective -> Soar. If Soar does not show up, choose Other and then Soar. If it isn’t in that menu, verify you have the Soar IDE installed and you restarted Eclipse.

# Testing NGS-4 in an agent

For this first example, we will implement the Wander agent in NGS-4. In this agent we will only use the most basic concepts from NGS. It should be very similar to the agent provided by the Soar tutorial.

Right click on the project and choose New -> Soar Folder. Call this Wander. By default it will give you a load.soar as the entry point for your agent. We want to source the NGS productions before we do anything else. Clone or download NGS-4 from the github repository and put it under the main project folder. To see it in Eclipse, right click on the project and select Refresh.

The first line in Wander/load.soar should be:

```
source "../new-goal-system-4/load.soar"
```

At this point we want to verify NGS loads properly in TankSoar. Launch TankSoar and source the `load.soar` file. Right now the only productions should be those for NGS-4. Check that the macros work by typing `np s1` and you should see an output similar to:

```
np 2 s1
+---------------------------------------+
+ OBJECT: s1
(S1 ^epmem E1)
  (E1 ^command C1)
  (E1 ^present-id 1)
  (E1 ^result R2)
(S1 ^io I1)
  (I1 ^input-link I2)
  (I1 ^output-link I3)
(S1 ^reward-link R1)
(S1 ^smem S2)
  (S2 ^command C2)
  (S2 ^result R3)
(S1 ^superstate nil)
(S1 ^svs S3)
  (S3 ^command C3)
  (S3 ^spatial-scene S4)
(S1 ^type state)
+---------------------------------------+
```

Congratulations! You're running NGS-4. This is the NGS Print command. Also try the `nps` command.

# Wander

For the first example, we are going to map the wander agent from the tutorial directly into NGS-4 commands. This will give an example of the command structure, but not necessarily the goal system approach. In fact, the following is **NOT** the recommended usage. It should, however give you a feel for the syntax.

Start with the first production in the Soar tutorial wander.soar file. In NGS-4, this would be:

```
sp "propose*move
    [ngs-match-top-state <s> "" <il>]
    [ngs-bind <il> blocked.forward:no]
-->
    [ngs-create-atomic-operator <s> move <o>]
    [ngs-create-attribute <o> actions.move.direction forward]
    "
```

Let's break it down. First off, you may notice that we are using quotes instead of curly braces for the productions. This is because all of these productions will go through the tcl processing to be turned into standard Soar productions. The first line will do two things. It will bind the top state to `<s>` and the input-link to `<il>`. All NGS productions should start with one `[ngs-match-` command. Rarely, you may use two match commands, but be careful you know what you are doing (Example: binding to two goals simultaneously). The second line will check the input-link to see if `blocked.forward` is equal to `no`.

On the right hand side, we create an operator. Normally, we would probably do something like `[ngs-create-typed-object-by-operator`. In this example, we mostly want to show the general syntax of NGS-4 before jumping into the design patterns typically employed. The last line will create the WMEs on the operator that later get applied to the `output-link`.

Since you're using the SoarIDE you should be able to click on this production and look in the Soar Source Viewer. It should show the lines expanded into standard Soar productions. It should look like:

```
sp {propose*move
    (state <s> ^superstate nil)
                 (<s> ^io <io31>)
                   (<io31> ^input-link <il>)
    (<il> ^blocked <blocked>)
                     (<blocked> ^forward no)
-->
    (<s> ^operator <o> + =)
          (<o> ^name     move
                       ^type     atomic)
          (<o> ^__tagged*ngs*i-supported *YES* +)
          
    (<o> ^actions.move.direction forward +)
    }
```

If you don't get this expansion, check to see if the SoarIDE is installed and that you have a `.soaragent` file that sources both your wander files and NGS-4.

You can look at the `Wander\wander.soar` to see the rest of the productions converted to NGS-4.

# NGS approach to TankSoar

If you look in the `GoalBased` folder you will find an agent designed with the NGS-4 approach. Let's first look at the files we're loading. As usual, load.soar is our entry point.

```
source "../new-goal-system-4/load.soar"

CORE_Source variables.soar
CORE_Source goal-types.soar
CORE_Source output-command-types.soar

CORE_Source main-agent.soar
CORE_Source tank-status.soar

CORE_Source search.soar
CORE_Source engage-enemy.soar
CORE_Source run-away.soar
```

After loading NGS-4, we create a bunch of variables in `variables.soar`, such as `CORE_CreateMacroVar TS_DECISION_MANEUVER maneuver-decision`. This is a recommended way to create tags and decision names so that we don't accidentally have a typo in our soar productions to prevent something from firing. When you look through the productions, you'll see this used as `$TS_DECISION_MANEUVER`.

`goal-types.soar` is where we will define our NGS goal structures. It's imporant that you make these calls to `NGS_DeclareGoal` before using macros such as `[ngs-create-goal-in-place`. These macros will generate the appropriate goal pools. Similarly `output-command-types.soar` has the macros for declaring the typed objects we will use. In this case, we only have output commands. In general, you may have multiple files for different typed objects.

`main-agent.soar` can be thought of as the entry-point of our agent. In Soar, all productions fire in parallel, but we have a few structures that need to be built before anything productive happens. We create `<s1>.tank-soar`, an identifier where we will store useful tags and information. It's often a good idea to have a way to traverse the data our agent will use. In the productions that follow you'll see we check the input link for data. This is primarily done as a convenience for this simple agent. In practical applications, we will often link data from `input-link` into a more easy to access place under `tank-soar`.

In `main-agent.soar` we create the top level goal for our program - to win the game. Under this goal we create our first decision. We want to decide what maneuver to take. This maneuver can be to move, turn, fire, or some combination of those. For the decision we can create 3 types of goals - search, engage the enemy, or run away. At any time, at least one of these goals will exist (search) and up to all 3 can exist simultaneously.

This file includes the methods used to make the decision once the goals have been created. For this case we explicitly create 2 ways of making the choice. The last (search) will occur when it is the only possible option. In the design of this agent, only 1 to 3 goals should exist at any given time.

`tank-status.soar` contains a few examples of elaborations that you might use to make it easier to find data of interest under `<s1>.tank-soar`.

You can look into `search.soar`, `engage-enemy.soar`, and `run-away.soar` to see when each of these goals is created and how they operate. In each of the files are notes about how they are intended to operate. The productions are designed to loosely mirror the Soar tutorial approach. In these methods we employ a couple of different ways of setting up goals. For turning left/right in search, we create a sub-decision to pick one if both exist. For engaging the enemy, we try to directly cover all possible options in the parameter space. For run away we try to identify all the possible reasons to perform it and then create a single goal to do so. These are three different approaches to setting up goals, and each has its advantages. When the parameter space is simple, it is often best to cover the cases ensure there are no overlapping cases. If the cases are very complex, series of decisions often work well. Somewhere in between, it's sometimes easiest to create series of tags and conditions that cause the goal creation to fire.

# Conclusion

We hope you've found this example helpful. NGS-4 is under active development. If you find a feature missing or have difficulty understanding how something should work, please feel free to reach out to us.

A couple of topics we haven't covered in this tutorial but would be worth revisiting are:

- Context variables: How to create structures such as dynamic binning, time delayed values, etc.
- Function operators: Creating all at once objects and then copying them to a desired location. These are particularly useful for copying data over from the input link or preparing an output command for the output link.
- Explanation variables: Giving a view of the internal operating picture to the outside world.

