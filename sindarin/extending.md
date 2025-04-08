## Extending the debugger with Sindarin scripts 

The debugger can be scripted in two ways: from the live scripting pane, during the debugging of an executing program, or through integrated debugging menus.
It is important to remember that every executing script might advance the execution forward or change its state.
Therefore, we must be careful when manipulating the execution as it can have irreversible effects.

In the following examples, we will debug a `STON` parsing operation, and we will implement domain-specific stepping operator for the STON parser.
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

The advanced debugger menu (fig. *@fig:debugger-advanced-menu@*) contains all Sindarin debugger scripts that are part of the standard Pharo debugging tools.
These scripts enable unconventional debugging actions that facilitate the free exploration of an execution:

- `SindarinCreateCommandFromScriptCommand`: Create a command from the current debugging script.
- `SindarinJumpToCaretCommand`: Move the execution to the caret position without executing any code but preserving current state. Execution resumes from this position.
- `SindarinLoadScriptCommand`: Load a debugging script.
- `SindarinRefreshCommand`: Refresh the UI. Necessary after manual control of Sindarin.
- `SindarinRemoveCommandCommand`: Remove the command corresponding to the current debugging script.
- `SindarinRunScriptCommand`: Run the current debugging script.
- `SindarinSaveScriptCommand`: Save the current debugging script.
- `SindarinStepBytecodeCommand`: Step a single bytecode.
- `SindarinStepCommand`: Step in.
- `SindarinStepOverCommand`: Step over.
- `SindarinStepToMethodEntryCommand`: Step to the beginning of the next method, then returns debugger control.
- `SindarinStepToNextInstanceCreation`: Steps to the next object instantiation.
- `SindarinStepToReturnCommand`: Steps to the next method return.
- `SindarinSTONParsingCommand`: Not described command.
- `SindarinSkipCommand`: Skips execution of the next step, then returns debugger control.
- `SindarinSkipUpToCommand`: Skips execution and stops before the selected instruction (or the instruction preceding the cursor).
- `SindarinStepToNextExecutionInClassCommand`: Steps until the execution comes back to code executing in the current class.
- `SindarinStepToNextExecutionInObjectCommand`: Steps until the execution comes back to code executing in the current receiver.

![Sindarin advanced debugger menu.](graphics/debugger-advanced-menu.drawio.pdf label=fig:debugger-advanced-menu)


### Adding a new command in the advanced debugger menu

Now, let us imagine that we will actually reuse a lot our script to step to the STON `parseObject` method.
We might want to have this script available from the advanced debugger menu!
To appear in the advance menu, scripts are implemented as commands which are automatically integrated into the menu.

We create such command by writing a class that inherits from the base classe `SindarinCommand`.
In this example, let us create this class into a `Sindarin-Chapter-Commands` package:

```Smalltalk
SindarinCommand << #SindarinSTONParsingCommand
	slots: {};
	package: 'Sindarin-Chapter-Commands'
```

In this class, we must write specific methods for the command to be available.
Class side, we need to write three methods: `defaultIconName` sets the icon used in the menu for that command, `defaultName` sets the name used in the menu for that command,  and `defaultDescription` sets the popover description shown when passing the mouse of that command in the menu.

```Smalltalk
defaultIconName
	^#through

defaultName
	^ 'steps to next #parseObject'

defaultDescription
	^ 'Advances to the next STON object parsing.'
```

Instance side, we only have to write the `execute` method that is called upon a click on the command in the menu.
We cannot reuse exactly our first script from the scripting pane.
We use the `context` interface to obtain a reference to the debugger presenter and its model that control the debuggeed execution.
First, we use the debugger (i.e., the context) to obtain a reference to the object exposing the Sindarin API (line 3).
Second, we encapsulate the script in a block (line 4) that executes the script without updating the debugger presenter.
Failing to do such encapsulation will trigger an update of the debugger presenter views after each step, which considerably slows down the script execution.

```
execute
1	| sindarin |
2	sindarin := self context sindarinDebugger.
3	self context debuggerActionModel preventUpdatesDuring: [
4			sindarin stepUntil: [ sindarin selector = #parseObject ] ]
```

Our new command is not yet available in the menu.
To make it appear, we must modify the class side `defaultName` method to add the pragma `<codeExtensionDebugCommand:1>`.
This pragma is used by the system to collect the commands that must be available from the menu, and instantiate them into the menu.

```Smalltalk
defaultName
	<codeExtensionDebugCommand:1>
	^ 'steps to next #parseObjects'
```

![The new command is available from the advanced menu!](graphics/new-command.drawio.pdf label=fig:integrated-command)


### Adding a new menu extending the debugger

We just created on script that implements a domain-specific stepping operator that steps between object parsing operations.
However, the STON parser has a variety of parsing operations that may also be interesting.
It makes little sense to group these commands under the Advanced menu, as they are domain-specific, unlike the general-purpose scripts typically found there.

Therefore, we will build a new menu for our STON stepping scripts!
For that, we will create a generic command that we use to instantiate a new menu with the STON domain-specific stepping operators.