# t

`t` is a command-line todo list manager for people that want to *finish* tasks, not organize them.

## Why t?

Yeah, I know, *another* command-line todo list manager. Several others already exist ([todo.txt][] and [TaskWarrior][] come to mind), so why make another one?

[todo.txt]: http://ginatrapani.github.com/todo.txt-cli/
[TaskWarrior]: http://taskwarrior.org/projects/show/taskwarrior/

### It does the simplest thing that could possibly work

Todo.txt and TaskWarrior are feature-packed. They let you tag tasks, split them into projects, set priorities, order them, color-code them, and much more.

**That's the problem.**

It's easy to say "I'll just organize my todo list a bit" and spend 15 minutes tagging your tasks. In those 15 minutes you probably could have *finished* a couple of them.

`t` was inspired by [j][]. It's simple, messy, has almost no features, and is extremely effective at the one thing it does. With `t` the only way to make your todo list prettier is to **finish some damn tasks**.

[j]: http://github.com/rupa/j2/

### It's flexible

`t`'s simplicity makes it extremely flexible.

Want to edit a bunch of tasks at once? Open the list in a text editor.

Want to view the lists on a computer that doesn't have `t` installed? Open the
list in a text editor.

Want to synchronize the list across a couple of computers? Keep your task
lists in a [Dropbox][] folder.

Want to use it as a distributed bug tracking system like [BugsEverywhere][]?
Make the task list a `bugs` file in the project repository.

[Dropbox]: https://www.getdropbox.com/
[BugsEverywhere]: http://bugseverywhere.org/

### It plays nice with version control

Other systems keep your tasks in a plain text file. This is a good thing, and `t` follows their lead.

However, some of them append new tasks to the end of the file when you create them. This is not good if you're using a version control system to let more than one person edit a todo list. If two people add a task and then try to merge, they'll get a conflict and have to resolve it manually.

`t` uses random IDs (actually SHA1 hashes) to order the todo list files. Once the list has a couple of tasks in it, adding more is far less likely to cause a merge conflict because the list is sorted.

## Installing t

`t` requires [Python][] 2.5 or newer, and some form of UNIX-like shell (bash
works well). It works on Linux, OS X, and Windows (with [Cygwin][]).

[Python]: http://python.org/
[Cygwin]: http://www.cygwin.com/

Installing and setting up `t` will take about one minute.

First, clone the newest version ([original] or [this fork]) and put it anywhere you like.

[original]: https://github.com/sjl/t
[this fork]: https://github.com/sheriferson/t

Next, decide where you want to keep your todo lists. I put mine in `~/tasks`. Create that directory:

    mkdir ~/tasks

Finally, set up an alias to run `t`. Put something like this in your `~/.bashrc` file:

    alias t='python ~/path/to/t.py --task-dir ~/tasks --list tasks'

Make sure you run `source ~/.bashrc` or restart your terminal window to make the alias take effect.

## Using t

`t` is quick and easy to use.

### Add a task

To add a task, use `t [task description]`:

```bash
$ t Clean the apartment.
$ t Write chapter 10 of the novel.
$ t Buy more beer.
$
```

### List your tasks

Listing your tasks is even easier -- just use `t`:

```bash
$ t
9  - Buy more beer.
30 - Clean the apartment.
31 - Write chapter 10 of the novel.
$
```

`t` will list all of your unfinished tasks and their IDs.

### Mark a task for today

_Note: This is a feature added in this fork and not available in the original project._

To mark a task for today (by appending `@today` as a suffix tag to the task[^1]), use `t -t ID`
(note that the `-t` operator used to be short for `--task-dir`, which has now been changed to `-k`)

```bash
$ t -t 9
📅 Buy more beer @today
$
```

Using the `-t` operator on a task that already has `@today` as a suffix will umark it as a today task:

```bash
$ t -t 9
❌ 📅  Buy more beer
$
```

I find it helpful to mark a few tasks for "today", and then list only those tasks using a `today` alias set up in my `.zshrc` (or your `.bashrc`) file:

    alias today='t -g @today'

The alias runs t with the `-g` operator that runs a grep search for the `@today` string in all tasks. This _will_ return a task that has `@today` somewhere in the middle, but that's just not something that will ever exist in any of my tasks.

