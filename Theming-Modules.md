# Modules

***Table of Contents***
* [Introduction](#Introduction)
* ***[Examples of modules](#Examples-of-modules)***
    * [Metatable of objects](#Metatable-for-objects)
	* [Single value return function](#Single-value-return-function)
* ***[Code practices and recommendations](#Code-practices-and-recommendations)***
	* [Avoid using theme specific variables.](#Avoid-using-theme-specific-variables)


## Introduction

Modules in Outfox are Lua files that themes can utilize to expand functionality in a quick manner. They are Drag'n'drop to setup and just require one line to add into a screen or function that requires it.

```lua
LoadModule( path_to_module )
-- or 
LoadModule( path_to_module )()
```

The loading process for modules goes as follows:

- If there’s a `Modules` folder present on your theme, it will look for the module in that location first.

- If no `Modules` folder is present on the current theme, it will instead look for a similar module name in the `_fallback` theme folder.

The kind of content in a module can be in any form, as long as it returns something, like a table, number, string, among other things.

## Examples of modules

### Metatable for objects

#### **`meta.lua`**
```lua
return setmetatable({},
	{
		__call = function(this, msg)
			-- the or operator in this example works if the variable expected is nil.
			-- if msg has something, it'll report it, otherwise, report "nothing".
			SCREENMAN:SystemMessage( "I received a message, and it’s " .. (msg or "nothing") )
			return this
		end
	}
)
```

```lua
-- Inserting module.
LoadModule("meta.lua")()
--> "I received a message, and it’s nothing"

LoadModule("meta.lua")("a sentence of words")
--> "I received a message, and it’s a sentence of words"
```

In this example, we’re creating a Lua Metatable that uses the `__call` Metamethod to modify its behavior. In this case, upon calling this module with the `()` operator, the SystemMessage will appear with the message that is contained inside the parentheses as a String, or "nothing" if nothing was inserted.

We finalize the command by return the same Metatable to allow future operations to take place. Because of this, it allows for command chaining when calling the module directly with a custom function.

#### **`meta.lua`**
```lua
return setmetatable({
	-- This command will increase the number above and report it.
	number = 0,
	MyCustomFunction = function(this)
		number = number + 1
		SCREENMAN:SystemMessage("hi! number is ".. this.number)
	end
},{
	__call = function(this, msg)
		return this
	end
})
```
```lua
-- Inserting module.
-- Notice the ammount of times we're calling the same command.
LoadModule("meta.lua")():MyCustomFunction():MyCustomFunction():MyCustomFunction()
--> "hi! number is 1"
--> "hi! number is 2"
--> "hi! number is 3"
```

For more information about what operators can be modified in Metatables, [Check the Lua documentation page for Metatables and Metamethods.](https://www.lua.org/manual/5.3/manual.html#2.4)

### Single value return function

#### **`numbersum.lua`**
```lua
return function(num1, num2)
	return num1 + num2
end
```
```lua
-- Inserting module.
print( LoadModule("numbersum.lua")(2,2) )
--> 4
```

This example showcases a module that when called, it will perform the sum of two numbers. Unlike the Metatable example, this can be performed once, given that the value returned is not the function anymore, but the result of the sum. You can use this kind of functions for quick operations that need to be calculated with parameters.

```lua
local sumnumbers = LoadModule("numbersum.lua")(4,2)
SCREENMAN:SystemMessage( "The sum was " .. sumnumbers )
--> "The sum was 6"
```

## Code practices and recommendations

### Avoid using theme specific variables.

Themes can contain variables stored in different spots, where they can be accessed, for example, global variables. It is recommended not to use these values as they can get lost if switching to another theme, and the module can begin to break as the variable is now lost.

```lua
-- A red variable.
-- Given it's not a local, it will be stored globally.
My_Cool_Variable = color("#ff0000")
```

#### **`module.global.lua`**
```lua
-- Let's say this module will convert the variable
-- into a darker red color.
return function()
	return BoostColor( My_Cool_Variable, 0.5 )
end
```

If then, we use the module in the theme that set the variable, this following would happen.
```lua
local col = LoadModule("module.global.lua")()
print( tostring(col) )
--> "#990000"
```

But now, let's switch themes or restart the game, and the following will happen.
```lua
local col = LoadModule("module.global.lua")()
--> "error running function. My_Cool_Variable is undefined"
print( tostring(col) )
```

If you need the variable to be detected on the module, then provide a fail-safe value for the variable that can lose data.

#### **`module.global.lua`**
```lua
-- Let's say this module will convert the variable
-- into a darker red color.
return function()
	-- Let's use the "or" operator again, to define a failsafe value.
	-- given it is nil.
	-- The default value we'll give is just full white.
	return BoostColor( My_Cool_Variable or color("#ffffff"), 0.5 )
end
```