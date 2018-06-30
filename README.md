# Kronos [wip] [help-wanted]

Kronos is a synchronized cross-platform task and time manager. In fact, it's a group of clients which follow a common interface and synchronize data with a common secret [Gist](https://gist.github.com).

## List of available clients

  - [Kronos.vim](https://github.com/kronos-io/kronos.vim)
  
## Roadmap

  - Kronos.web (TypeScript)
  - Kronos.ios (Swift ?)
  - Kronos.android (Java ? Kotlin ?)
  - Kronos.cli (Python ?)
  - Kronos.gui (No idea ... Qt ? Java ? Electron ?)
  - ...
  
## Interface
### Task model

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

### Database API

```typescript
// Tasks are stored in a local database (file),
// one line = one task at JSON format. No last empty line.
// Eg for a 2 tasks database:
// {"desc": "task1", "id": 4, ...}
// {"desc": "task2", "id": 12, ...}

// Should read the database and transform raw data to Task[].
function read(): Task[]

// Should transform tasks to raw data, write raw data to database
// and call postWrite with raw data.
function write(tasks: Task[]): void

// Should sync data with remote database (secret Gist)
// only if the user has enabled this feature.
async function postWrite(rawdata: string): void
```

### Task API

```typescript
function create(task: Task): int

function read(id: int): Task

function readAll(): Task[]

function update(id: int, params: Subtype<Task>): void

function delete(id: int): void
```

### UI API

```typescript
function add(args: string): void

function update(id: int, aprgs: string): void

function delete(id: int): void

function start(id: int): void

function stop(id: int): void

function toggle(id: int): void

function done(id: int): void

function undone(id: int): void

function worktime(id: int): void
```
