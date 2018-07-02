# Kronos protocol

Kronos is a synchronized cross-platform task and time manager. In fact, it's a group of clients which follow a protocol and synchronize data with a common secret [Gist](https://gist.github.com). This document aims to be this protocol.

## List of available clients

  - [Kronos.vim](https://github.com/kronos-io/kronos.vim)
 
## List of potential clients

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
No last empty line. Eg for a 2 tasks database:

```
{"desc": "task1", "id": 4, ...}
{"desc": "task2", "id": 12, ...}
```

### Read

Should read raw data from database and return a list of [Tasks](#model).

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

```
enum Mode
  List,
  Info,

interface Stringable
  public toString(mode: Mode): string

type Task
  id: Id
  desc: Desc
  tags: Tag[]
  active: DateTime
  lastactive: DateTime
  due: Due
  worktime: Duration
  done: DateTime

type Id extends int implements Stringable
  static generate(tasks: Task[]): Id
  pattern: > 0
  
type Desc extends string implements Stringable

type Tag extends string implements Stringable
  pattern: [a-zA-Z0-9\-_]*

type Due extends DateTime implements Stringable
  static fromString(format: string): Due

type DateTime extends int implements Stringable
  pattern: timestamp
  diff(datetime: DateTime): Duration

type Duration extends int implements Stringable
```

### Create

```typescript
function create(Task): int
```

### Read

```typescript
function read(int): Task
```

### Read All

```typescript
function readAll(): Task[]
```

### Update

```typescript
function update(int, Subtype<Task>): void
```

### Delete

```typescript
function delete(int): void
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
