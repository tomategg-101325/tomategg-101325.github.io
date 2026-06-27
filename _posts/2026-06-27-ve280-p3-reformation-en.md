---
layout: post-en
title:  "Rewriting Data Structures Course Project 3"
date:   2026-06-27 10:00:00 +0800
categories: en-us
tags: programming cpp notes
math: False
---

Here's the story: I'm taking a data structures course this semester. Although I don't find it particularly difficult, Project 3 came with so many restrictive rules and constraints that it made me quite frustrated. So after *submitting the required version*, I took some extra time to rebuild a program from scratch that implements exactly the same functionality, but with **no restrictions whatsoever**—except for using C++17. In the end, I came up with an architecture that uses **callback functions for decoupling**. This post is a brief introduction to **how I built this architecture**.

[Here](https://github.com/tomategg-101325/species-simulator) is the source code for the rewritten version. Since the functionality is also described in the readme, I won't go into too much detail here. In short, it's a **highly extensible species evolution simulation program**.

Note: This post is a translated version. See [here](/zh-cn/2026/06/27/ve280-p3-reformation-cn.html) for the original content.

## Pain Point: Complexity in the Simulator Layer

Due to the project requirements[^1], in the submitted code, the simulator layer had a huge `switch` statement block that directed the underlying engine to perform different physical operations based on the instruction type of the creature program. Below is a code snippet (not exactly the submitted code, but the functionality and structure are identical):

```cpp
void Simulator::SimulateCreature(unsigned int creature_index) {
  ...
  current_instruction = engine_.get_instruction(creature_index);  // obtain current instruction
  ...
  switch (current_instruction.opcode) {
    case Opcode::kHop:  // hop forward
      engine_.MoveCreature(creature_index);
      break;
    case Opcode::kLeft:  // turn left
      engine_.TurnCreatureLeft(creature_index);
      break;
    ...
  }
  ...
}
```

Functionally, this works fine, but it has several drawbacks:

- The single function body is **overly complex**;
- Adding new instructions in the future would require modifying this `switch` block, making it **harder to maintain**;
- Instruction type discrimination and instruction execution are all coupled together, which **makes debugging inconvenient**.

In fact, my local code quality tool directly warned that the function containing this `switch` block had excessive complexity. Fortunately, the online judge didn't give me a deduction for that.

But why was instruction execution placed inside the simulator in the first place? Because the engine is managed by the simulator, and the execution of instructions depends on various physical operations provided by the engine (e.g., moving forward, turning left, infecting the creature ahead, etc.).

## Association: Callback Functions Learned in Embedded Development

After I finished writing that function, I kept thinking about whether I could **decouple** instruction execution from the simulator.

One day while showering, I suddenly recalled **callback functions** learnt from embedded development. Callbacks are typically used with **interrupts** to handle specific events when they occur. The basic architecture taught by the teacher was:

- Callback layer:
  - Define a **function type** (i.e., the parameter types, count, and return type of the function), *excluding parameter names, function name, and concrete implementation*;
  - Declare an array of **function pointers** of that type as a **registry**;
  - Provide an **interface** for registering functions and an interface for executing all registered functions.
- Application layer:
  - **Implement** the concrete event handlers, ensuring they conform to the specified function type;
  - **Register** the function pointers of these handlers into the callback layer;
  - Inside the interrupt service routine, simply **call** the execution interface provided by the callback layer.

> 💡 In embedded systems, interrupts are one way to handle events. When an event occurs, the corresponding device notifies the microcontroller (MCU), which then interrupts its current program to process the event.

This architecture allows the callback layer to be completely unaware of what the application layer does—it mechanically executes the callback functions passed to it. Thus, when requirements change, the callback layer does not need to be modified. This is a form of decoupling.

## Solution: Dependency Inversion

In the architecture above, the callback layer does not depend on the concrete implementation of the application layer; instead, both depend on the **function type** of the callback. This is the **Dependency Inversion Principle**, which states:

> High-level modules should not depend on low-level ones; both should depend on **abstractions**.
>
> Abstractions should not depend on implementation details; **implementation details should depend on abstractions**.

In the example above, **the function type of the callback is the abstraction**. By extending this idea, the principle can also be applied to this project. Here's what I did:

- Abstract a **callback function list** `WorldCallbacks` that includes various basic physical operations (getters, setters, checking methods);
- Abstract an execution method abstract class `IMethod`, override its function-call operator for concrete execution, taking the above callback list as a parameter, and make it virtual;
- Inherit from the execution method abstract class to create concrete actions and sensors, implementing their specific execution logic;
- In the simulator, maintain a hash table of `IMethod` pointers, keyed by instruction name for quick lookup;
- Have the engine provide the concrete implementation of `WorldCallbacks`.

This callback list serves as the unified abstraction, and both the simulator layer and the engine layer depend on it. Now, in the simulator's execution logic, we only need to locate the corresponding execution method by the instruction name, call its function-call operator, and pass in the prepared callback list and other relevant parameters. The logic becomes much simpler, and for extensions, we can simply inherit a new execution class.

```cpp
void Simulator::SimulateCreature(unsigned int creature_index) {
  ...
  current_instruction = engine_.get_instruction(creature_index);  // obtain current instruction
  ...
  std::shared_ptr<IMethod> executor = get_executor(current_instruction.opcode);  // obtain executor
  ...
  (*executor)(world_callbacks, creature_index, ...);  // dispatch the executor
  ...
}
```

Although the rewritten `SimulateCreature` function may still look complex on the surface, its core is exactly the three lines above: **fetch the instruction, find the executor, and execute with parameters**. Everything else consists of checks and validations to satisfy various other requirements. If new features need to be added later, **this function never needs to be touched**.

## Footnotes

[^1]: What's the point of even whitelisting header files? They didn't even allow `stdexcept` or `memory`, so I had to throw exceptions using string literals and couldn't use smart pointers—very inelegant indeed.