### Mark a task as what I'm doing now

_Note: This is a feature added in this fork and not available in the original project._

It's useful to focus on one task at a time. To formalize this, one can mark that task by adding `@now` using the `-n` switch.

```bash
$ t Finish updating t README
$ t
...
2c - Finish updating t README
...
$ t -n 2c
$ 🎯 Finish updating t README @now
```

Using the switch again will remove the `@now` tag:

```bash
$ t -n 2c
$ ❌ 🎯 Finish updating t README
```

### Add a date to a task

_Note: This is a feature added in this fork and not available in the original project._

If you tag a task with a date in the future (e.g., `Turn on AI Sherif @2018-08-01`), `t` will convert the date tag to `@today` on the day of.

```bash
$ t Turn on AI Sherif @2018-08-01
$ t
d - Turn on AI Sherif @2018-08-01
$ # on August 1, 2018...
$ t
d - Turn on AI Sherif @today
```

### Finish a task

After you're done with something, use `t -f ID` to finish it:

```bash
$ t -f 31
$ t
9  - Buy more beer.
30 - Clean the apartment.
$
```

When finishing a task, `t` will remove the `@today` tag if it exists.

### Edit a task

Sometimes you might want to change the wording of a task.  You can use `t -e ID [new description]` to do that:

```bash
$ t -e 30 Clean the entire apartment.
$ t
9  - Buy more beer.
30 - Clean the entire apartment.
$
```

Yes, nerds, you can use sed-style substitution strings:

```bash
$ t -e 9 /more/a lot more/
$ t
9  - Buy a lot more beer.
30 - Clean the entire apartment.
$
```

### Delete the task list if it's empty

If you keep your task list in a visible place (like your desktop) you might want it to be deleted if there are no tasks in it. To do this automatically you can use the `--delete-if-empty` option in your alias:

    alias t='python ~/path/to/t.py --task-dir ~/Desktop --list todo.txt --delete-if-empty'

## Tips and Tricks

`t` might be simple, but it can do a lot of interesting things.

### Count your tasks

Counting your tasks is simple using the `wc` program:

```bash
$ t | wc -l
      2
$
```

### Put your task count in your bash prompt

Want a count of your tasks right in your prompt? Edit your `~/.bashrc` file:

    export PS1="[$(t | wc -l | sed -e's/ *//')] $PS1"

Now you've got a prompt that looks something like this:

```bash
[2] $ t -f 30
[1] $ t Feed the cat.
[2] $
```

### Multiple lists

`t` is for people that want to *do* tasks, not organize them. With that said, sometimes it's useful to be able to have at least *one* level of organization. To split up your tasks into different lists you can add a few more aliases:

    alias g='python ~/path/to/t.py --task-dir ~/tasks --list groceries'
    alias m='python ~/path/to/t.py --task-dir ~/tasks --list music-to-buy'
    alias w='python ~/path/to/t.py --task-dir ~/tasks --list wines-to-try'

### Distributed Bugtracking

Like the idea of distributed bug trackers like [BugsEverywhere][], but don't
want to use such a heavyweight system? You can use `t` instead.

Add another alias to your `~/.bashrc` file:

    alias b='python ~/path/to/t.py --task-dir . --list bugs'

Now when you're in your project directory you can use `b` to manage the list of bugs/tasks for that project. Add the `bugs` file to version control and you're all set.

Even people without `t` installed can view the bug list, because it's plain text.


Problems, Contributions, Etc
----------------------------

`t` was hacked together in a couple of nights to fit my needs. If you use it
and find a bug, please let me know.

If you want to request a feature feel free, but remember that `t` is meant to be simple. If you need anything beyond the basics you might want to look at [todo.txt][] or [TaskWarrior][] instead. They're great tools with lots of bells and whistles.

If you want to contribute code to `t`, that's great! Fork the [Mercurial repository][] on BitBucket or the [git mirror][] on GitHub and send me a pull request.

[Mercurial repository]: http://bitbucket.org/sjl/t/
[git mirror]: http://github.com/sjl/t/

[^1]: This tagging system inspired by [Taskpaper](!g "taskpaper app").
