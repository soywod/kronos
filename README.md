# Kronos protocol

Kronos is a synchronized cross-platform task and time manager. In fact, it's a group of clients which follow a protocol and synchronize data with a common secret [Gist](https://gist.github.com). This document aims to be this protocol.

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

## Database

Tasks are stored in a local database (file), one line = one task at JSON format.
No last empty line.

```typescript
// Example for a 2 tasks database
{"desc": "task1", "id": 4, ...}
{"desc": "task2", "id": 12, ...}
```

### Read

Read raw data from database and return a list of [Task](#model).

```typescript
function read(): Task[]
```

### Write

Receive tasks, turn them into raw data, write to database and call [sync](#sync) function with this raw data.

```typescript
function write(tasks: Task[]): void
```

### Sync

Receive raw data from [write](#write) and update the Gist file.

```typescript
function sync(data: string): void
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

A client can be a CLI, a GUI, or even both.

### CLI

The command name is `kronos`, and can have a shortcut named `k`. In some specific case, the command can be `Kronos`, and its shortcut `K`. If started without any parameter, then start the GUI (if present). Otherwise, the first parameter is the action, and the other parameters are transmitted to the action. Each action has a shortcut:

| Action | Shortcut | Function |
| --- | --- | --- |
| `add` | `a` | [Add](#add) |
| `info` | `i` | [Info](#info) |
| `update` | `u` | [Update](#update) |
| `delete` | `D` | [Delete](#delete) |
| `start` | `s` | [Start](#start) |
| `stop` | `S` | [Stop](#stop) |
| `toggle` | `t` | [Toggle](#toggle) |
| `done` | `d` | [Done](#done) |
| `undone` | `U` | [Undone](#undone) |
| `worktime` | `w` | [Worktime](#worktime) |

### GUI
### Main functions
#### Add

```typescript
function add(string): void
```

#### Update

```typescript
function update(int, string): void
```

#### Delete

```typescript
function delete(int): void
```

#### Start

```typescript
function start(int): void
```

#### Stop

```typescript
function stop(int): void
```

#### Toggle

```typescript
function toggle(int): void
```

#### Done

```typescript
function done(int): void
```

#### Undone

```typescript
function undone(int): void
```

#### Worktime

```typescript
function worktime(int): void
```
