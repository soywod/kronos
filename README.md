# Kronos protocol

Kronos is a synchronized, cross-platform task and time manager, inspired by [Taskwarrior](https://taskwarrior.org) and [Timewarrior](https://taskwarrior.org/docs/timewarrior).

Taskwarrior and Timewarrior are very good and complete tools, but complex and not so easy to understand. [Kronos](https://github.com/soywod/kronos.vim) aims to unify both tools in one, and to be more simple (focusing on what it's really needed). In fact, Kronos is a group of clients which follow this protocol. Feel free to contribute, share
some idea, or even code a Kronos client.

## List of clients

  - [Kronos.vim](https://github.com/kronos-io/kronos.vim)
  - [Kronos.web](https://github.com/kronos-io/kronos.web)
  - Kronos.mobile
  - Kronos.cli
  - Kronos.desktop
  - ... 

## Table of contents

  * [Database](#database)
    * [Read](#read)
    * [Write](#write)
  * [Options](#options)
    * [Hide done](#hide-done)
    * [Enable sync](#enable-sync)
      * [Host](#host)
      * [User id](#user-id)
      * [Device id](#device-id)
      * [Version](#version)
  * [Task](#task)
    * [Id](#id)
    * [Desc](#desc)
    * [Tag](#tag)
    * [Duration](#duration)
    * [DateTime](#datetime)
  * [Repository](#repository)
    * [Create](#create)
    * [Read](#read)
    * [Read All](#read-all)
    * [Update](#update)
    * [Delete](#delete)
    * [Generate Id](#generate-id)
    * [Stringify task props](#stringify-task-props)
      * [List context](#list-context)
      * [Info context](#info-context)
  * [User interface](#user-interface)
    * [Actions](#actions)
      * [Add](#add)
      * [Info](#info)
      * [List](#list)
      * [Update](#update-1)
      * [Delete](#delete-1)
      * [Start](#start)
      * [Stop](#stop)
      * [Toggle](#toggle)
      * [Done](#done)
      * [Undone](#undone)
      * [Toggle hide done](#toggle-hide-done)
      * [Worktime](#worktime)
      * [Context](#context)
    * [CLI](#cli)
    * [GUI](#gui)
  * [Sync](#sync)
    * [Initialization](#initialization)
    * [Notifications](#notifications)

## Database

Each client has its own locale database. It can be a file, a local storage, a
database or whatever, but it should be packaged with the client, so it's
possible to use the client offline. It's used to store [tasks](#task) and
[options](#options).

```typescript
interface Database {
  // User tasks
  tasks: Task[]

  // User options
  hide_done: boolean
  enable_sync: boolean
  sync_host: string
  sync_user_id: string
  sync_device_id: string
  sync_version: number
}
```

### Read

Reads all data from database. Throws `read database failed`.

```typescript
function read(): Database
```

### Write

Writes partial data to database. Throws `write database failed`.

```typescript
type Partial<T> = {
  [P in keyof T]?: T[P]
}

function write(data: Partial<Database>): void
```

## Options
#### Hide done

If `true`, the [list](#list) action does not display done tasks.

#### Enable sync

If `true`, tasks are synchronized with a [Kronos realtime
server](https://github.com/kronos-io/kronos.server) instance.

##### Host

The realtime server hostname. Eg: `localhost:5000`.

##### User id

The user identifier. It should be kept secretly by the users. It will be
prompted each time a user connects for the first time to a Kronos client.

##### Device id

The user's device identifier. It should be hidden from the users, and should be
kept by clients.

##### Version

The current version of the database. Each time the locale database is modified,
sets this version with the current date. This way, all clients are synchronized
with the most recent database version.

## Task

```typescript
interface Task {
  id: Id
  desc: Desc
  tags: Tag[]
  active: DateTime
  last_active: DateTime
  due: DateTime
  done: DateTime
  worktime: Duration
}
```

### Id

```typescript
type Id = number // An integer > 0
```

### Desc

```typescript
type Desc = string
```

### Tag

```typescript
type Tag = string // Matches [0-9a-zA-Z\-_]*
```

### Duration

```typescript
type Duration = number // An integer
```

### DateTime

```typescript
type DateTime = number // A timestamp
```

## Repository
### Create

Receives a partial Task, validates values, formats it as a Task (with default
values) and inserts it into database. Only the description is mandatory. The id
is [auto-generated](#generate-id). Throws `invalid <property>` and `create
task failed`.

```typescript
type Partial<T> = {
  [P in keyof T]?: T[P]
}

type PartialWithDesc<T> = Partial<T> & {
  desc: Desc
}

function create(task: PartialWithDesc<Task>): Id
```

If [sync](#sync) is enabled, send a
[create](https://github.com/kronos-io/kronos.server#create) request to the
server, in order to notify all other user's clients.

### Read

Retrieves a task by Id. Throws `task not found` and `read task failed`.

```typescript
function read(id: Id): Task
```

### Read All

Retrieves all tasks from database. Throws `read all tasks failed`.

```typescript
function read_all(): Task[]
```

### Update

Updates a task with the partial task given. Throws `task not found` and `update
task failed`.

```typescript
type Partial<T> = {
  [P in keyof T]?: T[P]
}

function update(id: Id, task: Partial<Task>): void
```

If [sync](#sync) is enabled, send a
[update](https://github.com/kronos-io/kronos.server#update) request to the
server, in order to notify all other user's clients.

### Delete

Deletes a task by id. Throws `task not found` and `delete task failed`.

```typescript
function delete(id: Id): void
```

If [sync](#sync) is enabled, send a
[delete](https://github.com/kronos-io/kronos.server#delete) request to the
server, in order to notify all other user's clients.

### Generate Id

Generate a unique id from a list of task.

```typescript
function generate_id(tasks: Task[]): Id
```

Here the algorithm to follow:

```typescript
function generate_id(tasks: Task[]): Id {
  const ids = tasks.map(t => t.id)
  let new_id = 1

  while (true) {
    if (ids.indexOf(new_id) !== -1) {
      return new_id
    }
    
    new_id++
  }
}
```

### Stringify task props

Transforms all properties of a task to string (in order to prepare the task to
be displayed in an [list](#list) or [info](#info) context).

```typescript
enum Context {
  List,
  Info,
}

type StringProps<T> = {
  [P in keyof T]: string;
}

function stringify_props(context: Context, task: Task): StringProps<Task>
```

#### List context

  - **Id**: if task is done, displays `-`, otherwise displays the id.
  - **Desc**: displays desc.
  - **Tags**: displays all tags separated by a space.
  - **Duration**: displays `<value> <unit>`. Displays only one unit, the bigger
    one. For example, for a duration of 3h 45min 10s, displays `4h`. For a
    duration of -3days 14h 11min 10s, displays `3d`. Table of unit: y for
    years, mo for month, w for week, d for day, h for hour, min for minute, s
    for second.
  - **DateTime**: displays a duration between now (the moment when the list is
    displayed) and the datetime (see Duration just above). If duration is
    positive, displays `in <value> <unit>`, otherwise `<value> <unit> ago`.

#### Info context

  - **Id**: displays the id.
  - **Desc**: displays desc.
  - **Tags**: displays all tags separated by a space.
  - **Duration**: displays `<value> <unit>`. Display the full duration. For
    example, for a duration of 3h 45min 10s, displays `3h 45min 10s`. For a
    duration of -3days 14h 11min 10s, displays `3d 14h 11min 10s`. Table of
    unit: y for years, mo for month, w for week, d for day, h for hour, min for
    minute, s for second.
  - **DateTime**: displays the full date at locale format (%c).

## User interface

A client can have a CLI, a GUI, or both.

### Actions
#### Add

Adds a new task. Throws `invalid <property>` and `task add failed`.

```typescript
function add(args: string): void
```

The args match this pattern: `<desc> <tags> <due>`.

A `tag` must start by `+` and should not contain any space. For example:

```typescript
add("+tag +tag-2 +tag_3")
```

A `due` must start by `:` and should contain numbers only.  The full format of
a valid due is `:DDMMYY:HHMM` but almost everything can be omitted. Here some
example to understand better the concept:

  - *\<year\>  means the current year (year when the command is executed)*
  - *\<month\> means the current month (month when the command is executed)*
  - *\<day\>   means the current day (day when the command is executed)*

Full due:

```typescript
add(":100518:1221") // 10th of May 2018, 12h21
```

If minutes omitted, set to `00`:

```typescript
add(":100518:12")   // 10th of May 2018, 12h00
```

If hours omitted, set to `00`:

```typescript
add(":100518")      // 10th of May 2018, 00h00
```

If years omitted, try first the current year. If the final date is exceeded,
try with the next year:

```typescript
add(":1005")        // 10th of May <year> or <year>+1, 00h00
```

If months omitted, try first the current month. If the final date is exceeded,
try with the next month:

```typescript
add(":10")          // 10th of <month> or <month>+1 <year>, 00h00
```

If days omitted, try first the current day. If the final date is exceeded try with the next day:

```typescript
add(":")            // <day> or <day>+1 of <month> <year>, 00h00
add("::8")          // <day> or <day>+1 of <month> <year>, 08h00
```

All together:

```typescript
// Command executed on 1st of March, 2018 at 21h21
add("my awesome task +firstTask :3:18 +awesome")
```

will result in:

```json
{
  "desc": "my awesome task",
  "tags": ["firstTask", "awesome"],
  "due": "3rd of March 2018, 18h00"
}
```

The order is not important, tags can be everywhere, and due as well. The desc
is the remaining of text present after removing tags and due. Both examples end
up with the same result:

```typescript
add("my awesome task +firstTask :3:18 +awesome")
add("my +awesome awesome :3:18 +firstTask task")
```

#### Info

Displays a task by id. The function [`stringify_props`](#info-context) is used
to format the task to show. Throws `task not found` and `task info failed`.

```typescript
function info(id: Id): void
```

#### List

Displays all tasks. The function [`stringify_props`](#list-context) is used to
format all tasks to show. If user option [hide done](#hide-done) is enabled,
does not show up done tasks. Throws `task list failed`.

```typescript
function list(): void
```

#### Update

Updates a task by id. Throws `task not found` and `task update failed`.

```typescript
function update(id: Id, args: string): void
```

Same usage as [Add](#add), except for `tags`. You can remove an existing tag by
prefixing it with a `-`.

For eg., to remove `oldtag` and add `newtag` to task `42`:

```typescript
update(42, "-oldtag +newtag")
```

#### Delete

Deletes a task by id. Throws `task not found` and `task delete failed`.

```typescript
function delete(id: Id): void
```

#### Start

Starts a task by id. Throws `task not found`, `task already started` and `task
start failed`.

```typescript
function start(id: Id): void
```

Also set `active` property to `now` (when this action is triggered).

#### Stop

Stops a task by id. Throws `task not found`, `task already stopped` and `task
stop failed`.

```typescript
function stop(id: Id): void
```

Also update the `worktime` (increase the amount by `now - active`), set
`active` to `0`, and set `last_active` to `now`.

#### Toggle

If task is active, triggers [stop](#stop) action, otherwise trigger
[start](#start) action. Throws `task not found`, `task already started`, `task
already stopped` and `task toggle failed`.

```typescript
function toggle(id: Id): void
```

#### Done

Marks a task as done. Throws `task not found`, `task already done` and `task
done failed`.

```typescript
function done(id: Id): void
```

If the task is active, triggers [stop](#stop) action first. Then sets `done`
property to `now`, and sets `id` to `${id}${now}`. For example, if the id = 5, and
now = 1530716924, then the new id will be `51530716924`.

#### Undone

Unmarks a task as done. Throw `task not found`, `task not done` and `task undone
failed`.

```typescript
function undone(id: Id): void
```

Also sets `done` to `0`, and [generate a new id](#generate-id) for this task.

#### Toggle hide done

Toggles the user configuration [hide done](#hide-done).

```typescript
function toggle_hide_done(): void
```

#### Worktime

Shows the total worktime by tags.

```typescript
function worktime(args: string): void
```

For example, to print the total worktime for tags `tag1` and `tag2`:

```typescript
worktime("tag1 tag2")
```

#### Context

Sets a context by tags. It means that only tasks containing tags in context
will be shown, and when a task is added, these tags will be applied by default.

```typescript
function context(args: string): void
```

For example, to set the context to `project-A`:

```typescript
context("project-A")
```

If you list all tasks, you will see only tasks with `project-A` tag. If you add
a new task, it will automatically get the tag `project-A`.

To clear the context, just call `context` with empty args.

### CLI

The command name is `kronos`, and has a shortcut named `k`. In some specific
case, the command can be `Kronos`, and its shortcut `K`. If a command is
entered without any parameter, then start the GUI (if exists). Otherwise, the
first parameter is the action, and the other parameters are transmitted to the
action. Each action has a shortcut:

| Action | Shortcut | Link to function |
| --- | --- | --- |
| `add` | `a` | [Add](#add) |
| `info` | `i` | [Info](#info) |
| `list` | `l` | [List](#list) |
| `update` | `u` | [Update](#update) |
| `delete` | `del` | [Delete](#delete) |
| `start` | `start` | [Start](#start) |
| `stop` | `stop` | [Stop](#stop) |
| `toggle start/stop` | `s` | [Toggle](#toggle) |
| `done` | `do` | [Done](#done) |
| `undone` | `undo` | [Undone](#undone) |
| `toggle hide done` | `h` | [ToggleHideDone](#toggle-hide-done) |
| `worktime` | `w` | [Worktime](#worktime) |
| `context` | `c` | [Context](#context) |

### GUI

When the GUI mode is started, the [list](#list) action is triggered as main
function. So there is no action `list` in GUI mode. But there is an action
`toggle hide done` to show and hide done tasks in this list. By default, the
first time this `list` is showed up, done tasks are hidden. To change the
default behaviour, check out the user configuration [Hide done](#hide-done).

Actions are triggered by screen events (mouse click, finger touch) or by
keyboard events (shortcuts). The list and the info show data in realtime. If
it's not possible, a `refresh` action is needed, in order to have up-to-date
informations.

| Action | Key mapping | Link to function |
| --- | --- | --- |
| `add` | `<a>` | [Add](#add) |
| `info` | `<i>` | [Info](#info) |
| `update` | `<u>` | [Update](#update) |
| `delete` | `<Backspace>`, `<Del>` | [Delete](#delete) |
| `start` | `<s>` | [Start](#start) |
| `stop` | `<S>` | [Stop](#stop) |
| `toggle start/stop` | `<Enter>` | [Toggle](#toggle) |
| `done` | `<d>` | [Done](#done) |
| `undone` | `<U>` | [Undone](#undone) |
| `toggle hide done` | `<H>` | [ToggleHideDone](#toggle-hide-done) |
| `worktime` | `<W>` | [Worktime](#worktime) |
| `context` | `<C>` | [Context](#context) |
| `refresh` | `<R>` | Refreshes all the GUI (only when there is no realtime showing) |
| `quit` | `<q>`, `<Esc>` | Quits the GUI mode (only if [CLI](#cli) mode exists also) |

## Sync

Tasks can be synchronized with a [Kronos realtime
server](https://github.com/kronos-io/kronos.server) instance. This feature can
be activated or deactivated from [user options](#enable-sync).

### Initialization

When the client starts, contact the server and send a
[login](https://github.com/kronos-io/kronos.server#login) request. `user_id`
and `device_id` can be omitted if it's the first time a client connects to the
server. You will receive a `user_id`, a `device_id` and a `version`.

  - If the locale `version` is older than the server `version`, send a
    [read-all](https://github.com/kronos-io/kronos.server#read-all) request.
    You will receive an up-to-date database that you will have to save into the
    locale database.
  - If the locale `version` is more recent than the server `version`, send a
    [write-all](https://github.com/kronos-io/kronos.server#write-all) request.

### Notifications

Once initialised, your client is connected to the server, and will receive a
notification each time the database changes. See [server
notifications](https://github.com/kronos-io/kronos.server#notifications) to
learn more about how to handle them.
