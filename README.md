# Kronos protocol

Kronos is a synchronized cross-platform task and time manager. In fact, it's a group of clients which follow this protocol. Feel free to contribute, share some idea, or even code a Kronos client.

## List of clients

  - [Kronos.vim](https://github.com/kronos-io/kronos.vim)
  - Kronos.web
  - Kronos.mobile
  - Kronos.cli
  - Kronos.desktop
  - ... 

## Table of contents

  * [Database](#database)
    * [Read](#read)
    * [Write](#write)
    * [Sync](#sync)
  * [Task](#task)
    * [Model](#model)
      * [Id](#id)
      * [Desc](#desc)
      * [Tag](#tag)
      * [Duration](#duration)
      * [DateTime](#datetime)
    * [Main funcitons](#main-functions)
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
    * [Main functions](#main-functions)
      * [Add](#add)
      * [Update](#update)
      * [Delete](#delete)
      * [Start](#start)
      * [Stop](#stop)
      * [Toggle](#toggle)
      * [Done](#done)
      * [Undone](#undone)
      * [Worktime](#worktime)
  * [Configuration](#configuration)
    * [Database](#database-1)
    * [Gist sync](#gist-sync)
    * [Hide done](#hide-done)

## Database

Tasks are stored in a local database (file), one line = one task at JSON format.
No last empty line.

```typescript
// Example for a 2 tasks database
{"desc": "task1", "id": 4, ...}
{"desc": "task2", "id": 12, ...}
```

### Read

Read raw data from database and return a list of [Task](#model). The database path is based on the [database](#database-1) user configuration.

```typescript
function read(): Task[]
```

### Write

Transform a list of [Task](#model) to raw data and write them to database. If the user configuration [gist sync](#gist-sync) is activated, synchronise this raw data with the user Gist.

```typescript
function write(tasks: Task[]): void
```

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

Should be an `integer` > 0.

```typescript
type Id = number
```

#### Desc

Should be a `string`.

```typescript
type Desc = string
```

#### Tag

Should be a `string` matching `[0-9a-zA-Z\-_]*`.

```typescript
type Tag = string
```

#### Duration

Should be an `integer`.

```typescript
type Duration = number
```

#### DateTime

Should be a `timestamp`.

```typescript
type Duration = number
```

### Main functions
#### Create

Receive a [Task](#model), [generate a unique Id](#generate-id) for this task, then insert it into database.

```typescript
function create(task: Task): Id
```

#### Read

Retrieve a [Task](#model) by Id. Throw `task-not-found` if task not found.

```typescript
function read(id: Id): Task
```

#### Read All

Retrieve all [Tasks](#model) from database.

```typescript
function readAll(): Task[]
```

#### Update

Update a [Task](#model) with all params received. Throw `task-not-found` if [Task](#model) not found. 

```typescript
type Partial<T> = {
  [P in keyof T]?: T[P]
}

function update(id Id, task Partial<Task>): void
```

#### Delete

Delete a [Task](#model) by id. Throw `task-not-found` if [Task](#model) not found.

```typescript
function delete(id: Id): void
```

### Helpers
#### Generate Id

Generate a unique [Id](#id) from a list of [Task](#model).

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

Transform all properties of a [Task](#model) to string (in order to prepare the [Task](#model) to be displayed in an List context).

  - Id: if task is done, display `-`, otherwise display the id
  - Desc: display desc
  - Tags: display all tags separated by a space
  - Duration: display `<value> <unit>`. Display only one unit, the bigger one. For example, for a duration of 3h 45min 10s, display `in 4h`. For a duration of -3days 14h 11min 10s, display `3d ago`. Table of unit: y for years, mo for month, w for week, d for day, h for hour, min for minute, s for second.
  - DateTime: display a duration between now (the moment when the list is displayed) and the datetime (see Duration just above). If duration is positive, display `in <value> <unit>`, otherwise `<value> <unit> ago`.
  
```typescript
type StringVal<T> = {
  [P in keyof T]: string;
}

function toStringList(task: Task): StringVal<Task>
```

#### Stringify info task

Should transform all properties of a [Task](#model) to string (in order to prepare the [Task](#model) to be displayed in an Info context).

  - Id: display the id
  - Desc: display desc
  - Tags: display all tags separated by a space
  - Duration: display `<value> <unit>`. Display the full duration. For example, for a duration of 3h 45min 10s, display `3h 45min 10s`. For a duration of -3days 14h 11min 10s, display `3d 14h 11min 10s`. Table of unit: y for years, mo for month, w for week, d for day, h for hour, min for minute, s for second.
  - DateTime: display the full date at locale format (%c).

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
| `delete` | `D` | [Delete](#delete) |
| `start` | `s` | [Start](#start) |
| `stop` | `S` | [Stop](#stop) |
| `toggle` | `t` | [Toggle](#toggle) |
| `done` | `d` | [Done](#done) |
| `undone` | `U` | [Undone](#undone) |
| `toggle hide done` | `<H>` | [ToggleHideDone](#toggle-hide-done) |
| `worktime` | `w` | [Worktime](#worktime) |

### GUI

When the GUI mode is started, the [list](#list) action is triggered as main function. So there is no action `list` in GUI mode. But there is an action `toggle hide done` to show and hide done tasks in this list. By default, the first time this `list` is showed up, done tasks are hidden. To change the default behaviour, check out the user configuration [Hide done](#hide-done).

Actions can be triggered by screen events (mouse click, finger touch) or by keyboard events (shortcuts). The list and the info should show data in realtime, otherwise a `refresh` action need to be implemented, in order to refresh manually the interface.

| Action | Key mappings | Link |
| --- | --- | --- |
| `add` | `<a>` | [Add](#add) |
| `info` | `<i>` | [Info](#info) |
| `update` | `<u>` | [Update](#update) |
| `delete` | `<D>`, `<Backspace>`, `<Del>` | [Delete](#delete) |
| `start` | `<s>` | [Start](#start) |
| `stop` | `<S>` | [Stop](#stop) |
| `toggle` | `<t>`, `<Enter>` | [Toggle](#toggle) |
| `done` | `<d>` | [Done](#done) |
| `undone` | `<U>` | [Undone](#undone) |
| `toggle hide done` | `<H>` | [ToggleHideDone](#toggle-hide-done) |
| `worktime` | `<w>` | [Worktime](#worktime) |
| `refresh` | `<r>` | Refresh all the GUI (only when there is no realtime showing) |
| `quit` | `<q>`, `<Esc>` | Quit the GUI mode (only if [CLI](#cli) mode exists also) |

### Actions
#### Add

Add a new task.

```typescript
function add(args: string): void
```

The args should match this pattern: `<desc> <tags> <due>`.

A **tag** must start by `+` and should not contain any space. For example:

```typescript
add("+tag +tag-2 +tag_3")
```

A **due** must start by `:` and should contain numbers only.  The full format of a valid due is `:DDMMYY:HHMM` but almost everything can be omitted. Here some example to understand better the concept:

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
  "due": "3rd of March 2018, 18h00",
  ...
}
```

The order is not important, tags can be everywhere, and due as well. The desc is the remaining of text present after removing tags and due. Both examples end up with the same result:

```typescript
add("my awesome task +firstTask :3:18 +awesome")
add("my +awesame awesome :3:18 +firstTask task")
```

#### Update

Update a task by id.

```typescript
function update(id: int, args: string): void
```

Same usage as [Add](#add), except for **tags**. You can remove an existing tag by prefixing it with a `-`.

For eg., to remove **oldtag** and add **newtag** to task **42**:

```typescript
update(42, "-oldtag +newtag")
```

#### Delete

Remove a task by id.

```typescript
function delete(id: int): void
```

#### Start

Start a task by id.
```typescript
function start(id: int): void
```

#### Stop

```typescript
function stop(id: int): void
```

#### Toggle

```typescript
function toggle(id: int): void
```

#### Done

```typescript
function done(id: int): void
```

#### Undone

```typescript
function undone(id: int): void
```


#### Toggle hide done

```typescript
function toggleHideDone(): void
```

#### Worktime

```typescript
function worktime(int): void
```

## Configuration
### Database
### Gist sync
### Hide done
