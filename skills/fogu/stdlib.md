# Fogu standard library

Generated from `docs/tools/reference/*.py` by `docs/tools/gen_reference.py` — do not edit.
Outputs marked `output` were produced by running the example. Signatures read
`call(args) → result — notes`; `value · error` means the call returns an `error`
value on operational failures (use `?` or `const [v, err] = …`).

- [Overview](#overview)
- [Global functions](#global-functions)
- [Strings, lists and maps](#strings-lists-and-maps)
- [The Fogu object](#the-fogu-object)
- [fogu:math](#fogumath)
- [fogu:json](#fogujson)
- [fogu:time](#fogutime)
- [fogu:system](#fogusystem)
- [fogu:fs](#fogufs)
- [fogu:path](#fogupath)
- [fogu:process](#foguprocess)
- [fogu:sync](#fogusync)
- [fogu:test](#fogutest)
- [fogu:log](#fogulog)
- [fogu:regex](#foguregex)
- [fogu:crypto](#fogucrypto)
- [fogu:csv](#fogucsv)
- [fogu:excel](#foguexcel)
- [fogu:yaml](#foguyaml)
- [fogu:toml](#fogutoml)
- [fogu:xml](#foguxml)
- [fogu:kv](#fogukv)
- [fogu:sql](#fogusql)
- [fogu:orm](#foguorm)
- [fogu:redis](#foguredis)
- [fogu:s3](#fogus3)
- [fogu:scylla](#foguscylla)
- [fogu:http](#foguhttp)
- [fogu:web](#foguweb)
- [fogu:jwt](#fogujwt)
- [fogu:websocket](#foguwebsocket)
- [fogu:net](#fogunet)
- [fogu:events](#foguevents)
- [fogu:cron](#fogucron)
- [fogu:nats](#fogunats)
- [fogu:rabbitmq](#fogurabbitmq)
- [fogu:mqtt](#fogumqtt)
- [fogu:fogumq](#fogufogumq)
- [fogu:bullmq](#fogubullmq)
- [fogu:mail](#fogumail)
- [fogu:ai](#foguai)
- [fogu:ssh](#fogussh)
- [fogu:shell](#fogushell)
- [fogu:ffi](#foguffi)
- [fogu:playbook](#foguplaybook)
- [fogu:vm](#foguvm)
- [fogu:cli](#fogucli)

## Overview

Modules are imported with `import name from "fogu:name"` (or by name: `import { parse } from "fogu:json"`). Each section below lists every function with its parameters and return value, and shows an example with its **real output** (the examples are run when this page is generated). Core [`fogu:math`](#fogumath) [`fogu:json`](#fogujson) [`fogu:time`](#fogutime) [`fogu:system`](#fogusystem) [`fogu:fs`](#fogufs) [`fogu:path`](#fogupath) [`fogu:process`](#foguprocess) [`fogu:sync`](#fogusync) [`fogu:test`](#fogutest) [`fogu:log`](#fogulog) [`fogu:regex`](#foguregex) [`fogu:crypto`](#fogucrypto)
Data formats [`fogu:csv`](#fogucsv) [`fogu:excel`](#foguexcel) [`fogu:yaml`](#foguyaml) [`fogu:toml`](#fogutoml) [`fogu:xml`](#foguxml)
Storage [`fogu:kv`](#fogukv) [`fogu:sql`](#fogusql) [`fogu:orm`](#foguorm) [`fogu:redis`](#foguredis) [`fogu:s3`](#fogus3) [`fogu:scylla`](#foguscylla)
Web & network [`fogu:http`](#foguhttp) [`fogu:web`](#foguweb) [`fogu:jwt`](#fogujwt) [`fogu:websocket`](#foguwebsocket) [`fogu:net`](#fogunet)
Messaging & events [`fogu:nats`](#fogunats) [`fogu:rabbitmq`](#fogurabbitmq) [`fogu:mqtt`](#fogumqtt) [`fogu:fogumq`](#fogufogumq) [`fogu:events`](#foguevents) [`fogu:cron`](#fogucron)
Integrations [`fogu:mail`](#fogumail) [`fogu:ai`](#foguai) [`fogu:ssh`](#fogussh) [`fogu:shell`](#fogushell) [`fogu:ffi`](#foguffi) [`fogu:playbook`](#foguplaybook)

### Conventions

**Errors are values.** Operational failures — a missing file, a refused connection, a server error, malformed input — come back as an `error` value: handle them with `?` (propagate) or `const [value, err] = f()`. Programming mistakes — a wrong argument type, an unknown option — stop the program with a message. **Options** are passed as a last map argument: `f(x, { timeout: 5s })`. **Durations** accept a duration literal (`500ms`, `2s`, `1h`) or an int number of milliseconds. Network operations wait cooperatively: other `fg` tasks keep running while one waits.

```fogu
import json from "fogu:json"

const [cfg, err] = json.parse("{ broken")      // an error value, not a crash
if err != nil {
    print("invalid config:", err.message)
}

fn load(text: str): error {
    const data = json.parse(text)?              // ? returns the error to the caller
    print("name:", data.name)
    return nil
}
print(load("{\"name\": \"fogu\"}"), load("nope") != nil)
```

output:

```text
invalid config: json.parse: unterminated JSON string
name: fogu
nil true
```

## Global functions

Available everywhere, without an import.

- `print(...values)` → nil — Writes the values separated by spaces and a newline (`Fogu.write` writes without them).
- `len(x)` → int — Length of a `str` (characters), `list`, `map` or `chan` (queued items).
- `typeof(x)` → str — `int float str bool nil list map fn duration chan error struct …`
- `str(x)` → str — Text form of any value (the same text `print` shows).
- `int(x)` → int — From a float (truncates), a numeric str or a bool. Invalid text stops the program.
- `float(x)` → float — From an int or a numeric str.
- `clock()` → float — Seconds since the UNIX epoch, with fractions — use differences to measure time.
- `error(message, cause?)` → error — A new error value; `err.message`, `err.cause`.
- `panic(...values)` → — — Stops the program with a message (for impossible situations).
- `timeout(d)` → chan — A channel that receives once after `d` (use with `select`).
- `close(ch)` → nil — Closes a channel: receivers get `nil` once it is empty.

```fogu
print(len("olá"), len([1, 2, 3]), len({ a: 1 }))
print(typeof(1), typeof(1.5), typeof("x"), typeof(nil), typeof([]), typeof({}), typeof(2s))
print(str(42) + "!", int("17") + 1, int(3.9), float("2.5") * 2)
const e = error("disk full")
print(typeof(e), e.message)
const t0 = clock()
print(clock() - t0 < 1)
```

output:

```text
3 3 1
int float str nil list map duration
42! 18 3 5.0
error disk full
true
```

## Strings, lists and maps

Methods of the built-in values. Indexes start at 0; negative indexes count from the end in `slice`.

### str

- `s.len()` → int — Number of characters.
- `s.upper() · s.lower()` → str — Case conversion.
- `s.trim()` → str — Without leading and trailing whitespace.
- `s.split(sep)` → list — Parts between `sep`.
- `s.lines() · s.chars()` → list — Lines (without line endings) · single characters.
- `s.contains(t) · s.starts_with(t) · s.ends_with(t)` → bool — Substring tests.
- `s.index_of(t)` → int — Character index of the first `t`, or `-1`.
- `s.replace(old, new)` → str — Every `old` replaced.
- `s.repeat(n)` → str — `s` repeated `n` times.
- `s.slice(start, end?)` → str — Characters from `start` up to (not including) `end`.

```fogu
const s = "  Fogu, fast & small  "
print(s.trim().upper(), s.trim().split(", "))
print("a-b-c".replace("-", "+"), "ab".repeat(3), "olá mundo".slice(4))
print("main.fg".ends_with(".fg"), "fogu".index_of("g"), "x\ny".lines(), "abc".chars())
print(`template: ${1 + 1} items`)
```

output:

```text
FOGU, FAST & SMALL [Fogu, fast & small]
a+b+c ababab mundo
true 2 [x, y] [a, b, c]
template: 2 items
```

### list

- `l.push(x) · l.pop()` → nil · value — Add to / remove from the end.
- `l.unshift(x) · l.shift()` → nil · value — Add to / remove from the start.
- `l.insert(i, x) · l.remove(i)` → nil · value — At an index.
- `l.first() · l.last()` → value — `nil` when empty.
- `l.contains(x) · l.index_of(x)` → bool · int — Search by value.
- `l.join(sep)` → str — Items as text joined with `sep`.
- `l.reverse() · l.sort(fn?)` → list — New list; `sort` takes an optional comparator `(a, b) => int`.
- `l.slice(start, end?) · l.concat(other)` → list — Copies.
- `l.map(fn) · l.filter(fn) · l.find(fn)` → list · list · value — Functional helpers.
- `l.any(fn) · l.all(fn) · l.each(fn)` → bool · bool · nil
- `l.reduce(fn, initial)` → value — `fn(acc, item)` over the items.
- `l.clear() · l.len()` → nil · int

```fogu
const nums = [5, 3, 8, 1]
nums.push(10)
print(nums, nums.sort(), nums.sort((a, b) => b - a))
print(nums.map((n) => n * 2), nums.filter((n) => n > 4), nums.find((n) => n > 6))
print(nums.reduce((acc, n) => acc + n, 0), nums.any((n) => n > 9), nums.all((n) => n > 0))
print(nums.slice(1, 3), nums.first(), nums.last(), nums.join(" | "))
```

output:

```text
[5, 3, 8, 1, 10] [1, 3, 5, 8, 10] [10, 8, 5, 3, 1]
[10, 6, 16, 2, 20] [5, 8, 10] 8
27 true true
[3, 8] 5 10 5 | 3 | 8 | 1 | 10
```

### map

- `m.keys() · m.values() · m.entries()` → list — Entries are `[key, value]` pairs.
- `m.has(key)` → bool
- `m.get(key, default?)` → value — The value, or `default` (nil).
- `m.set(key, value) · m[key] = value` → nil
- `m.remove(key)` → value — Removes and returns the value.
- `m.merge(other)` → map — A new map; `other` wins on equal keys.
- `m.len() · m.clear()` → int · nil

```fogu
const user = { name: "Ana", age: 30 }
user.city = "Lisboa"
print(user.keys(), user.has("age"), user.get("email", "none"))
print(user.merge({ age: 31, admin: true }))
for entry in user.entries() { print(entry[0], "=", entry[1]) }
```

output:

```text
[age, city, name] true none
{admin: true, age: 31, city: Lisboa, name: Ana}
age = 30
city = Lisboa
name = Ana
```

## The Fogu object

Runtime helpers available everywhere, without an import.

- `Fogu.version · Fogu.os` → str — Fogu's version · `windows`, `linux`, `macos`…
- `Fogu.env` → map — Environment variables.
- `Fogu.args` → list — Arguments after the script name (`fogu run app.fg a b`).
- `Fogu.file(path)` → File — `text() lines() json() write(s) append(s) exists() size() remove()`, fields `path name ext dir`.
- `Fogu.asset(path)` → str — A project file: relative to the project root — and inside `fogu build --compile` executables, the extracted embedded files.
- `Fogu.run(cmd, args?, opts?)` → map — Runs a program → `{ ok, code, stdout, stderr }` (see [fogu:process](#foguprocess)).
- `Fogu.glob(pattern)` → list — Files matching `*`, `?` and `**`.
- `Fogu.hash(data, algo?)` → str — Hex digest, `sha256` (default) or `sha512`.
- `Fogu.sql(url)` → Database — Same as `sql.open(url)`.
- `Fogu.sleep(d)` → nil — Waits (other tasks keep running).
- `Fogu.write(...values)` → nil — Like `print`, without spaces or newline (streaming output).
- `Fogu.watch(path | [paths], { recursive = true, debounce = 50ms })` → Watcher · error — File system events: `w.next(timeout?)` → `{ kind, path, paths }` (`create modify remove rename`), `tryNext()`, `close()`.
- `Fogu.exit(code?)` → — — Ends the program.

```fogu
Fogu.file("notes.txt").write("one\ntwo")?
const f = Fogu.file("notes.txt")
print(f.name, f.ext, f.exists(), f.size(), f.lines()?)
print(Fogu.hash("abc"))
print(typeof(Fogu.env), typeof(Fogu.args), Fogu.asset("data/logo.png").ends_with("data/logo.png"))
Fogu.write("no newline: ", 1, 2)
print()
```

output:

```text
notes.txt .txt true 7 [one, two]
ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad
map list true
no newline: 12
```

Watching files — rebuild on every change:

```fogu
const w = Fogu.watch("src")?
while true {
    const ev = w.next()?
    print(ev.kind, ev.path)
    Fogu.run("fogu", ["test"])
}
```

output:

```text
modify src/app.fg
create src/new.fg
remove src/old.fg
```

## fogu:math

```fogu
import math from "fogu:math"
```

- `math.PI · math.E` → float — Constants.
- `math.abs(x)` → number — Absolute value (int stays int).
- `math.sqrt(x)` → float — Square root.
- `math.pow(base, exp)` → number — Power.
- `math.floor(x) · math.ceil(x)` → int — Rounding down / up.

```fogu
import math from "fogu:math"
print(math.abs(-4), math.sqrt(16), math.pow(2, 10), math.floor(2.7), math.ceil(2.1))
print(math.PI)
```

output:

```text
4 4.0 1024.0 2 3
3.141592653589793
```

## fogu:json

```fogu
import json from "fogu:json"
```

- `json.parse(text)` → value · error — JSON text → maps, lists, str, int, float, bool, nil. Invalid JSON is an error value.
- `json.stringify(value)` → str — Compact JSON (map keys in order).

```fogu
import json from "fogu:json"
const data = json.parse("{\"name\": \"Ana\", \"tags\": [\"a\", \"b\"], \"age\": 30}")?
print(data.name, data.tags, typeof(data.age))
print(json.stringify({ ok: true, list: [1, 2.5, nil] }))
```

output:

```text
Ana [a, b] int
{"list":[1,2.5,null],"ok":true}
```

## fogu:time

```fogu
import time from "fogu:time"
```

- `time.now()` → int — Milliseconds since the UNIX epoch.
- `time.Date()` → Date — The current instant (UTC).
- `time.delay(d) · time.sleep(seconds)` → nil — Waits; other `fg` tasks keep running.

### Date

- `d.year · d.month · d.day` → int — Month 1–12.
- `d.hour · d.minute · d.second · d.millis · d.weekday` → int — Weekday 0 = Sunday.
- `d.getTime()` → int — Milliseconds since the epoch (also `d.ms`).
- `d.toISOString()` → str — `2026-10-10T12:00:00.000Z`
- `d.toString()` → str — Human readable.
- `d.addMs(ms)` → nil — Moves the date by `ms` (in place).
- `d.diff(other)` → int — Milliseconds between the dates.
- `getFullYear() getMonth() getDate() getDay() getHours() getMinutes() getSeconds() getMilliseconds()` → int — JavaScript-style getters (month 0–11).

```fogu
import time from "fogu:time"
const start = time.now()
time.delay(20ms)
print(time.now() - start >= 20)
const d = time.Date()
print(d.year > 2000, d.month >= 1, d.toISOString().ends_with("Z"))
const later = time.Date()
later.addMs(24 * 3600 * 1000)
print(later.diff(d) >= 86400000)
```

output:

```text
true
true true true
true
```

## fogu:system

```fogu
import system from "fogu:system"
```

- `system.args` → list — Script arguments (`fogu run app.fg a b` → `[a, b]`).
- `system.time()` → float — Seconds since the epoch.
- `system.exit(code)` → — — Ends the program.
- `system.info.os · .arch · .family` → str — `windows`/`linux`/`macos` · `x86_64`/`aarch64` · `windows`/`unix`.
- `system.info.cores · .pid` → int — Logical CPUs · process id.
- `system.info.env(name)` → str · nil — An environment variable.
- `system.info.sleep_ms(ms)` → nil — Blocks the whole program (prefer `time.delay`).

```fogu
import system from "fogu:system"
const info = system.info
print(typeof(info.os), info.cores > 0, typeof(info.pid), system.args)
print(info.env("FOGU_DOES_NOT_EXIST"), system.time() > 0)
```

output:

```text
str true int []
nil true
```

## fogu:fs

```fogu
import fs from "fogu:fs"
```

Files and folders. Failures (missing file, permission) are error values.

- `fs.readText(path)` → str · error — Whole file as UTF-8 text.
- `fs.readLines(path)` → list · error — Lines without line endings.
- `fs.writeText(path, text)` → nil · error — Creates or replaces.
- `fs.append(path, text)` → nil · error — Adds to the end (creates the file).
- `fs.exists(path) · fs.isFile(path) · fs.isDir(path)` → bool
- `fs.mkdir(path)` → nil · error — Creates the folder and its parents.
- `fs.list(path)` → list · error — Names inside a folder, sorted.
- `fs.stat(path)` → map · error — `{ size, isFile, isDir, modified }` (modified: ms since the epoch).
- `fs.copy(from, to) · fs.move(from, to)` → nil · error
- `fs.remove(path)` → nil · error — A file or an empty folder.
- `fs.removeAll(path)` → nil · error — A folder and everything inside.
- `fs.glob(pattern)` → list — Paths matching `*`, `?`, `**`, sorted.
- `fs.cwd() · fs.tempDir()` → str — Working folder · system temp folder.

```fogu
import fs from "fogu:fs"
fs.mkdir("data/logs")?
fs.writeText("data/a.txt", "hello")?
fs.append("data/a.txt", "\nworld")?
print(fs.readLines("data/a.txt")?, fs.stat("data/a.txt")?.size)
fs.copy("data/a.txt", "data/logs/b.txt")?
print(fs.list("data")?, fs.glob("data/**/*.txt"))
const [_, err] = fs.readText("missing.txt")
print(err != nil)
fs.removeAll("data")?
print(fs.exists("data"))
```

output:

```text
[hello, world] 11
[a.txt, logs] [data/a.txt, data/logs/b.txt]
true
false
```

## fogu:path

```fogu
import path from "fogu:path"
```

- `path.join(...parts)` → str — Joined with `/` and normalized.
- `path.dirname(p) · path.basename(p)` → str — Folder · last component.
- `path.stem(p) · path.extname(p)` → str — Name without extension · extension with the dot.
- `path.normalize(p)` → str — Resolves `.` and `..`.
- `path.isAbsolute(p)` → bool
- `path.absolute(p)` → str — Absolute path from the working folder.
- `path.sep` → str — The system separator.

```fogu
import path from "fogu:path"
print(path.join("src", "lib", "../main.fg"))
print(path.dirname("a/b/c.txt"), path.basename("a/b/c.txt"), path.stem("a/b/c.tar.gz"), path.extname("a/b/c.txt"))
print(path.normalize("a/./b/../c"), path.isAbsolute("/etc"), path.isAbsolute("etc"))
```

output:

```text
src/main.fg
a/b c.txt c.tar .txt
a/c true false
```

## fogu:process

```fogu
import process from "fogu:process"
```

A program that exits with a non-zero code is not an error: check `ok` and `code`. Not being able to start it is an error value. For pipes and Linux commands see [fogu:shell](#fogushell).

- `process.run(cmd, args?, { cwd, env, input })` → map · error — `{ ok, code, stdout, stderr }`.
- `process.shell(command)` → map — Through `sh -c` / `cmd /C`.
- `process.env(name) · process.setEnv(name, value)` → str · nil
- `process.cwd() · process.chdir(path)` → str · nil
- `process.pid · process.os · process.arch · process.args`
- `process.exit(code)` → —

```fogu
import process from "fogu:process"
const r = process.run("git", ["--version"])?
print(r.ok, r.code, r.stdout.starts_with("git version"))
const [_, err] = process.run("no-such-program-xyz")
print(err != nil)
```

output:

```text
true 0 true
true
```

## fogu:sync

```fogu
import sync from "fogu:sync"
```

Coordination of `fg` tasks. Waiting operations yield to the other tasks; waiting forever is reported as a deadlock.

- `sync.Mutex()` → Mutex — `lock()` `unlock()` `tryLock()` → bool, field `locked`.
- `sync.WaitGroup()` → WaitGroup — `add(n = 1)` `done()` `wait()`, field `count`.
- `sync.Once()` → Once — `do(fn)` runs `fn` only the first time.
- `sync.Atomic(value)` → Atomic — `get()` `set(v)` `add(n = 1)` → new value, `compareAndSwap(old, new)` → bool.

```fogu
import sync from "fogu:sync"
const wg = sync.WaitGroup()
const total = sync.Atomic(0)
const once = sync.Once()
for i in 1..5 {
    wg.add(1)
    fg fn() {
        once.do(() => print("first task"))
        total.add(i)
        wg.done()
    }()
}
wg.wait()
print(total.get())
```

output:

```text
first task
10
```

## fogu:test

```fogu
import { test, skip, expect } from "fogu:test"
```

Used in `*.test.fg` files, run with `fogu test`. A test fails when an expectation fails, when it raises an error or returns an error value.

- `test(name, fn)` → nil — Declares a test.
- `skip(name, fn)` → nil — A test that is reported but not run.
- `expect(value)` → Expectation — Then a matcher; `.not` negates it.
- `toBe(x) · toEqual(x)` — Equality (`toEqual` compares lists and maps deeply).
- `toBeTruthy() · toBeFalsy() · toBeNil() · toBeError()`
- `toBeType("int") · toContain(x) · toHaveLength(n)`
- `toBeGreaterThan(n) · toBeLessThan(n) · toBeCloseTo(x, eps?)`

```fogu
import { test, expect } from "fogu:test"

test("math", () => {
    expect(1 + 2).toBe(3)
    expect([1, 2]).toContain(2)
    expect("abc").not.toHaveLength(2)
    expect(0.1 + 0.2).toBeCloseTo(0.3)
})
```

fogu test:

```text
$ fogu test
math.test.fg
  ✓ math (0ms)

1 passed — 1 file(s) [1ms]
```

## fogu:log

```fogu
import log from "fogu:log"
```

Structured logs to stderr (default), stdout or a file. The level comes from `FOGU_LOG` (default `info`).

- `log.trace/debug/info/warn/error(message, fields?)` → nil — `fields`: a map added as `key=value` (or JSON keys).
- `log.setLevel(level)` → nil — `trace debug info warn error off`.
- `log.level()` → str — Current level.
- `log.setFormat("text" | "json")` → nil — One line per entry.
- `log.setOutput("stderr" | "stdout" | path)` → nil · error — A file is appended to.
- `log.setTime(bool)` → nil — Show the time stamp (default true).
- `log.child(fields)` → Logger — A logger that adds `fields` to every line; same methods + `child`.

```fogu
import log from "fogu:log"
log.setOutput("stdout")
log.setTime(false)
log.info("server started", { port: 3000 })
log.debug("not shown at the info level")
const db = log.child({ module: "db" })
db.warn("slow query", { ms: 812, sql: "select *" })
log.setFormat("json")
log.error("failed", { code: 23 })
```

output:

```text
INFO  server started port=3000
WARN  slow query module=db ms=812 sql="select *"
{"level":"error","msg":"failed","code":23}
```

## fogu:regex

```fogu
import regex from "fogu:regex"
```

Rust `regex` syntax (linear time, no backtracking). Positions are character indexes. In Fogu strings write `\\d` for `\d`. An invalid pattern is an error value.

- `regex.compile(pattern, { ignoreCase, multiline, dotAll, verbose })` → Regex · error
- `re.test(text)` → bool — Matches somewhere.
- `re.matches(text)` → bool — The whole text matches.
- `re.find(text)` → map · nil — `{ text, start, end, groups, named }`.
- `re.findAll(text)` → list — Every match.
- `re.replace(text, repl) · re.replaceAll(text, repl)` → str — `$1`, `${name}`, `$$` in `repl`.
- `re.split(text, limit?)` → list
- `re.source · re.groups · re.names` — Pattern · group count · group names.
- `regex.test/find/findAll/replace/replaceAll/split/matches(text, pattern, …)` — One-off shortcuts.
- `regex.escape(text)` → str — A literal pattern.

```fogu
import regex from "fogu:regex"
const email = regex.compile("(?P<user>[\\w.]+)@(?P<host>[\\w.]+)")?
print(email.test("write to ana@fogu.dev"))
const m = email.find("contact: ana@fogu.dev")
print(m.text, m.start, m.named.host, m.groups)
print(email.replaceAll("a@x.io, b@y.io", "${user} (at ${host})"))
print(regex.split("one, two;three", "\\s*[,;]\\s*"))
print(regex.findAll("v1.2 and v3.10", "\\d+\\.\\d+").map((x) => x.text))
```

output:

```text
true
ana@fogu.dev 9 fogu.dev [ana, fogu.dev]
a (at x.io), b (at y.io)
[one, two, three]
[1.2, 3.10]
```

## fogu:crypto

```fogu
import crypto from "fogu:crypto"
```

- `crypto.hash(algo, data, { base64 })` → str — `md5 sha1 sha256 sha384 sha512`; hex by default.
- `crypto.hmac(algo, key, data, { base64 })` → str
- `crypto.encrypt(data, key)` → str — AES-256-GCM; `key` is a password (argon2id) or 64 hex chars. Base64 output with salt and nonce.
- `crypto.decrypt(box, key, { bytes })` → str · error — Wrong key or altered data → error value.
- `crypto.hashPassword(password)` → str — argon2id PHC string, for storing passwords.
- `crypto.verifyPassword(password, hash)` → bool
- `crypto.randomBytes(n) · crypto.randomHex(n)` → list · str — Secure random.
- `crypto.randomInt(min, max)` → int — Inclusive, unbiased.
- `crypto.uuid()` → str — Random UUID v4.
- `crypto.base64(data, { url }) · crypto.fromBase64(text, { bytes })` → str — `data`: str or list of bytes.
- `crypto.hex(data) · crypto.fromHex(text, { bytes })` → str
- `crypto.equal(a, b)` → bool — Constant-time comparison.

```fogu
import crypto from "fogu:crypto"
print(crypto.hash("sha256", "abc"))
print(crypto.hmac("sha256", "key", "The quick brown fox jumps over the lazy dog"))
const box = crypto.encrypt("card 4242", "my secret")
print(crypto.decrypt(box, "my secret")?)
const [_, err] = crypto.decrypt(box, "wrong")
print(err)
const h = crypto.hashPassword("hunter2")
print(crypto.verifyPassword("hunter2", h), crypto.verifyPassword("hunter3", h))
print(crypto.base64("olá"), crypto.fromBase64("b2zDoQ=="), crypto.hex("hi"), len(crypto.uuid()))
```

output:

```text
ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad
f7bc83f430538424b13298e6aa6fb143ef4d59a14946175997479dbc2d1a3cd8
card 4242
error: crypto.decrypt: wrong key or corrupted data
true false
b2zDoQ== olá 6869 36
```

## fogu:csv

```fogu
import csv from "fogu:csv"
```

Rows with a header become maps; cells are typed (`42` → int, `3.5` → float, `true` → bool, empty → nil; codes like `007` stay text). The delimiter is guessed (`, ; tab |`).

- `csv.parse(text, { header = true, delimiter, types = true, trim })` → list · error — Maps (or lists without header).
- `csv.stringify(rows, { header = true, delimiter = ",", columns })` → str — Rows: maps (a header line is written) or lists.
- `csv.read(path, options?) · csv.write(path, rows, options?)` → list · nil — The same, from/to a file.

```fogu
import csv from "fogu:csv"
const people = csv.parse("name;age;code;city\nAna;30;007;Lisboa\nBob;25.5;12;\n")?
print(people[0])
print(people[1].age, people[1].city)
print(csv.stringify([{ name: "Ana", note: "says \"hi\", ok" }, { name: "Bob", age: 3 }]))
csv.write("out.csv", people, { columns: ["name", "age"] })?
print(csv.read("out.csv")?)
```

output:

```text
{age: 30, city: Lisboa, code: 007, name: Ana}
25.5 nil
name,note,age
Ana,"says ""hi"", ok",
Bob,,3

[{age: 30, name: Ana}, {age: 25.5, name: Bob}]
```

## fogu:excel

```fogu
import excel from "fogu:excel"
```

Reads `.xlsx .xlsm .xls .xlsb .ods`; writes `.xlsx` (bold, frozen header row, column widths fitted). Dates become ISO text.

- `excel.read(path, { sheet, header = true })` → list · map · error — The first sheet (or `sheet`: name or index) as rows; `sheet: "*"` → `{ name: rows }`.
- `excel.sheets(path)` → list · error — Sheet names.
- `excel.write(path, rows | { Sheet: rows }, { header = true, columns })` → nil · error — One sheet per map key.

```fogu
import excel from "fogu:excel"
const products = [
    { id: 1, name: "Café", price: 4.5, stock: true },
    { id: 2, name: "Pão", price: 0.8, stock: false },
]
excel.write("shop.xlsx", { Products: products, Notes: [["checked", true]] })?
print(excel.sheets("shop.xlsx")?)
print(excel.read("shop.xlsx")?)
print(excel.read("shop.xlsx", { sheet: "Notes", header: false })?)
```

output:

```text
[Notes, Products]
[]
[[checked, true]]
```

## fogu:yaml

```fogu
import yaml from "fogu:yaml"
```

- `yaml.parse(text)` → value · error
- `yaml.parseAll(text)` → list · error — Every document of a `---` stream.
- `yaml.stringify(value)` → str
- `yaml.read(path) · yaml.write(path, value)` → value · nil

```fogu
import yaml from "fogu:yaml"
const cfg = yaml.parse("server:\n  host: 0.0.0.0\n  ports: [80, 443]\ndebug: false\n")?
print(cfg.server.ports, cfg.debug)
print(yaml.stringify({ app: "fogu", tags: ["web", "cli"] }))
```

output:

```text
[80, 443] false
app: fogu
tags:
- web
- cli
```

## fogu:toml

```fogu
import toml from "fogu:toml"
```

Dates become text. TOML has no null: `nil` values are left out when writing; the top level must be a map.

- `toml.parse(text)` → map · error
- `toml.stringify(map)` → str
- `toml.read(path) · toml.write(path, map)` → map · nil

```fogu
import toml from "fogu:toml"
const t = toml.parse("title = \"app\"\n[db]\nurl = \"sqlite://x.db\"\npool = 5\n[[users]]\nname = \"ana\"\n")?
print(t.title, t.db.pool, t.users[0].name)
print(toml.stringify({ name: "fogu", version: 5, server: { port: 8080 } }))
```

output:

```text
app 5 ana
name = "fogu"
version = 5

[server]
port = 8080
```

## fogu:xml

```fogu
import xml from "fogu:xml"
```

An element is `{ name, attrs, children, text }`; `children` holds elements (maps) and text (str); `text` joins the element's own text.

- `xml.parse(text, { whitespace })` → map · error — The root element.
- `xml.stringify(element, { pretty = true, declaration = true })` → str
- `xml.read(path) · xml.write(path, element, options?)` → map · nil

```fogu
import xml from "fogu:xml"
const feed = xml.parse("<feed lang=\"pt\"><item id=\"1\">Olá &amp; adeus</item><item id=\"2\"/></feed>")?
print(feed.name, feed.attrs.lang, feed.children.map((c) => c.attrs.id), feed.children[0].text)
print(xml.stringify({ name: "note", attrs: { to: "Ana" }, children: [{ name: "body", text: "1 < 2" }] }))
```

output:

```text
feed pt [1, 2] Olá & adeus
<?xml version="1.0" encoding="UTF-8"?>
<note to="Ana">
  <body>1 &lt; 2</body>
</note>
```

## fogu:kv

```fogu
import kv from "fogu:kv"
```

An embedded key-value store in a single file (redb, ACID). Values are any Fogu value (stored as JSON).

- `kv.open(path)` → Store · error — Creates the file if needed.
- `store.set(key, value) · store.get(key)` → nil · value — `get` → nil when missing.
- `store.has(key) · store.delete(key)` → bool
- `store.keys() · store.len() · store.clear()` → list · int · nil
- `store.path · store.close()` → str · nil

```fogu
import kv from "fogu:kv"
const db = kv.open("app.db")?
db.set("user:1", { name: "Ana", roles: ["admin"] })
db.set("visits", 41)
db.set("visits", db.get("visits") + 1)
print(db.get("user:1").name, db.get("visits"), db.has("nope"), db.keys(), db.len())
db.close()
```

output:

```text
Ana 42 false [user:1, visits] 2
```

## fogu:sql

```fogu
import sql from "fogu:sql"
```

SQLite, PostgreSQL and MySQL/MariaDB by URL: `sqlite://app.db`, `sqlite::memory:`, `postgres://user:pass@host/db`, `mysql://user:pass@host/db`. Parameters: `?` (SQLite/MySQL) or `$1` (PostgreSQL).

- `sql.open(url)` → Database · error
- `db.run(sql, params?)` → int · error — One statement with parameters; returns the affected rows (alias `execute`).
- `db.query(sql, params?)` → list · error — Rows as maps.
- `db.one(sql, params?)` → map · nil · error — The first row.
- `db.exec(sql)` → nil · error — Raw SQL without parameters — may hold several statements (migrations).
- `db.begin() · db.commit() · db.rollback()` → nil · error — Transactions.
- `db.driver · db.url · db.close()`

```fogu
import sql from "fogu:sql"
const db = sql.open("sqlite::memory:")?
db.exec("CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT, age INT)")?
const n = db.run("INSERT INTO users (name, age) VALUES (?, ?)", ["Ana", 30])?
db.run("INSERT INTO users (name, age) VALUES (?, ?)", ["Bob", 25])?
print("inserted:", n)
print(db.query("SELECT * FROM users WHERE age > ? ORDER BY name", [20])?)
print(db.one("SELECT COUNT(*) AS n FROM users")?.n, db.driver)
const [_, err] = db.query("SELECT * FROM nope")
print(err != nil)
```

output:

```text
inserted: 1
[{age: 30, id: 1, name: Ana}, {age: 25, id: 2, name: Bob}]
2 sqlite
true
```

## fogu:orm

```fogu
import { create_client } from "fogu:orm"
```

A Prisma-style ORM for SQLite, PostgreSQL and MySQL. Models are described in a map; `migrate: "auto"` creates tables (with foreign keys) and adds new columns. Each model is reached as `db. `. Relations load with one query per level.

### Client and schema

- `create_client({ schema, migrate, log })` → Client · error — `schema: { provider, url, models }`; `log: true` prints every SQL statement.
- `field: { type, primary, autoincrement, required, unique, defaultValue }` — Types `Int BigInt String Bool Float DateTime Json`; defaults `"now"`, `"uuid"` or a value.
- `relation: { model, kind, foreignKey, references }` — `one-to-many`, `many-to-one`, `one-to-one`.
- `db.query(sql, params?) · db.execute(sql, params?)` → list · int — Raw SQL.
- `db.begin() · db.commit() · db.rollback() · db.migrate() · db.disconnect()`

### Model methods

`where`: `{ field: value }`, `{ field: { equals, not, in, notIn, lt, lte, gt, gte, contains, startsWith, endsWith, mode: "insensitive" } }`, `AND`/`OR`/`NOT`, and `nil` for IS NULL. `orderBy`: `{ field: "asc" | "desc" }` or a list of them.

- `create({ data, include, select })` → map — Missing required fields stop the program; a duplicate is an error value.
- `createMany({ data: [...] })` → { count }
- `findMany({ where, select, include, orderBy, take, skip })` → list
- `findFirst(opts) · findUnique(opts)` → map · nil
- `update({ where, data })` → map · error — `data` values may be `{ increment, decrement, multiply, divide, set }`.
- `updateMany({ where, data }) · deleteMany({ where })` → { count }
- `upsert({ where, create, update })` → map
- `delete({ where })` → map · error — The deleted record.
- `count({ where })` → int

```fogu
import { create_client } from "fogu:orm"

const db = create_client({
    schema: {
        provider: "sqlite",
        url: "sqlite::memory:",
        models: {
            User: {
                fields: {
                    id:    { type: "Int", primary: true, autoincrement: true },
                    email: { type: "String", required: true, unique: true },
                    role:  { type: "String", defaultValue: "user" },
                },
                relations: { posts: { model: "Post", kind: "one-to-many", foreignKey: "authorId" } }
            },
            Post: {
                fields: {
                    id:       { type: "Int", primary: true, autoincrement: true },
                    title:    { type: "String", required: true },
                    authorId: { type: "Int", required: true },
                },
                relations: { author: { model: "User", kind: "many-to-one", foreignKey: "authorId" } }
            }
        }
    },
    migrate: "auto",
})?

const ana = db.user.create({ data: { email: "ana@x.com" } })?
db.post.createMany({ data: [{ title: "Hello", authorId: ana.id }, { title: "Fogu", authorId: ana.id }] })?
print(ana)
const users = db.user.findMany({ include: { posts: { select: { title: true } } } })?
print(users[0].posts)
print(db.post.findFirst({ where: { title: { startsWith: "Fo" } }, include: { author: true } })?.author.email)
print(db.user.update({ where: { id: ana.id }, data: { role: "admin" } })?.role, db.post.count()?)
```

output:

```text
{email: ana@x.com, id: 1, role: user}
[{title: Hello}, {title: Fogu}]
ana@x.com
admin 2
```

## fogu:redis

```fogu
import redis from "fogu:redis"
```

Redis, Valkey and KeyDB. **Every Redis command is a method** (`r.lpush(…)`, `r.xadd(…)`) or goes through `r.command("CMD", …)`; maps become `field value` pairs. Redis errors (`WRONGTYPE`, `NOAUTH`) are error values.

- `redis.connect(url | { host, port, username, password, db, tls, timeout, name })` → Redis · error — `redis://[user:pass@]host[:port][/db]`, `rediss://` for TLS, default `$REDIS_URL`.
- `r.set(key, value, { ex, px, exat, nx, xx, keepTtl, get })` → bool · value — `false` when `nx`/`xx` prevented it.
- `r.get(key) · r.del(keys) · r.incr(key) · r.expire(key, s) · r.ttl(key)`
- `r.hset(key, map) · r.hget(key, f) · r.hgetall(key)` → int · str · map
- `r.zrange(key, start, stop, { withScores, rev, byScore })` → list — With scores: `[{ member, score }]`.
- `r.setJson(key, value, opts?) · r.getJson(key)` → bool · value — JSON values.
- `r.pipeline([[cmd, …args], …])` → list — One round trip; a failed command is an error value in its place.
- `r.transaction([...])` → list · error — `MULTI … EXEC`.
- `r.subscribe(...channels) · r.psubscribe(...patterns)` → Subscription — Its own connection: `next(timeout?)` → `{ channel, message, pattern? }`, `tryNext()`, `close()`.
- `r.publish(channel, message)` → int — Receivers.
- `r.close()` → nil

```fogu
import redis from "fogu:redis"
const r = redis.connect("redis://127.0.0.1:6379/3")?
print(r.set("name", "Ana", { ex: 60 })?, r.get("name")?, r.ttl("name")?)
print(r.set("name", "x", { nx: true })?)              // exists: not set
r.hset("user:1", { name: "Ana", age: 30 })?
print(r.hgetall("user:1")?)
r.zadd("rank", [10, "ana", 25, "bob"])?
print(r.zrange("rank", 0, -1, { withScores: true, rev: true })?)
print(r.pipeline([["INCR", "n"], ["INCR", "n"], ["GET", "n"]])?)
const sub = r.subscribe("news")?
r.publish("news", "hello")?
print(sub.next(1s)?)
```

output (with a Redis server):

```text
true Ana 60
false
{age: 30, name: Ana}
[{member: bob, score: 25.0}, {member: ana, score: 10.0}]
[1, 2, 2]
{channel: news, message: hello}
```

## fogu:s3

```fogu
import s3 from "fogu:s3"
```

AWS S3 and compatible storage (MinIO, Cloudflare R2, Backblaze B2, DigitalOcean Spaces, Wasabi) with AWS Signature V4. Credentials default to `$AWS_ACCESS_KEY_ID`, `$AWS_SECRET_ACCESS_KEY`, `$AWS_REGION`, `$AWS_ENDPOINT_URL`.

Every method also takes `{ bucket }` in its options.

- `s3.connect({ endpoint, region, accessKey, secretKey, sessionToken, bucket, pathStyle, timeout })` → S3 — Path-style URLs when an `endpoint` is given.
- `store.put(key, body, { contentType, metadata, cacheControl, acl })` → { etag } · error — `body`: str, list of bytes, or any value (JSON).
- `store.upload(path, key?, opts?)` → { etag } · error — A local file.
- `store.get(key) · store.getBytes(key) · store.download(key, path)` → str · list · nil
- `store.head(key) · store.exists(key)` → map · nil · bool — `{ size, etag, contentType, lastModified, metadata }`.
- `store.list(prefix?, { delimiter, max })` → list — `{ key, size, etag, lastModified }`; folders `{ key, dir: true }`.
- `store.copy(from, to, { fromBucket }) · store.delete(key | keys)`
- `store.presign(key, { method = "GET", expires = 1h })` → str — A temporary URL that needs no credentials.
- `store.url(key) · store.withBucket(name)` → str · S3
- `store.createBucket(name?) · store.deleteBucket(name?) · store.listBuckets()`

```fogu
import s3 from "fogu:s3"
const store = s3.connect({ endpoint: "http://127.0.0.1:9000", accessKey: "fogu", secretKey: "secret", bucket: "docs" })?
store.put("notes/hello.txt", "Olá, S3!", { metadata: { author: "ana" } })?
print(store.get("notes/hello.txt")?)
print(store.head("notes/hello.txt")?.metadata, store.exists("nope")?)
print(store.list("notes/")?.map((o) => o.key))
print(store.presign("notes/hello.txt", { expires: 15m }).contains("X-Amz-Signature="))
```

output (with an S3 server):

```text
Olá, S3!
{author: ana}
false
[notes/hello.txt]
true
```

## fogu:scylla

```fogu
import scylla from "fogu:scylla"
```

ScyllaDB and Apache Cassandra (CQL). Statements with parameters are prepared once and cached; parameters are converted to the column types (collections, UDTs, uuid, timestamp…).

- `scylla.connect("host:port" | { hosts, keyspace, user, password, singleNode, addressMap })` → Scylla · error — `addressMap` translates addresses of nodes behind Docker/NAT.
- `db.execute(cql, params?)` → nil · error
- `db.query(cql, params?)` → list · error — Rows as maps.
- `db.one(cql, params?)` → map · nil · error
- `db.use(keyspace) · db.close()`

```fogu
import scylla from "fogu:scylla"
const db = scylla.connect({ hosts: ["127.0.0.1:9042"] })?
db.execute("CREATE KEYSPACE IF NOT EXISTS shop WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1}")?
db.use("shop")?
db.execute("CREATE TABLE IF NOT EXISTS users (id int PRIMARY KEY, name text, tags set<text>)")?
db.execute("INSERT INTO users (id, name, tags) VALUES (?, ?, ?)", [1, "Ana", ["admin"]])?
print(db.one("SELECT * FROM users WHERE id = ?", [1])?)
```

output (with a ScyllaDB server):

```text
{id: 1, name: Ana, tags: [admin]}
```

## fogu:http

```fogu
import http from "fogu:http"
```

A minimal HTTP server (one handler for every request) and an HTTP client. For routing, middleware, static files, uploads, SSE and WebSocket use [fogu:web](#foguweb).

### Client

- `http.get(url) · http.delete(url)` → Response · error — Network failures are error values; HTTP 4xx/5xx are normal responses.
- `http.post(url, body) · http.put(url, body)` → Response · error — `body`: str, or a map sent as JSON.
- `res.status · res.headers · res.body · res.json()`

### Server

- `http.serve(port | "ip:port", fn(req))` → nil — Requests are handled while the program runs (e.g. in a `while` loop with `time.delay`). The address is an int port (on 127.0.0.1) or `"0.0.0.0:8080"`, `"[::1]:8080"`, `":8080"`.
- `req.method · req.path · req.query · req.headers · req.body` — `query` is the raw query string.
- `req.header(name) · req.json() · req.form() · req.bytes()` — `form()`: urlencoded and multipart bodies.
- `http.text(body, opts?) · http.json(value, opts?) · http.html(body, opts?)` → Response — `opts: { status, headers }`.

```fogu
import http from "fogu:http"
import time from "fogu:time"

http.serve(8080, (req) => {
    if req.path == "/hello" {
        return http.json({ hello: "world", method: req.method })
    }
    return http.text("not found", { status: 404 })
})
while true { time.delay(50ms) }
```

```fogu
import http from "fogu:http"
const r = http.get("http://127.0.0.1:8080/hello")?
print(r.status, r.headers["content-type"], r.json())
print(http.get("http://127.0.0.1:8080/nope")?.status)
const [_, err] = http.get("http://127.0.0.1:1/")
print(err != nil)
```

output:

```text
200 application/json {hello: world, method: GET}
404
true
```

## fogu:web

```fogu
import web from "fogu:web"
```

A web framework: routes with parameters, middleware, static files and SPA builds, uploads, downloads, Server-Sent Events and WebSocket — on one port.

### App

- `web.app()` → App — Methods return the app, so calls can be chained.
- `app.get/post/put/patch/delete/head/options/all(path, ...middlewares, handler)` → App — `/users/:id` parameters, `*` wildcard.
- `app.use(fn) · app.use(prefix, fn)` → App — Middleware for every request, or under a path prefix.
- `app.after((ctx, res) => …)` → App — After the handler; returning a value replaces the response.
- `app.onError((ctx, err) => response)` → App — For runtime errors and returned `error` values (default: 500).
- `app.static(dir) · app.static(prefix, dir, { spa, index, maxAge, dotfiles })` → App — Files served by the server thread: ETag/304, `Cache-Control`; `spa: true` serves `index.html` for unknown HTML routes (React, Vue, Svelte builds).
- `app.ws(path, ...middlewares, (ws, ctx) => …, { protocols })` → App — WebSocket routes; the handler runs in its own task with a [WebSocket](#foguwebsocket).
- `app.set(key, value) · app.store` → App · map — Shared state (`ctx.store`).
- `app.listen(port | "ip:port", { host = "127.0.0.1", maxBody = 10 MB, quiet })` → nil · error — `app.listen("0.0.0.0:3000")` (or `host: "0.0.0.0"`) accepts other machines; IPv6 as `"[::1]:3000"`. A host in the string wins over `host`.
- `app.handler()` → fn — Calls the app without a network (tests): `h({ method, path, headers, body })` → Response.
- `app.routes()` → list

### Context and responses

- `ctx.method · ctx.path · ctx.params · ctx.query · ctx.headers · ctx.body` — `query` is a map; header names are lower case.
- `ctx.req.json() · ctx.req.form() · ctx.req.bytes()` — `form()`: fields as str, files as `UploadedFile`.
- `ctx.set.status · ctx.set.headers` — Adjust the final response from middleware or handlers.
- `ctx.store` → map — The app's shared map.
- `handler result` — A str (text or HTML), any value (JSON), a response, or nil (204).
- `web.json/text/html(body, { status, headers })` → Response
- `web.redirect(url, status = 302)` → Response
- `web.file(path, { download, contentType, status })` → Response · error — Sent by the server thread (binary safe).
- `web.sse({ retry, headers })` → SseStream — Return it from the handler; `send(data, { event, id })`, `comment(text)`, `close()`, field `open`.
- `web.cors({ origin, methods, headers, credentials, maxAge })` → middleware — Answers preflight requests.
- `web.logger()` → middleware — `GET /path 200 0.4ms`.
- `UploadedFile: filename, contentType, size, text(), bytes(), json(), save(path)` — `save("uploads/")` keeps a sanitized file name.

```fogu
import web from "fogu:web"
import time from "fogu:time"

const app = web.app()
app.use(web.logger())
app.use("/api", (ctx) => {
    if ctx.headers["x-key"] != "secret" { return web.json({ error: "forbidden" }, { status: 403 }) }
})
app.get("/api/users/:id", (ctx) => ({ id: ctx.params.id, page: ctx.query.page ?? "1" }))
app.post("/api/upload", (ctx) => {
    const form = ctx.req.form()?
    return { saved: form.file.save("uploads/")?, size: form.file.size }
})
app.get("/events", (ctx) => {
    const stream = web.sse()
    fg fn() {
        for i in 1..4 { stream.send({ n: i }, { event: "tick" }); time.delay(1s) }
        stream.close()
    }()
    return stream
})
app.ws("/chat", (ws, ctx) => {
    while true {
        const m = ws.next()?
        if m == nil { break }
        ws.send(`echo: ${m.data}`)
    }
})
app.static("/", "dist", { spa: true, maxAge: 1d })
app.listen(3000)?
while true { time.delay(50ms) }
```

Without a network, through `app.handler()`:

```fogu
import web from "fogu:web"
const app = web.app()
app.use((ctx) => { ctx.set.headers["x-served-by"] = "fogu" })
app.get("/users/:id", (ctx) => ({ id: ctx.params.id, sort: ctx.query.sort }))
app.get("/old", (ctx) => web.redirect("/new"))
app.get("/fail", (ctx) => error("boom"))
app.onError((ctx, err) => web.json({ error: err.message }, { status: 500 }))
const h = app.handler()
const show = (path) => {
    const r = h({ method: "GET", path: path })
    print(r.status, r.body, r.headers["x-served-by"], r.headers["location"])
}
show("/users/7?sort=name")
show("/old")
show("/fail")
show("/missing")
```

output:

```text
200 {"id":"7","sort":"name"} fogu nil
302  fogu /new
500 {"error":"boom"} fogu nil
404 not found fogu nil
```

## fogu:jwt

```fogu
import jwt from "fogu:jwt"
```

JSON Web Tokens with HMAC (HS256/HS384/HS512). A token that fails verification is an error value.

- `jwt.sign(payload, secret, { algorithm, expiresIn, notBefore, issuer, audience, subject, jwtid, header, noTimestamp })` → str — `iat` is added; durations or seconds for times.
- `jwt.verify(token, secret, { algorithms, issuer, audience, subject, leeway, maxAge, ignoreExpiration })` → map · error — The claims.
- `jwt.decode(token)` → map · error — `{ header, payload }` without verifying.
- `jwt.middleware(secret, { property = "user", optional, …verify options })` → middleware — For `fogu:web`: `Authorization: Bearer` → `ctx.user`, otherwise 401.

```fogu
import jwt from "fogu:jwt"
const token = jwt.sign({ sub: "42", role: "admin" }, "s3cret", { expiresIn: 1h, issuer: "fogu" })
const claims = jwt.verify(token, "s3cret", { issuer: "fogu" })?
print(claims.sub, claims.role, claims.exp - claims.iat)
print(jwt.decode(token)?.header)
const [_, e1] = jwt.verify(token, "wrong")
const [_, e2] = jwt.verify(jwt.sign({ a: 1 }, "s3cret", { expiresIn: -10 }), "s3cret")
print(e1, e2)
```

output:

```text
42 admin 3600
{alg: HS256, typ: JWT}
error: jwt: invalid signature error: jwt: token expired
```

## fogu:websocket

```fogu
import websocket from "fogu:websocket"
```

WebSocket client (`ws://`, `wss://`) and a standalone server. The same connection object is used by `app.ws` in `fogu:web`.

- `websocket.connect(url, { headers, protocols, timeout })` → WebSocket · error — A rejected handshake (401, 404…) is an error value.
- `websocket.listen(port, { host })` → Server · error — `port 0` picks a free port; `server.accept(timeout?)` → WebSocket, fields `port url`.
- `ws.send(data)` → nil · error — str → text frame, list of bytes → binary, other values → JSON text.
- `ws.next(timeout?)` → Message · nil — `{ type: "text" | "binary", data, bytes, size, json() }`; nil on timeout or when closed.
- `ws.tryNext() · ws.ping() · ws.close(code = 1000, reason)`
- `ws.open · ws.closeCode · ws.closeReason · ws.path · ws.url · ws.headers · ws.protocol`

```fogu
import websocket from "fogu:websocket"
const server = websocket.listen(0)?
fg fn() {
    const conn = server.accept()?
    while true {
        const m = conn.next()?
        if m == nil { break }
        conn.send(`echo ${conn.path}: ${m.data}`)
    }
}()
const ws = websocket.connect(server.url + "/room?x=1")?
ws.send("hello")
print(ws.next(2s)?.data)
ws.send({ kind: "json" })
print(ws.next(2s)?.data)
ws.close()
```

output:

```text
echo /room?x=1: hello
echo /room?x=1: {"kind":"json"}
```

## fogu:net

```fogu
import net from "fogu:net"
```

TCP and UDP sockets. Data is text (UTF-8).

- `net.tcpListen("host:port")` → Listener · error — `accept()` → Stream, `setNonblocking(bool)`, `close()`.
- `net.tcpConnect("host:port")` → Stream · error
- `stream.write(text) · stream.writeLine(text)` → nil · error
- `stream.read(n = 4096) · stream.readLine() · stream.readAll()` → str · error — `readLine` without the newline; "" at the end.
- `stream.setTimeout(ms) · stream.close()`
- `net.udpBind("host:port")` → Socket · error — `sendTo(addr, data)`, `recvFrom()` → `{ data, addr, len }`, `connect(addr)`, `send(data)`, `recv()`, `setTimeout(ms)`.
- `net.resolve(host)` → list · error — IP addresses.

```fogu
import net from "fogu:net"
const server = net.tcpListen("127.0.0.1:40123")?
const client = net.tcpConnect("127.0.0.1:40123")?
const conn = server.accept()?
client.writeLine("ping")?
print(conn.readLine()?)
conn.writeLine("pong")?
print(client.readLine()?)

const a = net.udpBind("127.0.0.1:40124")?
const b = net.udpBind("127.0.0.1:40125")?
a.sendTo("127.0.0.1:40125", "hello udp")?
const p = b.recvFrom()?
print(p.data, p.addr)
```

output:

```text
ping
pong
hello udp 127.0.0.1:40124
```

## fogu:events

```fogu
import events from "fogu:events"
```

- `events.EventEmitter()` → Emitter
- `e.on(event, fn) · e.once(event, fn)` → nil — Listeners receive every emitted argument (declare them all).
- `e.off(event, fn?) · e.removeAll(event?)` → nil
- `e.emit(event, ...args)` → bool — `true` when someone listened.
- `e.listeners(event) · e.eventNames() · e.hasListeners(event)`

```fogu
import events from "fogu:events"
const bus = events.EventEmitter()
bus.on("order", (id, total) => print("order", id, "total", total))
bus.once("order", (id, total) => print("first order only:", id))
print(bus.emit("order", 1, 9.9))
print(bus.emit("order", 2, 15))
print(bus.emit("nobody-listens"), bus.eventNames())
```

output:

```text
order 1 total 9.9
first order only: 1
true
false
false []
```

## fogu:cron

```fogu
import cron from "fogu:cron"
```

Jobs run when the program calls `cron.tick()` (typically in its main loop).

- `cron.every(interval, fn)` → nil — A job every `interval` (duration or ms).
- `cron.schedule(expression, fn)` → nil — 5-field cron expression (`"*/5 * * * *"`); an invalid one stops the program.
- `cron.tick()` → int — Runs the jobs that are due; returns how many ran.
- `cron.list()` → list — Job names (`every(10ms)`, the cron expression…).
- `cron.clear()` → nil — Removes every job.

```fogu
import cron from "fogu:cron"
import time from "fogu:time"
let beats = 0
cron.every(10ms, () => { beats = beats + 1 })
cron.schedule("0 3 * * *", () => print("nightly backup"))
for i in 0..6 {
    cron.tick()
    time.delay(10ms)
}
print(beats >= 3, cron.list())
```

output:

```text
true [every(10ms), schedule(0 3 * * *)]
```

## fogu:nats

```fogu
import nats from "fogu:nats"
```

NATS core protocol plus JetStream (streams, acknowledged publishing, pull consumers).

- `nats.connect(url | { url, user, pass, token, name, timeout })` → Nats · error — `nats://[user:pass@|token@]host:4222`, default `$NATS_URL`.
- `nc.publish(subject, data?, { headers, reply })` → nil · error — `data`: str, or any value as JSON.
- `nc.subscribe(subject, { queue, max })` → Subscription — Wildcards `*` and `>`; queue groups split the messages.
- `sub.next(timeout?) · sub.tryNext() · sub.unsubscribe()` → Message · nil
- `nc.request(subject, data?, { timeout, headers })` → Message · error — No responders → error value.
- `nc.flush() · nc.close() · nc.connected · nc.server`
- `msg.subject · .data · .headers · .reply · .json() · .respond(data)`

### JetStream

- `nc.jetstream({ prefix, timeout })` → JetStream
- `js.addStream({ name, subjects, storage, max_msgs, max_age, … })` → map · error — NATS stream config (snake_case keys).
- `js.updateStream(cfg) · js.streamInfo(name) · js.deleteStream(name) · js.purgeStream(name)`
- `js.publish(subject, data, { msgId, headers, timeout })` → { stream, seq, duplicate } — `msgId` deduplicates.
- `js.consumer(stream, { durable, filter, ackWait, deliver, maxDeliver })` → Consumer · error — A durable pull consumer.
- `consumer.fetch(batch = 1, expires = 5s)` → list — Up to `batch` messages.
- `msg.ack() · msg.nak(delay?) · msg.term() · msg.inProgress()` → nil — `msg.seq`, `msg.delivered`, `msg.stream`.

```fogu
import nats from "fogu:nats"
const nc = nats.connect("nats://127.0.0.1:4222")?
const sub = nc.subscribe("orders.*")
nc.publish("orders.created", { id: 1 })
print(sub.next(1s)?.json())

const svc = nc.subscribe("math.double")
fg fn() { const req = svc.next()?; req.respond(str(int(req.data) * 2)) }()
print(nc.request("math.double", "21")?.data)

const js = nc.jetstream()
js.addStream({ name: "TASKS", subjects: ["tasks.>"] })?
print(js.publish("tasks.email", { to: "ana@x.com" }, { msgId: "t1" })?)
const c = js.consumer("TASKS", { durable: "workers" })?
for msg in c.fetch(10, 1s)? { print(msg.subject, msg.seq); msg.ack() }
```

output (with a NATS server, -js):

```text
{id: 1}
42
{duplicate: false, seq: 1, stream: TASKS}
tasks.email 1
```

## fogu:rabbitmq

```fogu
import rabbitmq from "fogu:rabbitmq"
```

RabbitMQ (AMQP 0-9-1). Publishing waits for the broker's confirmation. A channel closed by an error (for example declaring a queue with other options) is reopened on the next call.

- `rabbitmq.connect(url?)` → RabbitMQ · error — `amqp://user:pass@host:5672/vhost`, default `$AMQP_URL`.
- `mq.exchange(name, kind = "direct", { durable, autoDelete, internal, arguments })` → nil · error — `direct fanout topic headers`.
- `mq.queue(name = "", { durable, exclusive, autoDelete, ttl, maxLength, deadLetter, passive })` → { name, messages, consumers } — "" gives a server-named queue.
- `mq.bind(queue, exchange, key = "") · mq.unbind(…)` → nil · error
- `mq.publish(exchange, key, data, { persistent, headers, contentType, correlationId, replyTo, messageId, expiration, priority, mandatory })` → nil · error — Maps are sent as JSON.
- `mq.send(queue, data, opts?)` → nil · error — Through the default exchange.
- `mq.get(queue, { noAck })` → Message · nil
- `mq.consume(queue, { prefetch = 10, noAck, exclusive, tag })` → Consumer — `next(timeout?)`, `tryNext()`, `cancel()`.
- `msg.data · .json() · .routingKey · .exchange · .headers · .redelivered · …` — Properties: `contentType correlationId replyTo messageId type persistent`.
- `msg.ack() · msg.nack({ requeue = true }) · msg.reject({ requeue })` → nil
- `mq.purge(queue) · mq.deleteQueue(name) · mq.deleteExchange(name) · mq.close()`

```fogu
import rabbitmq from "fogu:rabbitmq"
const mq = rabbitmq.connect("amqp://guest:guest@127.0.0.1:5672/%2f")?
mq.exchange("shop", "topic", { durable: true })?
mq.queue("emails", { durable: true })?
mq.bind("emails", "shop", "order.*")?
mq.publish("shop", "order.created", { id: 7 }, { persistent: true })?
const worker = mq.consume("emails", { prefetch: 5 })?
const msg = worker.next(2s)?
print(msg.routingKey, msg.json(), msg.persistent)
msg.ack()
```

output (with RabbitMQ):

```text
order.created {id: 7} true
```

## fogu:mqtt

```fogu
import mqtt from "fogu:mqtt"
```

MQTT 3.1.1 with automatic reconnection (subscriptions are restored).

- `mqtt.connect(url?, { clientId, username, password, keepAlive, clean = true, will, timeout })` → Mqtt · error — `mqtt://` or `mqtts://`; `will: { topic, payload, qos, retain }`.
- `m.publish(topic, data, { qos = 0, retain })` → nil · error
- `m.subscribe(filter | [filters], { qos }) · m.unsubscribe(…)` → nil · error — `+` one level, `#` the rest.
- `m.next(timeout?) · m.tryNext()` → Message · nil — `{ topic, data, qos, retain, json(), matches(filter) }`.
- `m.connected · m.clientId · m.subscriptions · m.close()`
- `mqtt.matches(filter, topic)` → bool

```fogu
import mqtt from "fogu:mqtt"
const m = mqtt.connect("mqtt://127.0.0.1:1883", { clientId: "monitor" })?
m.subscribe("home/+/temperature", { qos: 1 })?
m.publish("home/kitchen/temperature", "21.5", { qos: 1, retain: true })?
const msg = m.next(2s)?
print(msg.topic, msg.data, msg.qos, msg.matches("home/#"))
print(mqtt.matches("a/+/c", "a/b/c"))
```

output (with an MQTT broker):

```text
home/kitchen/temperature 21.5 1 true
true
```

## fogu:fogumq

```fogu
import fogumq from "fogu:fogumq"
```

Client of the Fogu broker (binary protocol, JSON payloads): key/value store, channels, pub/sub, RPC and API keys. Methods that wait for the server let the other `fg` tasks run.

- `fogumq.connect("fogu://token@host:port", { timeout, requestTimeout })` → FoguMQ · error — Fields `connected authenticated isRoot url`; `ping() close()`.
- `mq.kv.get(key) · set(key, value, ttl?) · del(key) · exists(key) · incr(key, n?) · decr(key, n?)` → value · error — `get` gives nil for a missing key.
- `mq.channel(name)` → Channel — `join(alias?) leave() send(data) members() list()`.
- `channel.onMessage()` → Listener — `next(timeout?)` → `{ channel, data, ... }`; `tryNext() close()`.
- `channel.subscribe(topic) · channel.publish(topic, data)` → Subscription · error — Topics live under the channel: `name/topic`.
- `channel.act(method)` → Service · error — `next()` → Request with `method params reply(result) fail(msg)`.
- `channel.call(method, params?, timeout?)` → value · error
- `mq.apikeys.create(name, { channels, kv, topics, rpc }) · delete(id) · get(id) · list()` → value · error — Root token only.

```fogu
import fogumq from "fogu:fogumq"
const mq = fogumq.connect(Fogu.env.FOGUMQ_URL)?
mq.kv.set("visits", 1)?
print(mq.kv.incr("visits")?)
const room = mq.channel("calc")
const svc = room.act("sum")?
fg fn() { const r = svc.next(); r.reply(r.params.reduce((a, n) => a + n, 0)) }()
print(room.call("sum", [1, 2, 3])?)
```

output (with a Fogu broker):

```text
2
6
```

## fogu:bullmq

```fogu
import bullmq from "fogu:bullmq"
```

Job queues on Redis, wire-compatible with [BullMQ](https://docs.bullmq.io) for Node.js v5 (on the `bullmq-rs` crate): Fogu and Node producers and workers can share the same queues, and dashboards such as Bull Board see them. The worker owns locks, stalled-job recovery, retries with backoff, concurrency and rate limits; each job is handed to the program, which finishes it with `complete` or `fail`. Redis failures are `error` values.

### Queue

- `bullmq.queue(name, { connection, prefix = "bull" })` → BullQueue · error — `connection` is a `redis://` / `rediss://` URL (default `$REDIS_URL` or `redis://127.0.0.1:6379`).
- `q.add(name, data, opts?)` → job · error — Options: `delay attempts backoff: { type: "fixed" | "exponential", delay, max } priority jobId ttl deduplication: { id, ttl }`.
- `q.addBulk([{ name, data, opts }])` → list · error
- `q.getJob(id) · q.getJobs(states, start = 0, end = -1, asc = false)` → job | nil · list — A job: `{ id name data state timestamp attemptsMade progress processedOn finishedOn failedReason returnvalue opts }`.
- `q.counts() · q.count()` → map · int — Per state: `wait active delayed prioritized completed failed paused waiting-children`.
- `q.upsertJobScheduler(id, { every | pattern, tz, limit, immediately }, { name, data, opts })` → job · error — Repeatable jobs (cron `pattern` or `every` duration).
- `q.removeJobScheduler(id)` → bool · error
- `q.pause() · resume() · isPaused() · drain() · remove(id) · clean(grace, limit, state) · obliterate({ force }) · getJobLogs(id)`

### Worker and events

Limitation: the worker of `bullmq-rs` publishes `completed` events with an empty return value; `q.getJob(id).returnvalue` has the value given to `complete`.

- `bullmq.worker(name, { connection, prefix, concurrency = 1, lockDuration, maxStalledCount, limiter: { max, duration } })` → BullWorker · error
- `w.next(timeout?) · w.tryNext()` → BullJob | nil · error — Waits cooperatively (other `fg` tasks keep running); nil on timeout or after `close()`.
- `job.complete(value?) · job.fail(reason)` → nil — `fail` retries the job while `attempts` remain. A job dropped without an answer fails.
- `job.progress(value) · job.log(text)` → nil — Fields are those of the job map, plus `done`.
- `w.pause() · w.resume() · w.close() · w.running · w.pending`
- `bullmq.events(name, { connection, prefix, lastEventId })` → BullEvents · error — `next(timeout?)` → `{ event, jobId, id, … }` (`added waiting active progress completed failed delayed stalled removed`).

```fogu
import bullmq from "fogu:bullmq"
const conn = { connection: "redis://127.0.0.1:6379" }
const q = bullmq.queue("emails", conn)?
q.add("welcome", { to: "ana@x.com" }, { attempts: 3, backoff: { type: "exponential", delay: 1s } })?
const w = bullmq.worker("emails", { connection: conn.connection, concurrency: 4 })?
while true {
    const job = w.next()?
    print(job.name, job.data.to, job.attemptsMade)
    job.progress(50)
    job.complete({ sent: true })          // or job.fail("smtp down") → retried
}
```

output (with Redis):

```text
welcome ana@x.com 0
```

## fogu:mail

```fogu
import mail from "fogu:mail"
```

E-mail over SMTP (presets for Gmail, Outlook, Office 365, Yahoo, iCloud, Zoho, Fastmail, AOL, GMX, Amazon SES, Mailtrap, Mailpit) or the HTTP APIs of Resend, SendGrid, Mailgun, Postmark, Brevo, MailerSend and Mailjet. The `console` provider only prints (development and tests).

- `mail.connect({ provider, … } | url)` → Mailer — No connection is made yet.
- `SMTP: { provider: "gmail" | "smtp" | …, host, port, secure, user, password, region }` — `secure`: `tls` (465), `starttls` (587), `none`; URLs `smtp://user:pass@host:587`, `smtps://…`.
- `APIs: { provider: "resend" | "sendgrid" | …, apiKey, domain, region, baseUrl }` — `apiKey` defaults to `$RESEND_API_KEY`, `$SENDGRID_API_KEY`…; Mailgun needs `domain`, Mailjet `apiSecret`.
- `common: { from, timeout }` — Default sender.
- `mailer.send(message)` → { id, provider } · error — A rejected message, a bad address or a network failure is an error value.
- `mailer.sendMany([messages])` → list · error
- `mailer.verify()` → bool · error — SMTP: connects and authenticates.
- `mailer.render(message)` → str — The RFC 5322 text that SMTP would send.
- `message: { from, to, cc, bcc, replyTo, subject, text, html, attachments, headers }` — Addresses: `"a@x.com"`, `"Ana "`, `{ name, email }` or a list.
- `attachment: { filename, content | path, contentType, cid, encoding }` — `content`: str or list of bytes; `cid` makes it inline (` `).

```fogu
import mail from "fogu:mail"
const mailer = mail.connect({ provider: "console", from: "Shop <shop@example.com>" })?
const r = mailer.send({
    to: ["ana@example.com", { name: "Bob", email: "bob@example.com" }],
    subject: "Your order",
    text: "Thanks for buying!",
    attachments: [{ filename: "invoice.csv", content: "item,price\nbook,10\n" }],
})?
print(r)
const gmail = mail.connect({ provider: "gmail", user: "me@gmail.com", password: "app-password" })?
print(gmail.host, gmail.port, gmail.secure)
const [_, err] = mailer.send({ to: "not-an-address", subject: "x", text: "x" })
print(err)
```

output:

```text
--- mail ---
From: Shop <shop@example.com>
To: ana@example.com, Bob <bob@example.com>
Subject: Your order
Attachment: invoice.csv (text/csv, 19 bytes)

Thanks for buying!
---
{id: console-1, provider: console}
smtp.gmail.com 587 starttls
error: mail: invalid address 'not-an-address' in 'to'
```

## fogu:ai

```fogu
import ai from "fogu:ai"
```

LLMs over HTTP: OpenAI-compatible APIs (OpenAI, Groq, OpenRouter, DeepSeek, Mistral, xAI, Together, Fireworks, Perplexity, Ollama, LM Studio), Anthropic and Google Gemini — chat, streaming, tools, agents, embeddings. Keys default to `$OPENAI_API_KEY`, `$ANTHROPIC_API_KEY`, `$GEMINI_API_KEY`…

### Client

- `ai.client({ provider, model, apiKey, baseUrl, timeout, headers, maxTokens } | "provider/model")` → AiClient — Provider `compatible` + `baseUrl` for any OpenAI-style server.
- `client.chat(prompt | { messages, prompt, system, model, temperature, maxTokens, topP, stop, tools, toolChoice, json })` → map · error — `{ text, toolCalls, finishReason, usage: { inputTokens, outputTokens }, model, message }`.
- `client.stream(same)` → AiStream · error — `next(timeout?)` → the next piece of text or nil at the end; `collect()` → like `chat`; fields `text done result`.
- `client.models()` → list · error — `[{ id, name }]`.
- `client.embed(text | [texts], { model })` → list · error — Vectors (OpenAI-style and Gemini).
- `message: { role: "system" | "user" | "assistant" | "tool", content, toolCalls, toolCallId, name }` — `result.message` can be appended to continue the conversation.

### Tools and agents

A tool that returns an `error` value sends `error: …` back to the model, which can recover.

- `ai.tool(name, description, parameters, run?)` → map — `parameters` is a JSON schema; also a plain map `{ name, description, parameters, run }`.
- `ai.agent({ client, model, instructions, tools, maxSteps = 10, temperature, maxTokens, memory, onStep })` → Agent — Tools need `run(args)`.
- `agent.run(input)` → map · error — Calls the model and the tools until it answers: `{ text, steps, messages, usage, finishReason }`.
- `agent.reset() · agent.history` — With `memory: true` the conversation continues across runs.

```fogu
import ai from "fogu:ai"
const llm = ai.client({ provider: "anthropic" })?          // $ANTHROPIC_API_KEY
print(llm.chat("Say hi in Portuguese, one word")?.text)

const s = llm.stream("Count to 3")?
while true { const t = s.next()?; if t == nil { break }; Fogu.write(t) }
print()

const weather = ai.tool("weather", "Current temperature of a city",
    { type: "object", properties: { city: { type: "string" } }, required: ["city"] },
    (args) => ({ city: args.city, celsius: 21 }))
const agent = ai.agent({ client: llm, instructions: "Use the tools", tools: [weather] })
const answer = agent.run("Is it warm in Lisbon?")?
print(answer.text)
print(answer.steps[0].toolCalls[0].name, answer.steps[0].toolCalls[0].arguments)
```

output (the model's words vary):

```text
Olá
1, 2, 3
Yes — it is 21 °C in Lisbon, a pleasant temperature.
weather {city: Lisbon}
```

## fogu:ssh

```fogu
import ssh from "fogu:ssh"
```

Commands, file copies and tunnels over the system's OpenSSH client (key or password).

- `ssh.connect("user@host:port" | { host, user, port, key, password, strictHostKey, knownHosts, timeout, options })` → Ssh · error — Checks the connection; `strictHostKey` defaults to `accept-new`.
- `s.run(command, { input })` → { ok, code, stdout, stderr } · error — A non-zero exit code is not an error.
- `s.upload(local, remote) · s.download(remote, local)` → nil · error — scp, folders included.
- `s.forward(localPort | 0, host, port)` → Tunnel · error — `-L`: a local port that reaches `host:port` through the server.
- `s.reverse(remotePort, host, localPort)` → Tunnel · error — `-R`: a port on the server that reaches a local service.
- `tunnel.localPort · .remotePort · .alive() · .close()`
- `s.close()` → nil

```fogu
import ssh from "fogu:ssh"
const s = ssh.connect("deploy@server.example.com:22", { key: "~/.ssh/id_ed25519" })?
print(s.run("uname -s")?.stdout.trim())
s.upload("dist/app", "/srv/app")?
const db = s.forward(0, "127.0.0.1", 5432)?          // the server's PostgreSQL, locally
print("postgres://app@127.0.0.1:" + str(db.localPort) + "/app")
db.close()
s.close()
```

output (with an SSH server):

```text
Linux
postgres://app@127.0.0.1:53110/app
```

## fogu:shell

```fogu
import shell from "fogu:shell"
```

A small POSIX-style shell: pipes, `&&` `||` `;`, redirections (`> >> 2>&1 &>`), `$VAR`, `$(…)`, quotes, globs and `~`. The usual Linux commands are built in, so scripts behave the same on Windows: `echo cat ls pwd cd mkdir rm cp mv touch grep head tail wc sort uniq cut tr tee basename dirname which env export true false exit sleep seq`. Other commands run as programs.

- `shell.run(command, { cwd, env, input, native = true })` → { ok, code, stdout, stderr } — A non-zero exit is not an error; `native: false` uses system programs only.
- `shell.exec(command, opts?)` → same — Output goes to the terminal as it happens.
- `shell.text(command, opts?)` → str · error — stdout without the final newline; a failure is an error value.
- `shell.lines(command, opts?) · shell.json(command, opts?)` → list · value
- `shell.quote(text)` → str — Safe to interpolate into a command.
- `shell.cd(dir) · shell.cwd()` → nil · str — The shell's own working folder.
- `shell.which(program) · shell.builtins` → str · list

```fogu
import shell from "fogu:shell"
shell.run("mkdir -p data && echo banana > data/fruits.txt && echo apple >> data/fruits.txt")?
print(shell.text("cat data/fruits.txt | sort | head -n 1")?)
print(shell.lines("ls data")?, shell.text("cat data/*.txt | wc -l")?)
const name = "file with spaces.txt"
shell.run(`echo ok > data/${shell.quote(name)}`)?
print(shell.text("echo \"$WHO has $(cat data/fruits.txt | wc -l) fruits\"", { env: { WHO: "Ana" } })?)
const r = shell.run("grep kiwi data/fruits.txt || echo 'no kiwi'")?
print(r.ok, r.stdout.trim())
const [_, err] = shell.text("cat missing.txt")
print(err)
```

output:

```text
apple
[fruits.txt] 2
Ana has 2 fruits
true no kiwi
error: shell.text: exit code 1: cat: missing.txt: No such file or directory
```

## fogu:ffi

```fogu
import ffi from "fogu:ffi"
```

Calls functions of native libraries (C ABI): `.dll`, `.so`, `.dylib`. **Unsafe by nature**: a wrong signature can crash the program. Supported signatures: up to 4 arguments mixing integers and floats, or up to 8 integer/pointer arguments.

- `ffi.open(name)` → Library · error — A path or a name (`"user32"` → `user32.dll` / `libuser32.so`); `"c"` and `"m"` are the C runtime and the math library.
- `lib.func(symbol, [argTypes], returnType = "void")` → fn · error — A Fogu function that calls the symbol.
- `lib.has(symbol) · lib.name` → bool · str
- `types` — `i8 i16 i32 i64 u8 u16 u32 u64 isize usize bool f64 ptr str buffer`; results also `f32 void`. `str` passes a NUL-terminated copy; a `str` result reads a C string.
- `ffi.alloc(size)` → Buffer — Memory for out-parameters: `text() bytes(start?, len?) write(offset, str | bytes) i32/u32/i64/f64(offset)`, fields `size address`.
- `ffi.types · ffi.suffix` → list · str

```fogu
import ffi from "fogu:ffi"
const libc = ffi.open("c")?
const strlen = libc.func("strlen", ["str"], "usize")?
const atoi = libc.func("atoi", ["str"], "i32")?
print(strlen("hello"), strlen("olá"), atoi("42abc"))
const pow = ffi.open("m")?.func("pow", ["f64", "f64"], "f64")?
print(pow(2.0, 10))
const strcpy = libc.func("strcpy", ["buffer", "str"], "ptr")?
const buf = ffi.alloc(16)
strcpy(buf, "copied")
print(buf.text(), buf.bytes(0, 3))
```

output:

```text
5 4 42
1024.0
copied [99, 111, 112]
```

## fogu:playbook

```fogu
import playbook from "fogu:playbook"
```

Configuration management in the style of Ansible: an **inventory** of hosts and **playbooks** of idempotent tasks, run over SSH (the system's OpenSSH client) or on the local machine (through [fogu:shell](#fogushell), so it also works on Windows). Hosts run in parallel; the output and the `PLAY RECAP` follow Ansible's. From the command line: `fogu play site.yml -i inventory.yml [--check] [-l hosts] [-t tags] [--skip-tags t] [-e key=value] [-f forks] [-v]` (exit code 2 when a host failed or was unreachable).

### Inventory

`vars` for every host; `hosts`: name → `{ host, port, user, key, password, become_password, connection: "local", …vars }`; `groups`: name → list of hosts or `{ hosts, vars }`. Patterns in `hosts:` — `all`, a host, a group, `web:db`, `web:!web2`.

```yaml
hosts:
  web1: { host: 192.0.2.10, user: deploy, key: ~/.ssh/id_ed25519 }
  laptop: { connection: local }
groups:
  web: { hosts: [web1], vars: { port: 8080 } }
```

### Playbooks and tasks

Templates: `{{ expr }}` with variables (`item`, `facts`, `inventory_hostname`, `groups`, registered results), `. []`, `+ - * / % ~`, comparisons, `and or not in`, `is defined`, and the filters `default upper lower trim length join first last replace split int float string bool to_json basename dirname`.

- `play: { name, hosts, vars, become, gather_facts = true, tags, tasks, handlers }` — Facts: `facts.os arch hostname distribution distribution_version pkg_mgr service_mgr user`.
- `task keys: name, when, loop, register, notify, ignore_errors, changed_when, check_mode, become, tags` — `when` is an expression; `loop` sets `item`; `register` keeps the result.
- `shell / command: "cmd" | { cmd, chdir, creates, removes, stdin }` — Skipped in check mode (unless `check_mode: false`); `{ rc, stdout, stderr, stdout_lines }`.
- `copy: { content | src, dest, mode } · template: { src, dest, mode }` — Written only when the content or mode differs; `template` renders `{{ }}`.
- `file: { path, state: directory | absent | touch | file, mode }`
- `lineinfile: { path, line, regexp, state, create, mode }` — Replaces the last line matching `regexp`, or appends.
- `package: { name, state: present | absent, update_cache, use }` — apt, dnf, yum, zypper, apk, pacman (from the facts).
- `service: { name, state: started | stopped | restarted | reloaded, enabled }` — systemd.
- `debug: { msg } | { var } · set_fact: { … } · fail: { msg } · wait_for: { port | path, host, timeout }`

```yaml
- name: Web servers
  hosts: web
  become: true
  vars: { app: shop }
  tasks:
    - name: App folder
      file: { path: "/srv/{{ app }}", state: directory, mode: "0755" }
    - name: Config
      template: { src: templates/app.conf, dest: "/srv/{{ app }}/app.conf" }
      notify: restart nginx
    - name: Packages
      package: { name: [curl, nginx], state: present }
      when: facts.os == 'linux'
  handlers:
    - name: restart nginx
      service: { name: nginx, state: restarted }
```

### From Fogu

- `playbook.run(plays | path, { inventory, vars, check, limit, tags, skipTags, forks, verbose, quiet })` → map · error — `{ ok, failed: [hosts], recap: { host: { ok, changed, unreachable, failed, skipped, ignored } } }`.
- `playbook.inventory(path | map)` → map — The hosts and groups it describes.
- `playbook.render(text, vars)` → str · error — The template engine on its own.

```fogu
import playbook from "fogu:playbook"
const plays = [{
    hosts: "all",
    gather_facts: false,
    vars: { dir: "site", pages: ["index", "about"] },
    tasks: [
        { name: "pages", copy: { content: "<h1>{{ item | upper }}</h1>", dest: "{{ dir }}/{{ item }}.html" }, loop: "{{ pages }}" },
        { debug: { msg: "{{ pages | length }} pages in {{ dir }}" } },
    ],
}]
const first = playbook.run(plays)?
const again = playbook.run(plays, { quiet: true })?
print(first.ok, again.recap.localhost.changed)
print(playbook.render("{{ name | default('anon') | upper }}: {{ n * 3 }}", { n: 7 }))
```

output:

```text
PLAY [all] *************************************************************

TASK [pages] ***********************************************************
changed: [localhost] => (item=index)
changed: [localhost] => (item=about)

TASK [debug] ***********************************************************
ok: [localhost] => 2 pages in site

PLAY RECAP *************************************************************
localhost : ok=2    changed=1    unreachable=0    failed=0    skipped=0    ignored=0
true 0
ANON: 21
```

## fogu:vm

```fogu
import vm from "fogu:vm"
```

Runs Fogu source code at run time, in the style of Node's `node:vm`. The code runs on the same interpreter, so its functions can be called from the program and vice versa. A new context only sees the global natives plus the keys of its sandbox map (it may still `import` modules); like `node:vm`, it is **not a security boundary**. The value of the last expression is returned; syntax, name, runtime and timeout errors come back as `error` values.

- `vm.runInNewContext(code, sandbox?, options?)` → value · error — Options: `{ filename = " ", timeout }` or a filename. Variables the code declares are copied into the sandbox map.
- `vm.createContext(sandbox?) · vm.isContext(x)` → VmContext · bool — A context keeps its variables between runs: `ctx.sandbox`, `ctx.name` reads a variable, `ctx.run(code, options?)`.
- `vm.runInContext(code, ctx, options?)` → value · error — Changes to the sandbox map are seen by the next run.
- `vm.runInThisContext(code, options?)` → value · error — Sees the program's globals; its declarations persist between calls.
- `vm.Script(code, options?)` → VmScript · error — Parsed once: `runInContext(ctx) runInNewContext(sandbox?) runInThisContext()`, fields `source filename`.
- `vm.compileFunction(code, params = [], { context, filename })` → fn · error — A function whose body is `code`.
- `timeout` — A duration; loops and calls stop once it passes (`script execution timed out after 50ms`). The JIT is not used while it runs.

```fogu
import vm from "fogu:vm"
print(vm.runInNewContext("x * 2 + y", { x: 20, y: 2 })?)
const ctx = vm.createContext({ count: 0 })
vm.runInContext("count += 1; let msg = `count is ${count}`", ctx)?
print(ctx.sandbox.count, ctx.msg)
const square = vm.Script("n * n")?
print([1, 2, 3].map((n) => square.runInNewContext({ n: n })?))
const add = vm.compileFunction("return a + b", ["a", "b"])?
print(add(2, 3))
const [v, err] = vm.runInNewContext("while true {}", nil, { timeout: 50ms })
print(err)
print(vm.runInNewContext("1 +", nil, "bad.fg"))
```

output:

```text
42
1 count is 1
[1, 4, 9]
5
error: <vm>:1: runtime error: script execution timed out after 50ms
error: bad.fg: line 1:4: syntax error: unexpected expression: Eof
```

## fogu:cli

```fogu
import cli from "fogu:cli"
```

Terminal interaction: prompts (`dialoguer`), colors and terminal control (`console`), spinners and progress bars (`indicatif`), tables (`comfy-table`) and argument parsing (`clap`). Without a terminal on stdin (pipes, CI) the prompts read plain lines, so `echo Ana | fogu app.fg` works; spinners and bars draw on stderr, only on a terminal. Colors are off when stdout is not a terminal or `NO_COLOR` is set.

### Prompts

- `cli.input(prompt, { default, allowEmpty, type: "str" | "int" | "float" })` → str | int | float · error
- `cli.password(prompt, { confirm: "Repeat password", allowEmpty })` → str · error — Hidden input.
- `cli.confirm(prompt, { default })` → bool · error
- `cli.select(prompt, items, { default, fuzzy, index })` → value | nil · error — Items are strings or `{ label, value }`; nil when cancelled with Esc; `fuzzy` filters while typing.
- `cli.multiSelect(prompt, items, { defaults, index })` → list | nil · error — Without a terminal: names or numbers separated by commas.

```fogu
import cli from "fogu:cli"
const name = cli.input("Your name", { default: "Ana" })?
const age = cli.input("Age", { type: "int" })?
const lang = cli.select("Language", ["Fogu", "Rust", { label: "Go lang", value: "go" }])?
if cli.confirm(`Save ${name}?`, { default: true })? { print(name, age, lang) }
```

output (answers piped):

```text
$ printf 'Bia\n30\n3\ny\n' | fogu app.fg
Bia 30 go
```

### Styles, terminal, spinners and tables

- `cli.style(text, ...styles)` → str — `bold dim italic underline strike inverse`, colors `red green yellow blue magenta cyan white black gray`, `brightRed`, backgrounds `bgRed`, 256 colors `"208"` / `"bg208"`.
- `cli.red(text) · green · yellow · blue · magenta · cyan · white · black · gray · bold · dim · italic · underline · strike · inverse` → str
- `cli.colors(on?) · cli.stripAnsi(s) · cli.textWidth(s) · cli.link(text, url)` → bool · str · int · str — `colors(true)` forces colors.
- `cli.isTTY("stdin" | "stdout" | "stderr") · cli.size()` → bool · { columns, rows } | nil
- `cli.clear() · clearLine() · hideCursor() · showCursor() · readKey()` — `readKey` → a character or `Enter Escape Backspace Tab ArrowUp ArrowDown ArrowLeft ArrowRight Home End PageUp PageDown Delete`.
- `cli.spinner(text, { frames })` → Spinner — `text(t) succeed(msg?) fail(msg?) warn(msg?) info(msg?) stop()`.
- `cli.progress(total, { message, prefix, template, chars })` → ProgressBar — `inc(n = 1) set(n) length(n) message(m) println(line) finish(msg?)`, fields `position length elapsed finished`. Templates of `indicatif`.
- `cli.table(rows, { headers, style: "rounded" | "utf8" | "ascii" | "markdown" | "none", align, width })` → str — Rows are lists or maps; numbers are right aligned.

```fogu
import cli from "fogu:cli"
print(cli.red("error"), cli.style("ok", "bold", "green"))
print(cli.table([["ana", 3], ["rui", 15]], { headers: ["name", "jobs"] }))
print(cli.table([{ name: "fogu", stars: 9 }], { style: "markdown" }))
const bar = cli.progress(3)
for i in 0..3 { bar.inc() }
bar.finish()
print(bar.position, cli.stripAnsi(cli.red("x")))
```

output:

```text
error ok
╭──────┬──────╮
│ name ┆ jobs │
╞══════╪══════╡
│ ana  ┆    3 │
│ rui  ┆   15 │
╰──────┴──────╯
| name | stars |
|------|-------|
| fogu |     9 |
3 x
```

### Arguments

- `cli.parse(spec, argv = Fogu.args, { exit = true })` → map · error — `spec`: `{ name, version, about, args: [...], commands: [{ name, about, args }], commandRequired }`. `--help` and `--version` print and exit; a usage error prints and exits with 2, or is an `error` value with `{ exit: false }`.
- `arg: { name, type, short, long, help, positional, required, multiple, default, choices, env }` — Types `str int float bool count`; `multiple` gives a list. A subcommand sets `command` and its values in `args`.

```fogu
import cli from "fogu:cli"
const opts = cli.parse({
    name: "deploy", version: "1.0.0", about: "Deploys the app",
    args: [
        { name: "env", positional: true, required: true, choices: ["dev", "prod"] },
        { name: "force", short: "f", type: "bool", help: "skip the checks" },
        { name: "tag", type: "str", multiple: true },
        { name: "verbose", short: "v", type: "count" },
    ],
    commands: [{ name: "logs", args: [{ name: "lines", short: "n", type: "int", default: 10 }] }],
}, ["prod", "-f", "--tag", "a", "--tag", "b", "-vv", "logs", "-n", "5"])
print(opts)
```

output:

```text
{args: {lines: 5}, command: logs, env: prod, force: true, tag: [a, b], verbose: 2}
```
