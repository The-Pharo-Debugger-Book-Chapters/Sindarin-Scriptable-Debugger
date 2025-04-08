## Extending the debugger with Sindarin scripts 

The debugger can be scripted in two ways: from the live scripting pane, during the debugging of an executing program, or through integrated debugging menus.
It is important to remember that every executing script might advance the execution forward or change its state.
Therefore, we must be careful when manipulating the execution as it can have irreversible effects.

In the following examples, we will debug a `STON` parsing operation.
We will use the following code:

```Smalltalk
STON fromString: (STON toString: OrderedCollection new).
```

Each time we refer to "debug", we mean "debug the example expression".


### The Sindarin live scripting pane
The live scripting pane allows us to directly execute scripts from the debugger.
The scripting pane might not be activated, in which case we have to open the debugger extensions menu and activate the scripting pane (*@fig:activating-scripting-pane@*)

![Activating the Sindarin debugger scripting pane.](graphics/scripting-pane.drawio.pdf&label=fig:activating-scripting-pane)

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