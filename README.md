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
    * [Create](#create)
    * [Read](#read)
    * [Read All](#read-all)
    * [Update](#update)
    * [Delete](#delete)
  * [User interface](#user-interface)
    * [CLI](#cli)
    * [GUI](#gui)
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

Should read raw data from database and return a list of [Task](#model).

```typescript
function read(): Task[]
```

### Write

Should receive tasks, turn them into raw data, write to database and call [sync](#sync) function with this raw data.

```typescript
function write(Task[]): void
```

### Sync

Should receive raw data from [write](#write) and update the Gist file.

```typescript
function sync(string): void
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

### Helpers
#### Generate Id

Should generate a unique [Id](#id) from a list of Task.

```typescript
function generateId(tasks: Task[]): Id
```

It should follow this algorithm:

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

Should transform all properties of a Task to string (in order to prepare the task to be displayed in an List context).

  - Id: if task is done, should display `-`, otherwise should display the id
  - Desc: should display desc
  - Tags: should display all tags separated by a space
  - Duration: should display `<value> <unit>`. Should display only one unit, the bigger one. For example, for a duration of 3h 45min 10s, should display `in 4h`. For a duration of -3days 14h 11min 10s, should display `3d ago`. Table of unit: y for years, mo for month, w for week, d for day, h for hour, min for minute, s for second.
  - DateTime: should display a duration between now (the moment when the list is displayed) and the datetime (see Duration just above). If duration is positive, should display `in <value> <unit>`, otherwise `<value> <unit> ago`.
  
```typescript
type StringVal<T> = {
  [StringVal in keyof T]: string;
}

function toStringList(task: Task): StringVal<Task>
```

#### Stringify info task

Should transform all properties of a Task to string (in order to prepare the task to be displayed in an Info context).

  - Id: should display the id
  - Desc: should display desc
  - Tags: should display all tags separated by a space
  - Duration: should display `<value> <unit>`. Should display the full duration. For example, for a duration of 3h 45min 10s, should display `3h 45min 10s`. For a duration of -3days 14h 11min 10s, should display `3d 14h 11min 10s`. Table of unit: y for years, mo for month, w for week, d for day, h for hour, min for minute, s for second.
  - DateTime: should display the full date at locale format (%c).

```typescript
type StringVal<T> = {
  [StringVal in keyof T]: string;
}

function toStringInfo(task: Task): StringVal<Task>
```

### Create

```typescript
function create(task: Task): Id
```

### Read

```typescript
function read(id: Id): Task
```

### Read All

```typescript
function readAll(): Task[]
```

### Update

```typescript
function update(id Id, params Subtype<Task>): void
```

### Delete

```typescript
function delete(id: Id): void
```

## User interface
### CLI
### GUI
### Add

```typescript
function add(string): void
```

### Update

```typescript
function update(int, string): void
```

### Delete

```typescript
function delete(int): void
```

### Start

```typescript
function start(int): void
```

### Stop

```typescript
function stop(int): void
```

### Toggle

```typescript
function toggle(int): void
```

### Done

```typescript
function done(int): void
```

### Undone

```typescript
function undone(int): void
```

### Worktime

```typescript
function worktime(int): void
```
