## Extending the debugger with Sindarin scripts 

The debugger can be scripted in two ways: from the live scripting pane, during the debugging of an executing program, or through integrated debugging menus.
It is important to remember that every executing script might advance the execution forward or change its state.
Therefore, we must be careful when manipulating the execution as it can have irreversible effects.

In the following examples, we will debug a `STON` parsing operation.
We will use the following code:

```Smalltalk
STON fromString: (STON toString: OrderedCollection new).
```

Each time we refer to "debug", we mean "debug that example expression".


### The Sindarin live scripting pane
The live scripting pane allows us to directly execute scripts from the debugger.
The scripting pane might not be activated, in which case we have to open the debugger extensions menu and activate the scripting pane (fig. *@fig:activating-scripting-pane@*).

![Activating the Sindarin debugger scripting pane.](graphics/scripting-pane.drawio.pdf label=fig:activating-scripting-pane)

In our example, since we're parsing a serialized object, we want to directly get to the method parsing the object.
One might argue that we could just set a breakpoint in that method, then proceed the execution and wait until the breakpoint hits.
That is true.
However, there are advantages to use a script:
- Since we're using an API to a parser, there might be several strategies implementing a "parse object" method, how should we choose in which one set a breakpoint?
- Since we're already in the debugger, it might break our flow to look for the method(s) to set breakpoints then come back to the debugger.

Let us script our debugger: we will write a script that steps the execution until a `parseObject` method is activated.
We write the following in the scripting pane:
```Smalltalk
sindarin stepUntil: [ sindarin selector = #parseObject ]
```

In this script, `sindarin` is an object that exposes the Sindarin API.
The `stepUntil:` interface steps the execution until the condition passed as a parameter in a block.
Here, we the stopping condition is when we reach a method whose selector is `parseObject`.
We execute the script by clicking on the *play* button of the scripting pane, and the debugger arrives to the `parseObject` method of the STON parser.



### The Sindarin advanced debugger menu

### Adding a new command in the advanced debugger menu
New commands can easily be added in the *Advanced* menu of the debugger.

#### Building the new command

- inherit from SindarinCommand
- write the subclass responsibilities requirements
- put a halt in execute
- open a debugger => see the command (with screenshot)

#### Testing the new command


### Adding a new menu extending the debugger