# Commands
## ICommand
Each plugin can have multiple commands.

Also see [`ILocalizedCommand`](/api-reference/command.md#ilocalizedcommand).

Each command has the following properties:

Member |    Type    | Description
-------|------------|-------------
`name` | `string`   | Friendly-name of the command (not necessarily the words used to call it).
`match`| `string | string[] | `[`IDynamicMatch`](/api-reference/command.md#idynamicmatch) | The word(s) the user can say to execute this command. Use "#" in the string as an ordinal place holder. Use "*" as a wildcard placeholder. Lastly, a function [`IDynamicMatch`](/api-reference/command.md#idynamicmatch) can be used for the most advanced cases.
`description` | `string` | _(optional)_ Detailed description visible in the options page.
`global` | `boolean` |  _(default: false)_ let the command match on any page (not restricted by the `match` of the Plugin)
`pageFn` | `(number) => Promise<any> | (string) => Promise<any>` | _(optional)_ The async function to run on the page when the command is called. There will be a number parameter if the match string accepts an ordinal (eg. has a `#`) in it, or a string argument if the match string accepts a wildcard (eg. has a `*` in it).
`delay` | `number | number[]` | _(optional)_ How long to wait for additional input for before executing this command. <br><br> For example with the Google command the user can have a long search term like "search how to boost my wifi signal" in order to prevent the command from executing the search as soon as it hears "search how" instead of the full phrase, we put a delay so that it waits for X ms since the last voice input. <br><br> Use an array with indices that correspond to the different match strings if you should have different delays based on the match string.
`nice` | [`INiceCommand`](/api-reference/command.md#inicecommand) |  _(optional)_ See [`INiceCommand`](/api-reference/command.md#inicecommand).
`run` | `((tabIndex: number) => any)` | _(optional)_ Code to run in the Chrome extension context.
`test` | `() => void` | _(optional but recommended)_ Selenium unit test for this command.


## IDynamicMatch

A function that decides whether a command matches based on a transcript input for more dynamic command word possibilities.

Member | Type | Description
-------|------|---------------
`fn` | `(transcript: string) => `[`MatchResult`](/api-reference/command.md#matchresult)`| undefined` | A function that takes in the transcript and returns a [`MatchResult`](/api-reference/command.md#matchresult)if the command should execute on the given transcript.
`description` | `string` | Used to decribe to the user what command words match. Seen in plugins list in options.

## MatchResult

**Type:** `boolean|any[]`

Array of args to pass over to `pageFn`.


-or-


`false` if there is a partial match. If there is a partial match we will delay other commands that might already want to execute.

::: tip E.g.
Imagine there's a command word for <span class="voice-cmd">reddit</span> and a [dynamic match command](/api-reference/command.md#idynamicmatch) for <span class="voice-cmd">reddit message</span> that are both valid on a given page. If the user says <span class="voice-cmd">reddit message</span> the transcripts will come down the wire something like this:

 - red
 - reddit
 - read it
 - read it mess
 - reddit mess
 - reddit message

 Can you see the problem? Our "reddit" command will execute even though we only want "reddit" message to.
:::

If you don't want the first "reddit" command to match, return `false` when there is a partial match for the dynamic <span class="voice-cmd">reddit</span> command for transcripts that start with "reddit".

## ILocalizedCommand

Also see [`ICommand`](/api-reference/command.md#icommand).

Member | Type       | Description
-------|------------|-------------
`name`   | `string`   | The original name of the command to match this localized version with.
`description` | `string` | _(optional)_
`match`  | `string | string[] | `[`IDynamicMatch`](/api-reference/command.md#idynamicmatch) | The way localized version of command match can be completely different from the base English version.
`delay` | `number | number[]` | _(optional)_ Delays for a localized version of a command can be completely different from the base English version.
`nice` | [`INiceCommand`](/api-reference/command.md#inicecommand) | _(optional)_ See [`INiceCommand`](/api-reference/command.md#inicecommand).

## INiceCommand

**Type:** `string | ((rawInput: string, matchOutput: any[]) => string)`


Sometimes we want to adjust the transcript as it is shown on the live text. For example if the user says <span class="voice-cmd">go to are meal time videos</span> we would want to show that as <span class="voice-cmd">go to r/mealtimevideos</span>.

Returns the complete "live text" that should be shown. `rawInput` is the transcript (eg. "go to are meal time videos") `matchOutput` is an array returned from the match command if [`IDynamicMatch`](/api-reference/command.md#idynamicmatch)is used.

