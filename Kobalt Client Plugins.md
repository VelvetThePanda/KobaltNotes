
This document describes the overview and technical design of Kobalt's [JS] plugin system.

Plugins are written in JS, and loaded per-server. Multiple plugins be loaded on the same server, but for security reasons, plugins cannot "see" multiple servers; this has the tradeoff of using more resources (as plugins are instantiated once per-server, and persisted for the lifetime of the shard). 




# Commands:

## TODO: 
[This PR](https://github.com/Remora/Remora.Commands/pull/20) Must be merged; it serves as the groundwork to allow plugins to be registered at runtime.

From C#, the command code would look something like this:


```csharp

var plugin = LoadPlugin(somePluginPath, RegisterCommandsFunc, _commands);

// ...

internal void RegisterCommandsFunc(JSObject commands, CommandTreeBuilder builder)
{
	foreach (var command in commands)
	{
		var cmd = new CommandGroupBuilder()
		.WithName(command["name"])
		.WithDescription(command["description"])
		.WithInvocation(BuildInvocationDelegate(command["parameters"]))
	}
}
```

And in JS:

```js
exports.commands = [
	{
		name: "my_plugin_command",
		description: "A pretty cool command from a plugin!",
		parameters: {
			string: CommandHelper.string(min: 10),
			user: CommandHelper.user(),
			delay: CommandHelper.delay(),
			
		}
	}
]


async function my_plugin_command(state, args) {
	await args.user.send_message("Hi from plugin!!");
}
```
