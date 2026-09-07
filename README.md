# C++ To-Do CLI

[![CI](https://github.com/ItsMrZxD/cpp-todo-cli/actions/workflows/ci.yml/badge.svg)](https://github.com/ItsMrZxD/cpp-todo-cli/actions/workflows/ci.yml)
[![C++11](https://img.shields.io/badge/C%2B%2B-11-blue.svg)](https://en.cppreference.com/w/cpp/11)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

**A dependency-free command-line to-do list manager written in C++ — a
menu-driven terminal task manager that persists your tasks to a plain text file
between runs. Single source file, standard library only.**

Everything runs in the terminal behind a numbered menu: add a task, list tasks,
mark one complete, delete one, quit. Tasks are saved to `tasks.txt` when you
exit and reloaded on the next run, so the list survives between sessions. The
whole program is one `main.cpp` with no external libraries, which makes it a
compact reference for file I/O, input validation, and `std::vector` handling in
C++ — and a small, self-contained utility if you just want a to-do list in a
terminal.

## Features

- Numbered menu interface — add, list, mark complete, delete, quit
- **Persistent storage** — tasks are saved to `tasks.txt` and reloaded on start
- Tasks display as `[ ]` / `[x]` with a done/pending summary line
- **Robust input handling** — non-numeric input, out-of-range choices, and
  empty task descriptions are rejected with a message rather than crashing
- A missing, empty, or partially malformed `tasks.txt` degrades to an empty
  list instead of failing on first run
- No external libraries — the C++ standard library only
- Builds and tested on both Linux and Windows in CI

## Requirements

- A C++11-capable compiler. Developed against **g++ (MinGW GCC 6.3.0)** and
  built in CI with g++ on both `ubuntu-latest` and `windows-latest`.
- Nothing else — no CMake, no package manager, no third-party headers.

## Build

From the project folder:

```sh
g++ -std=c++11 -Wall -O2 -o todo main.cpp
```

That produces `todo` (or `todo.exe` on Windows). CI builds with
`-Wall -Wextra -O2`, and on Windows adds `-static` so the binary does not pick
up a mismatched `libstdc++` DLL from the runner's PATH.

## Run

```sh
./todo
```

You will see the menu:

```
  ============================================
                MY TO-DO LIST
  ============================================
  Loaded 2 task(s) from tasks.txt.

  --------------------------------------------
    1.  Add task
    2.  List tasks
    3.  Mark task complete
    4.  Delete task
    5.  Quit
  --------------------------------------------
  Choose an option (1-5):
```

Listing tasks looks like this:

```
  Your tasks:
  --------------------------------------------
   1. [ ] Buy milk
   2. [x] Walk the dog
  --------------------------------------------
  Total: 2  |  Done: 1  |  Pending: 1
```

The program is interactive and takes no command-line flags or arguments — all
actions are chosen from the menu.

### Scripted use

Because it reads from standard input, you can drive it non-interactively. This
adds a task, lists the tasks, and quits:

```sh
printf '1\nBuy milk\n\n2\n\n5\n' | ./todo
```

(The blank lines answer the "press Enter to continue" prompts.) This is exactly
the end-to-end smoke test CI runs against the real binary.

## Data format

Tasks are stored in `tasks.txt`, one per line, as `doneFlag|description`:

```
0|Buy milk
1|Walk the dog
```

- `doneFlag` is `1` for completed tasks and `0` otherwise.
- Everything after the **first** `|` is the description, so descriptions may
  themselves contain `|` characters without corrupting the file.
- The file is written when you quit, and is git-ignored since it is your
  personal data.

## Tests

```sh
g++ -std=c++11 -Wall -Wextra -O2 -o run_tests tests/test_main.cpp
./run_tests
```

The unit tests cover the input-parsing helpers and the save/load round trip,
including descriptions containing `|` and malformed lines in the data file. CI
additionally builds the real binary and drives a scripted add/list/quit session
through it on both Linux and Windows.

## Limitations

- **`tasks.txt` is read from and written to the current working directory**, so
  running the binary from a different directory gives you a different task
  list. Run it from a consistent location, or keep the binary and its
  `tasks.txt` together.
- Tasks are saved on quit. Exiting with Ctrl+C instead of menu option 5 loses
  changes made in that session.
- No due dates, priorities, tags, or search — the task model is a description
  and a done flag.
- No undo, and deletion is immediate.
- Single-user and single-file; there is no locking, so two copies running
  against the same `tasks.txt` will overwrite each other.

## License

MIT — see [LICENSE](LICENSE).
