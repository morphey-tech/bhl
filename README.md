# **B**e**H**avior **L**anguage

**bhl** is a programming language specifically tailored for Behavior Trees(BT) coding using familiar imperative style patterns. It was presented at the [nucl.ai](https://nucl.ai/) conference in 2016. Here's the [presentation slides](https://docs.google.com/presentation/d/1Q1wpy9M5XPmY6zU9Kjo2v9YiJQjrDBXdDZaSjcuh71s/edit?usp=sharing). 

Please note that bhl is in pre-alpha state and currently targets only C# platform. Nonetheless it has been battle tested in real world project and heavily used by BIT.GAMES for mobile games development.

## bhl features

* [ANTLR](http://www.antlr.org/) based: C# frontend + C# interpreting backend
* Statically typed
* Supports core BT building blocks: *seq, paral, paral_all, prio, not, forever, until_success, until_failure,* etc
* Basic types: *float, int, bool, string, enums, arrays, classes*
* Supports imperative style control constructs: *if/else, while, break, return*
* Allows user defined: *functions, lambdas, classes*
* Supports C# bindings to user types and functions
* Golang alike *defer*
* Passing arguments to function by *ref* like in C#
* Multiple returned values like in Go
* Hot code reload
* Strict control over memory allocations 

## Code samples

### Imperative style mixed with BT

```go
func AlphaAppear(int id, float time_to_appear) {
  float time_start = time()
  paral {
    forever {
      float alpha = clamp01((time()-time_start)/time_to_appear)
      SetObjAlpha(id: id, alpha: alpha)
    }
    Wait(sec: time_to_appear)
 }
}
```

### Imperative style only

```go
func Unit FindUnit(Vec3 pos, float radius) {
  Unit[] us = GetUnits()
  int i = 0
  while(i < us.Count) {
    Unit u = us.At(i)
    if(u.position.Sub(pos).len < radius) {
     return u
    } 
    i = i + 1
  }
  return null
}
```

### Generic initializers support

```go
Color c = {r:0.9, g:0.5, b:0.1, a:0.3}
Vec3[] vs = [{x: 10}, {y: 100, z: 100}, {y: 1}]
```

### **ref** support

```go

Unit FindTarget(Unit self, ref float dist_to_target) {
...
  dist_to_target = u.position.Sub(self.position).length
  return u
}

float dist_to_target = 0
Unit u = FindTarget(self, ref dist_to_target)
```
### **Multiple returned values** support

```go

Unit,float FindTarget(Unit self) {
...
  float dist_to_target = u.position.Sub(self.position).length
  return u,dist_to_target
}

Unit u,float dist_to_target = FindTarget(self)
```

### **lambda** support

```go
Unit u = FindTarget()
float distance = 4
u.InjectScript(func() {
  paral_all {
    PushBack(distance: distance)
    Stun(time: 0.4, intensity: 0.15)
  }
})
```

### Function pointers

```go
bool^(int) p = func bool(int b) { return b > 1 }
return p(10)
```

### **defer** support

```go
seq {
  RimColorSet(color: {r:  0.65, a: 1.0}, power: 1.1)
  defer { RimColorSet(color: {a: 0}, power: 0) }
     ... 
}
```
### Some unit's top behavior

```go
func UNIT_GREMLIN(float radius_max)
{
  int HEAVY_LAST_TIME = 0
  int ROLL_LAST_TIME = 0

  paral_all {
    SCATTER_AFTER_GET_HIT()
    forever {
      paral {
        StateChanged()
        prio {
          OVERRIDE()
          SPAWNED()
          ON_WATCH()
          WANDER()
          DEAD()
          DYING()
          SCATTER()
          paral {
            RETHINK_LISTENER(func bool () {
              return HEAVY_LAST_TIME <= time() || 
                     ROLL_LAST_TIME <= time()
            })
            prio {
              GREMLIN_ROLL_ATTACK(stamp : ref ROLL_LAST_TIME, radius_min : 3, radius_max : radius_max, radius_attack : 2, cooldown : 6, global_cooldown : 4, push_dist : 1)
              GREMLIN_HEAVY_ATTACK(stamp : ref HEAVY_LAST_TIME, radius_max : radius_max, cooldown : 8, global_cooldown : 5, angle : 60)
              ATTACK()
            }
          }
          ATTACK()
          IDLE()
        }
      }
    }
  }
}
```
## Architecture

![bhl architecture](https://puu.sh/qEkYv/edf3b678aa.png)

bhl utilizes a standard interpreter architecture with a **frontend** and a **backend**. Frontend is responsible for reading input files, static type checking and bytecode generation. Binary bytecode is post-processed and optimized in a separate stage. Processed byte code can be used by the backend. Backend is a interpreter responsible for runtime bytecode evaluation. Backend can be nicely integrated with Unity3d. 

### Frontend

In order to use the frontend you can use the **bhl** tool which ships with the code. See the quick build example below for instructions.  

### Backend

Before using the backend you have to compile the **bhl_back.dll** and somehow integrate it into your build pipeline. See the quick build example below for instructions.  

## Quick build example

Currently bhl assumes that you have [mono](http://www.mono-project.com/) installed and its binaries are in your PATH.

In the example directory you can find a simple illustration of gluing together **frontend** and **backend**. 

Just try running *run.sh* script: 

> cd example && ./run.sh

This example executes the following [ simple script ](example/unit.bhl)

```markdown
Unit starts...
No target in range
Idling 3 sec...
State changed!
Idle interrupted!
Found new target 703! Approaching it.
Attacking target 703
Target 703 is dead!
Found new target 666! Approaching it.
State changed!
Found new target 902! Approaching it.
...
```

Please note that while bhl works fine under Windows the example assumes you are using \*nix platform.     

### Unity3d integration

The example script has also a special Unity3d compatibility mode. It illustrates how you can build a bhl backend dll(**bhl_back.dll**) for Unity3d. After that you can put it into Assets/Plugins directory and use bhl for your Unity3d game development. This mode can be enabled just as follows: 

> cd example && ./run.sh -unity

## Building

bhl comes with its own simple build tool **bhl**. bhl tool is written in PHP and should work just fine both on \*nix and Windows platforms. 

It allows you to build frontend dll, backend dll, compile bhl sources into a binary, run unit tests etc. 

You can view all available build tasks with the following command:

> $ bhl help

## Tests

For now there is no any documentation for bhl except presentation slides. However, there are many [unit tests](test.cs) which cover all bhl features.

You can run unit tests by executing the following command:

> $ bhl test

# Roadmap

## Version 0.9

1. ~~**ref** semantics similar to C#~~
2. ~~Generic functors support~~
3. ~~Generic initializers~~
4. ~~Multiple return values support~~
5. **while** syntax sugar: **for(...) {}** support
6. Ternary operator support
7. User defined classes

## Version ???

1. Byte code optimization
2. More optimal executor