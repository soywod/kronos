# Kronos protocol

Kronos is a synchronized cross-platform task and time manager. In fact, it's a group of clients which follow this protocol. Feel free to contribute, share some idea, or even code a Kronos client.

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
  * [Task](#task)
    * [Model](#model)
      * [Id](#id)
      * [Desc](#desc)
      * [Tag](#tag)
      * [Duration](#duration)
      * [DateTime](#datetime)
    * [CRUD](#crud)
      * [Create](#create)
      * [Read](#read)
      * [Read All](#read-all)
      * [Update](#update)
      * [Delete](#delete)
    * [Helpers](#helpers)
      * [Generate Id](#generate-id)
      * [Stringify list task](#stringify-list-task)
      * [Stringify info task](#stringify-info-task)
  * [User interface](#user-interface)
    * [CLI](#cli)
    * [GUI](#gui)
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
  * [Configuration](#configuration)
    * [Database](#database-1)
    * [Gist sync](#gist-sync)
    * [Hide done](#hide-done)

## Database

Tasks are stored in a locale database (file), where one line = one task at JSON format.
No last empty line.

```typescript
// Example for a 2 tasks database
{"desc": "task1", "id": 4, ...}
{"desc": "task2", "id": 12, ...}
```

### Read

Read raw data from database and return a list of [Task](#model).
The database path is based on the user configuration [database](#database-1).

```typescript
function read(): Task[]
```

### Write

Transform a list of [Task](#model) to raw data and write them to database. Tasks should be ordered by done (ascending), then by arrival (ascending) before being written to database.

```typescript
function write(tasks: Task[]): void
```

If the user configuration [gist sync](#gist-sync) is enabled, synchronise raw data with the user Gist.

## Task
### Model

```typescript
interface Task {
  id: Id
  desc: Desc
  tags: Tag[]
  active: DateTime
  lastactive: DateTime
  due: DateTime
  done: DateTime
  worktime: Duration
}
```

#### Id

```typescript
type Id = number // An integer > 0
```

#### Desc

```typescript
type Desc = string
```

#### Tag

```typescript
type Tag = string // Match [0-9a-zA-Z\-_]*
```

#### Duration

```typescript
type Duration = number // An integer
```

#### DateTime

```typescript
type DateTime = number // A timestamp
```

### CRUD
#### Create

Receive a task, [generates a unique Id](#generate-id) for this task, then insert into database.

```typescript
function create(task: Task): Id
```

#### Read

Retrieve a task by Id. Throw `task-not-found`.

```typescript
function read(id: Id): Task
```

#### Read All

Retrieve all tasks from database.

```typescript
function readAll(): Task[]
```

#### Update

Update a task with all params received. Throw `task-not-found`.

```typescript
type Partial<T> = {
  [P in keyof T]?: T[P]
}

function update(id Id, task Partial<Task>): void
```

#### Delete

Delete a task by id. Throw `task-not-found`.

```typescript
function delete(id: Id): void
```

### Helpers
#### Generate Id

Generate a unique id from a list of task.

```typescript
function generateId(tasks: Task[]): Id
```

Algorithm to follow:

```typescript
let newid = 1

while (true) {
  if (tasks.map(t => t.id).indexOf(newid) !== -1) {
    return newid
  }
  
  newid++
}
```

#### Stringify list task

Transform all properties of a task to string (in order to prepare the task to be displayed in an [list](#list) context).

  - **Id**: if task is done, display `-`, otherwise display the id
  - **Desc**: display desc
  - **Tags**: display all tags separated by a space
  - **Duration**: display `<value> <unit>`. Display only one unit, the bigger one. For example, for a duration of 3h 45min 10s, display `4h`. For a duration of -3days 14h 11min 10s, display `3d`. Table of unit: y for years, mo for month, w for week, d for day, h for hour, min for minute, s for second.
  - **DateTime**: display a duration between now (the moment when the list is displayed) and the datetime (see Duration just above). If duration is positive, display `in <value> <unit>`, otherwise `<value> <unit> ago`.
  
```typescript
type StringVal<T> = {
  [P in keyof T]: string;
}

function toStringList(task: Task): StringVal<Task>
```

#### Stringify info task

Transform all properties of a task to string (in order to prepare the task to be displayed in an [info](#info) context).

  - **Id**: display the id
  - **Desc**: display desc
  - **Tags**: display all tags separated by a space
  - **Duration**: display `<value> <unit>`. Display the full duration. For example, for a duration of 3h 45min 10s, display `3h 45min 10s`. For a duration of -3days 14h 11min 10s, display `3d 14h 11min 10s`. Table of unit: y for years, mo for month, w for week, d for day, h for hour, min for minute, s for second.
  - **DateTime**: display the full date at locale format (%c).

```typescript
type StringVal<T> = {
  [P in keyof T]: string;
}

function toStringInfo(task: Task): StringVal<Task>
```

## User interface

A client can have a CLI, a GUI, or both.

### CLI

The command name is `kronos`, and has a shortcut named `k`. In some specific case, the command can be `Kronos`, and its shortcut `K`. If command entered without any parameter, then start the GUI (if exists). Otherwise, the first parameter is the action, and the other parameters are transmitted to the action. Each action has a shortcut:

| Action | Shortcut | Link to function |
| --- | --- | --- |
| `add` | `a` | [Add](#add) |
| `info` | `i` | [Info](#info) |
| `list` | `l` | [List](#list) |
| `update` | `u` | [Update](#update) |
| `delete` | `del` | [Delete](#delete) |
| `start` | `+` | [Start](#start) |
| `stop` | `-` | [Stop](#stop) |
| `toggle start/stop` | `s` | [Toggle](#toggle) |
| `done` | `do` | [Done](#done) |
| `undone` | `undo` | [Undone](#undone) |
| `toggle hide done` | `h` | [ToggleHideDone](#toggle-hide-done) |
| `worktime` | `w` | [Worktime](#worktime) |
| `context` | `c` | [Context](#context) |

### GUI

When the GUI mode is started, the [list](#list) action is triggered as main function. So there is no action `list` in GUI mode. But there is an action `toggle hide done` to show and hide done tasks in this list. By default, the first time this `list` is showed up, done tasks are hidden. To change the default behaviour, check out the user configuration [Hide done](#hide-done).

Actions can be triggered by screen events (mouse click, finger touch) or by keyboard events (shortcuts). The list and the info should show data in realtime, otherwise a `refresh` action is needed, in order to have up-to-date informations.

Mappings follow vim mappings:

| Action | Key mappings | Link |
| --- | --- | --- |
| `add` | `<a>` | [Add](#add) |
| `info` | `<K>` | [Info](#info) |
| `update` | `<cc>` | [Update](#update) |
| `delete` | `<dd>`, `<Backspace>`, `<Del>` | [Delete](#delete) |
| `start` | `<+>` | [Start](#start) |
| `stop` | `<->` | [Stop](#stop) |
| `toggle start/stop` | `<cs>` | [Toggle](#toggle) |
| `done` | `<do>` | [Done](#done) |
| `undone` | `<u>` | [Undone](#undone) |
| `toggle hide done` | `<ch>` | [ToggleHideDone](#toggle-hide-done) |
| `worktime` | `<W>` | [Worktime](#worktime) |
| `context` | `<co>` | [Context](#context) |
| `refresh` | `<r>` | Refresh all the GUI (only when there is no realtime showing) |
| `quit` | `<q>`, `<Esc>` | Quit the GUI mode (only if [CLI](#cli) mode exists also) |

### Actions
#### Add

Add a new task.

```typescript
function add(args: string): void
```

The args match this pattern: `<desc> <tags> <due>`.

A `tag` must start by `+` and should not contain any space. For example:

```typescript
add("+tag +tag-2 +tag_3")
```

A `due` must start by `:` and should contain numbers only.  The full format of a valid due is `:DDMMYY:HHMM` but almost everything can be omitted. Here some example to understand better the concept:

  - *\<day\>   means the current day (day when the command is executed)*
  - *\<month\> means the current month*
  - *\<year\>  means the current year*

Full due:

```typescript
add(":100518:1200") // 10th of May 2018, 12h00
```

If minutes omitted, set to `00`:

```typescript
add(":100518:12")   // 10th of May 2018, 12h00
```

If hours omitted, set to `00`:

```typescript
add(":100518")      // 10th of May 2018, 00h00
```

If years omitted, try first the current year. If the final date is exceeded, try with the next year:

```typescript
add(":1005")        // 10th of May <year> or <year>+1, 00h00
```

If months omitted, try first the current month. If the final date is exceeded, try with the next month:

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

The order is not important, tags can be everywhere, and due as well. The desc is the remaining of text present after removing tags and due. Both examples end up with the same result:

```typescript
add("my awesome task +firstTask :3:18 +awesome")
add("my +awesame awesome :3:18 +firstTask task")
```

#### Info

Display a task by id. Throw `task-not-found`.

```typescript
function info(id: number): void
```

#### List

Display all tasks. If user option [hide done](#hide-done) is enabled, do not show up done tasks.

```typescript
function list(): void
```

#### Update

Update a task by id. Throw `task-not-found`.

```typescript
function update(id: number, args: string): void
```

Same usage as [Add](#add), except for `tags`. You can remove an existing tag by prefixing it with a `-`.

For eg., to remove `oldtag` and add `newtag` to task `42`:

```typescript
update(42, "-oldtag +newtag")
```

#### Delete

Remove a task by id. Throw `task-not-found`.

```typescript
function delete(id: number): void
```

#### Start

Start a task by id. Throw `task-not-found` and `task-already-started`.

```typescript
function start(id: number): void
```

Also set `active` property to `now` (when this action is triggered).

#### Stop

Stop a task by id. Throw `task-not-found` and `task-already-stopped`.

```typescript
function stop(id: number): void
```

Also update the `worktime` (increase the amount by `now - active`), set `active` to `0`, and set `lastactive` to `now`.

#### Toggle

If task active, trigger [stop](#stop) action, otherwise trigger [start](#start) action. Throw `task-not-found`.

```typescript
function toggle(id: number): void
```

#### Done

Mark a task as done. Throw `task-not-found` and `task-already-done`.

```typescript
function done(id: number): void
```

If the task is active, trigger [stop](#stop) action first. Then set `done` property to `now`, and set `id` to `id . now`. For example, if the id = 5, and now = 1530716924, then the new id will be 51530716924.

#### Undone

Unmark a task as done. Throw `task-not-found` and `task-not-done`.

```typescript
function undone(id: number): void
```
Also set `done` to `0`, and [generate a new id](#generate-id) for this task.

#### Toggle hide done

Toggle the user configuration [hide done](#hide-done).

```typescript
function toggleHideDone(): void
```

#### Worktime

Show the total worktime by tags.

```typescript
function worktime(args: string): void
```

For example, to print the total worktime for tags `tag1` and `tag2`:

```typescript
worktime("tag1 tag2")
```

#### Context

Set a context by tags. It means that only tasks containing tags in context will be shown,
and when a task is added, these tags will be applied by default.

```typescript
function context(args: string): void
```

For example, to set the context to `project-A`:

```typescript
context("project-A")
```

If you list all tasks, you will see only tasks with `project-A` tag.
If you add a new task, it will automatically get the tag `project-A`.

To clear the context, just call `context` with empty args.

## Configuration

The user is able to configure some options.

### Database

Contain the path to the database file. In some special case, when the database can't be stored as a file, refer to the database key name. Default: `ROOT_APP_FOLDER/kronos.db`.

### Gist sync

Contain a boolean. If `true`, activate the [Gist](https://gist.github.com/) sync. In this case, the GitHub token is prompted each time the application starts, till the user enters a valid token or the user disables this option. Default: `true`.

### Hide done

Contain a boolean. If `true`, when the [list](#list) action is triggered, does not display done tasks. Default: `true`.
