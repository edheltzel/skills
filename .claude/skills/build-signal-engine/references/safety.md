# Permission and safety checks

Every model call and every agent process in this build starts with permission checks and
provider safety checks on. That includes the agent following this skill, any subagent it
starts, the keep/drop calls, the digest call, tests, and the daily run.

## Defaults

The engine's daily path is a program that fetches the four sources and calls a model API.
It does not shell out to a coding agent.

When an agent CLI or SDK is invoked anyway (a subagent during the build, or a later call
site in the engine), the base command is the tool's ordinary invocation. Nothing in the
base command, the environment, or the model request body may turn checks off.

The names below are a denylist, not an example to copy into a command. Do not pass
them, do not set them in the environment, and do not put them in the default command the
engine writes:

| Do not pass | What it turns off |
|---|---|
| `--dangerously-skip-permissions` | Claude Code permission checks |
| `--permission-mode bypassPermissions` | Claude Code permission checks |
| `--yolo`, `yolo` | agent approval checks |
| `--ask-for-approval never` | the approval prompt |
| `CLAUDE_DANGEROUSLY_SKIP_PERMISSIONS` | Claude Code permission checks, via the environment |

Also refuse any flag, mode, or variable whose documented effect is to skip permission
prompts, auto-approve with no check, or disable the sandbox. Flag names drift between
tools and versions. The effect is the test, not the spelling.

Do not send a model parameter that disables safety filters, harm blocks, or moderation.
That includes a safety setting of `BLOCK_NONE` or `OFF`, and a moderation-disable or
safety-disable field under any name. Omit those fields. The provider default is the
setting this skill uses.

Do not add these flags to make a test pass, to finish a stage, or because a command
looked inconvenient. Ask the user instead.

There is no config key that turns model safety off. Do not invent one.

## `agent_args_override`

Some agent CLIs accept extra arguments. The engine exposes one opt-in for that, and it is
off unless the user set it.

| | |
|---|---|
| Key | `agent_args_override` |
| Where it lives | `PROFILE.md`, and the engine config copied from it |
| Type | string |
| Default | `""` |
| Off when | missing, null, empty, or only whitespace |
| Who may set it | the user, by editing the config. The build leaves it empty. |

Behavior when a call site builds an agent command:

1. Start from `base_command`. `base_command` already obeys the table above.
2. Read `agent_args_override`. If it is off, run `base_command` unchanged.
3. If it is set, split it with the platform's argument splitter (`shlex.split` in Python)
   and append those tokens to the argument list.
4. Do not run the override through a shell. Do not put it in front of the program name.
   It is appended arguments, nothing else.

```python
import shlex

override = (config.get("agent_args_override") or "").strip()
command = list(base_command)
if override:
    command.extend(shlex.split(override))
```

One helper, `agent_command(base_command)`, is the only place an agent argv is assembled.
The daily digest path does not call it. Any future agent call site must.

A non-empty value is an explicit opt-in. It can disable checks if the user writes those
flags into it. The skill does not write them, does not suggest them as the example value,
and does not set the key to get a run unblocked. A safe thing to put there, if they want
a cap, is a spend or budget flag the tool documents. The example value in config is `""`.

Document the key with a comment next to it in the engine config: empty means permission
and safety checks stay on; a non-empty value is their opt-in and is appended as arguments.
