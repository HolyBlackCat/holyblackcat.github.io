---
layout: post
title: "Escaping <code>CreateProcess()</code> arguments on Windows"
tags: C++ windows winapi
toc: true
---

## TL;DR:

On Windows, to start a process, you have to pass the executable and its arguments to WinAPI as a single string, escaped and quoted in a very particular way. I got [nerd-sniped](https://xkcd.com/356/) into researching this.

Messing up the escaping can lead to [vulnerabilities](https://flatt.tech/research/posts/batbadbut-you-cant-securely-execute-commands-on-windows/).

This post explains how to do the escaping and quoting correctly. [Skip to the escaping algorithm.](#the-escaping-algorithm)

## Errata

This was updated on 2026-09-06 to cover the [custom quoting rules](#special-quoting-rules-of-cmd-c) of `cmd /c` and fix minor mistakes.

Minor changes not affecting the escaping algorithm may not be listed here.

## Intro

As you may know, unlike on POSIX (Linux, etc), on Windows the command line arguments (`argv`) are internally represented as a single long string. There is a WinAPI function to split it to an `argv`-style array, and the C runtime splits it for you when calling your `main()`, but you can still access the original combined string, and interpret it differently if you want. Most applications respect the stock split.

Starting a new process (`CreateProcess()`) requires you to provide a single string, not an array.

So when you run `./my_program arg1 arg2` in a shell on Windows (at least in CMD, the legacy Windows shell; might no longer be true in PowerShell), the entire line goes straight to `CreateProcess()` with minimal changes. Compare this to POSIX, where it's the shell's job to split the line on spaces/quotes and produce an `argv` array.

WinAPI (in its infinite wisdom) [doesn't provide](#existing-wrapper-functions) the reverse operation, a function to combine an argument array into one string. You have to implement it yourself, and there's no clear documentation on how to do it. This post explains how to do it.

There's no new knowledge here, it's all been explained and done before, just not in one place. I couldn't find good documentation covering everything.

If you find any inaccuracies in the post, please report them to [my github](https://github.com/HolyBlackCat/holyblackcat.github.io).

Main sources:

* [Documentation on `CreateProcessW()`](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessw) - Not very detailed.

* ["Everyone quotes command line arguments the wrong way"](https://learn.microsoft.com/en-us/archive/blogs/twistylittlepassagesallalike/everyone-quotes-command-line-arguments-the-wrong-way) - Explains basic escaping, but doesn't correctly explain how to deal with batch files.

* ["BatBadBut: You can't securely execute commands on Windows"](https://flatt.tech/research/posts/batbadbut-you-cant-securely-execute-commands-on-windows/) - Has some clever suggestions about escaping `%` for batch files, but is wrong about some other things (e.g. says that `CreateProcess()` implicitly adds the `.bat` extension). I think their idea of escaping `%` is ultimately not reliable, more on that below.

* [Zig PR](https://github.com/ziglang/zig/pull/19698) implementing and explaining most of this.

## Basic use of `CreateProcess()`

### Test snippets

Here's a minimal program calling `CreateProcess()`:
```cpp
#include <windows.h>
#include <iostream>

int main()
{
    STARTUPINFOW si{};
    si.cb = sizeof(si);

    PROCESS_INFORMATION out{};

    const wchar_t *app = nullptr;    // You can change this!
    wchar_t command[] = L"calc.exe"; // This too!

    if (CreateProcessW(app, command, nullptr, nullptr, false, 0, nullptr, nullptr, &si, &out))
    {
        std::cout << "ok\n";
        CloseHandle(out.hProcess);
        CloseHandle(out.hThread);
    }
    else
    {
        std::cout << "not ok\n";
    }
}
```
And here's a handy program that prints `argv`. No unicode support by default, see below.
```cpp
#include <iostream>

int main(int argc, char **argv)
{
    for (int i = 0; i < argc; i++)
        std::cout << "argv[" << i << "] = `" << argv[i] << "`\n";
}
```
Side note: By default the narrow `char **argv` is unusable for anything serious because of the missing unicode support. You should either enable UTF-8 support for it through [the manifest](https://learn.microsoft.com/en-us/windows/apps/design/globalizing/use-utf8-code-page) (if you're not a library developer and can control the manifest), or use `CommandLineToArgvW(GetCommandLineW(), ...)` or `int wmain(int argc, wchar_t **argv)` to get UTF-16 encoded `argv`, and then [convert it to UTF-8 if needed](#unicode). If you don't do any of this and keep the default behavior, then in addition to missing unicode support you may also get vulnerabilities, see ["WorstFit: Unveiling Hidden Transformers in Windows ANSI!"](https://devco.re/blog/2025/01/09/worstfit-unveiling-hidden-transformers-in-windows-ansi/#-argument-splitting).

### Unicode

See: [Unicode in the Windows API](https://learn.microsoft.com/en-us/windows/win32/intl/unicode-in-the-windows-api) and [Use UTF-8 code pages in Windows apps](https://learn.microsoft.com/en-us/windows/apps/design/globalizing/use-utf8-code-page).

There are `CreateProcess()`, `CreateProcessA()`, and `CreateProcessW()`. You probably want to use `CreateProcessW()`.

Like many other WinAPI functions, `CreateProcess()` has two versions: `CreateProcessW()` that uses UTF-16 and `CreateProcessA()` that uses the currently active code page (so a narrow string, but usually not UTF-8). `CreateProcess()` is a macro that's defined to one or the other (depending on `#define UNICODE`, but that doesn't matter).

By default `CreateProcessA()`, is basically unusable for anything serious, because it can't deal with unicode, and can only handle a very limited set of symbols from the active code page. You can fix that by setting the active code page to UTF-8 ([e.g. through the manifest](https://learn.microsoft.com/en-us/windows/apps/design/globalizing/use-utf8-code-page), or should be possible at runtime too), but if you're a library, asking the users to do that isn't nice.

`CreateProcess()` (without `A`/`W`) and `#define UNICODE` are even more useless. Having code that compiles both with narrow and wide strings depending on a macro is a lot of work for little benefit, just choose one.

Since the UTF-16 is what's actually used under the hood, you usually want to use the UTF-16 version, `CreateProcessW()`. If your strings are UTF-8 (as they should be), convert them to UTF-16 using [`MultiByteToWideChar()`](https://learn.microsoft.com/en-us/windows/win32/api/stringapiset/nf-stringapiset-multibytetowidechar) or some other method.

Also consider using [WTF-8](https://wtf-8.codeberg.page) instead of UTF-8 to store the original strings. Some invalid UTF-16 strings (that can nonetheless appear in file paths) are impossible to represent as UTF-8, but you can trivially extend it to fix it (which is what WTF-8 is). From what I understand, using the narrow `CreateProcessA()` (with UTF-8 codepage) makes those impossible to support, so might not be the best idea.

### Arguments of `CreateProcess()`

[`CreateProcess()`](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessw) takes two strings (the first two arguments). For the `CreateProcessW()` version, they are pointers to `wchar_t`, so you can store them in `std::wstring`. (And for `CreateProcessA()` they are pointers to `char`, see [the section about unicode in WinAPI](#unicode).)

The signature is `(const wchar_t *lpApplicationName, wchar_t *lpCommandLine, ....)`.

* `lpApplicationName` is usually not needed, you should pass `nullptr`.
* `lpCommandLine` is the executable, followed by its arguments, space-separated.

The function can modify the second string. It's unspecified what it does with it, the important part is that the string you pass can't be const. (This is only true for the `CreateProcessW()` variant. [`CreateProcessA()`](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessa) still takes a non-const `char *`, but promises to not modify it, so there you can `const_cast`.)

If `lpApplicationName` isn't null, it's used instead of the first part of `lpCommandLine` as the executable path. But the first part of `lpCommandLine` still ends up passed to `argv[0]` of the new process, which by convention should match the executable path. So this is the same as what POSIX does: `argv[0]` usually matches the executable name, but is not required to.

But `lpApplicationName` uses a different search mechanism: unlike `lpCommandLine` it doesn't respect `PATH` and doesn't allow the `.exe` extension to be omitted. If the path is relative, it searches relative to the current directory (which `lpCommandLine` does too, unlike on POSIX). If `lpApplicationName` starts with `\`, it searches on the current drive letter, so usually `\foo\bar.exe` -> `C:\foo\bar.exe`.

If `lpApplicationName` isn't null but `lpCommandLine` is null, it normally just becomes the quoted version of `lpApplicationName` (so `lpApplicationName` gets passed to `argv[0]`; but also check out [this fun bug](#trailing-garbage-in-executable-name)).

So, in short: to get sane behavior, you usually don't want to pass `lpApplicationName`, only pass `lpCommandLine`.

### Existing wrapper functions

Windows does have `_exec()`/`_spawn()`/etc which accept arguments as separate strings.

But as documented [here](https://learn.microsoft.com/en-us/archive/blogs/twistylittlepassagesallalike/everyone-quotes-command-line-arguments-the-wrong-way#the-c-runtime-library-is-useless),  they are useless. They don't do any escaping and just combine the arguments with spaces, which leads to wrong results.

## Basic escaping

This method is sufficient if you're running an executable and [not a batch file](#how-to-check-if-its-a-batch-file), more on that below. I.e. this is fine if you hardcode your program name and it doesn't include `.bat` or `.cmd`.

`lpCommandLine` is a space-separated list of arguments, the first of which is the program name.

If an argument (or the program name) is empty or contains spaces or tabs (<code> </code>, `\t`), it must be quoted with `"`. Unnecessary quoting doesn't do any harm.

It's safer to always quote the program name even if it contains no spaces, [more on that later](#quoting-the-executable-name).

Existing `"` can be quoted either as `""` or as `\"`. If you use `""`, then the entire argument must be quoted even if it doesn't contain spaces or tabs (otherwise `""` becomes an empty string, this is because partially quoting an argument is allowed too, but it's not useful to us).

For example, `foo"bar"` can be escaped as either `foo\"bar\"` or `"foo""bar"""`.

In most cases there seems to be no difference between `""` and `\"`, but for [batch files](#batch-files) only `""` works correctly, so it's easier to always use `""` instead of `\"`. [More on that below.](#basic-batch-escaping)

Existing `\` must be escaped as `\\` **ONLY IF** it is followed by 0 or more `\` and then a `"` (if you quoted the whole argument, your own trailing quote does count for this). In other words, if you encounter one or more `\` followed by a quote, you output twice as many `\`, and then the escaped quote as usual (or not escaped if it's your own closing quote at the end of argument).

Some examples:

Input|Output|Alternative output|Comment
---|---|---|---
empty|`""`||Empty, must quote.
`foo`|`foo`
`foo bar`|`"foo bar"`||Space, must quote.
`foo↹bar`|`"foo↹bar"`||Here `↹` means a tab character. Must quote that.
`foo"bar`|`"foo""bar"`|`foo\"bar`|Can escape quote either as `""` (preferred) or `\"`. If using `""`, must quote the entire argument.
`foo" bar`|`"foo"" bar"`|`"foo\" bar"`|Same, but must quote the entire argument because of a space.
`foo\bar`|`foo\bar`||Slashes not followed by `"` are not escaped.
`foo\\bar`|`foo\\bar`||^
`foo\\\bar`|`foo\\\bar`||^
`foo\"bar`|`"foo\\""bar"`|`foo\\\"bar`|N `\` followed by `"` become N\*2 `\` followed by escaped quote.
`foo\\"bar`|`"foo\\\\""bar"`|`foo\\\\\"bar`|^
`foo\\\"bar`|`"foo\\\\\\""bar"`|`foo\\\\\\\"bar`|^
`foobar\`|`foobar\`||Trailing `\` are not escaped if the argument is not quoted.
`foobar\\`|`foobar\\`||^
`foobar\\\`|`foobar\\\`||^
`foo bar\`|`"foo bar\\"`||Trailing `\` are all escaped if the argument is quoted. The closing quote itself it not escaped, regardless of `\` before it.
`foo bar\\`|`"foo bar\\\\"`||^
`foo bar\\\`|`"foo bar\\\\\\"`||^

## Batch files

### The problem

Batch files have extensions `.bat` or `.cmd`. They are shell scripts for the legacy Windows shell called CMD (as opposed to the new PowerShell).

If given a batch file, `CreateProcess()` will automatically run it with `cmd`.

CMD is dumb. Unlike Bash, it doesn't have syntax for escaping variable expansions. Here's what I mean:

Imagine you're writing a shell script that calls another program and forwards `argv` to it. That would be:

* In Bash:
  ```bash
  my_program "$@"
  ```
* In CMD: (I'll call this `foo.bat` below.)
  ```bat
  my_program %*
  ```

The Bash version propagates the arguments literally, as is. Despite how the quotes make it look, it will correctly pass multiple arguments as separate arguments.

The CMD version will not only expand env variables in the arguments, **it will also run arbitrary commands for you**. Try running `foo.bat &calc.exe`, and `my_program %*` will expand to `my_program &calc.exe`, with `&` being interpreted as a command separator. That will run `calc.exe`.

Before you ask, quoting `"%*"` doesn't fix it. (Funnily enough, removing quotes from the Bash example still doesn't allow those exploits. The only thing it seems to change is to further split `argv` on spaces embedded in the arguments.)

Quoting the argument `foo.bat "&calc.exe"` does fix it though (in this simple case).

That's right, **CMD relies on the caller to pre-escape the arguments correctly.** If your arguments are user-provided and you escape them wrong, the user can run arbitrary executables.

### Basic batch escaping

Batch has two escaping mechanisms:

1. **`"..."` quotes.** They:

   * Allow spaces in arguments.
   * Disable special character handling, like in `"&calc.exe"`.
   * **Don't** disable `%BLAH%` env variable expansion, though.

   The quotes are not removed when evaluated. They are propagated to the child process as is, which is convenient, since `CreateProcess()` also uses them for spaces in the arguments. In other words, `my_program %*`, if given `"foo bar"`, passes `my_program "foo bar"` to `CreateProcess()`, which is exactly what we want.

   Notably `\"` doesn't work in batch quotes. It's considered to end the string. It means two things:

   * Running the batch file as `foo.bat "foo\" &calc.exe"` is going to run `calc.exe`, because the quotes only escape the `"foo\"` part.

   * It **also** prevents the arguments from splitting correctly. Batch provides `%1`, `%2`, etc, to access its `argv[i]`, and it must be using some custom splitting logic (or an older version of the C runtime?), because `"foo\" bar"` is treated as two arguments, not one. This only matters when using `%1`, `%2`, and not for `%*`, since that just expands to one long string, and doesn't care about batch argument splitting logic.

   `""` does work correctly though, which is why I recommended it earlier.

2. **`^`**. You can prepend it to a special character to escape it.

   `^` itself can be escaped as `^^`. There's no funky behavior [like with `\` from earlier](#basic-escaping), you can always double `^` to escape it. Redundantly prepending `^` to a character with no special meaning just removes the `^`.

   Batch files expand the arguments **twice** though, once on startup, and again when using them in a command. So running `foo.bat ^&calc.exe` still runs `calc.exe` (on the second expansion), while `foo.bat ^^^&calc.exe` (double escaping) works correctly and passes `&calc.exe` literally to `my_program`.

   You're not observing double expansion with quotes because they expand to themselves, so you can't notice how many times the expansion happens with those.

   Like quotes, `^` **don't** disable env variable expansion. It may look like they do, since `^%FOO^%` becomes `%FOO%` and doesn't expand `FOO`. But that stops working if you make `FOO^` an env variable. `^%FOO^%` in fact expands `FOO^` if it exists. (CMD preserves `%X%` literally if `X` is not an env variable, unlike Bash, which replaces missing variables with empty strings by default.) So if in CMD you do `set FOO^^=^&calc.exe` (which sets `FOO^` to `&calc.exe`), then run your batch file as `foo.bat ^%FOO^%`, it's going to expand and run `calc.exe`.

The fun continues!

1. **`^` can escape `"`**, removing their magical properties (but still producing the quote). So while passing `foo.bat "&calc.exe"` is safe, `foo.bat ^"&calc.exe^"` will instead run `calc.exe`.

   You might then assume that `foo.bat ^"foo" &calc.exe"` is safe, but it too runs `calc.exe`, and so does `foo.bat "foo^" &calc.exe"`. Meaning that even the escaped quotes update the "are we in a quoted string" flag. It seems `^` only matters on the opening quote, and is ignored on the closing quote.

   Note that escaped quotes still affect argument splitting, so `^"foo bar^"` is considered a single argument by `%1`. (And of course by `CreateProcess()` too, since it doesn't know about `^`; and `^` should get removed by the time `CreateProcess()` is called anyway, unless you escape it twice). Similarly, escaping a space does nothing and doesn't affect splitting: `foo^ bar` is two arguments.

   I assume this is because argument splitting runs as a separate pass, between the first and second expansion.

2. And conversely, **quotes disable `^`** (if the quotes themselves are not escaped with `^`).

   Quoted `^` are preserved literally. So `foo.bat "^&calc.exe"` is safe but passes the wrong string (`^&calc.exe` instead of `&calc.exe`).

   `^"^&calc.exe^"` is safe, but it's no different than `"&calc.exe"`. It literally becomes that after the first expansion, and the second expansion sees `"&calc.exe"`.

   `^^^"^^^&calc.exe^^^"` is also safe, and is again no different than `"&calc.exe"` after the two expansions.

Some sources recommend quoting arguments as `^"...^"`, with every special character inside escaped with `^`. But as we established, that's just `"..."` with more steps (since `^"...^"` will become `"..."` after the first expanding pass).

### Batch files running other batch files

One batch file can call another. In several different ways:

1. Naively running `foo.bat %*` from another batch file behaves like `exec` - it runs that file and then stops, instead of returning to the previous batch file.
2. `call foo.bat %*` behaves as you'd expect (goes back to the previous batch file after executing that one).
3. `cmd /c foo.bat %*` behaves similar to `call ...` (with some extra expanding).

All those can introduce additional expansion passes (in addition to the two happening normally), therefore **any escaping strategy relying on `^` is unreliable**. You'd have to tune the amount of `^` for what the batch file is going to do with the argument. This is unlike `"..."`, which are left unchanged by repeated expansion.

For example, with those batch files:
* `foo2.bat`:
  ```bat
  foo.bat %*
  ```
* `foo3.bat`:
  ```bat
  call foo.bat %*
  ```
* `foo4.bat`:
  ```bat
  cmd /c foo.bat %*
  ```

`foo.bat ^^^&calc.exe` works fine (passes `&calc.exe` literally), but `foo2.bat ^^^&calc.exe` starts `calc.exe`.

`foo2.bat ^^^^^^^&calc.exe` works fine, but `foo.bat ^^^^^^^&calc.exe` ends up passing `^&calc.exe`.

Interestingly, `foo3.bat ^^^&calc.exe` just refuses to execute, and so does `foo3.bat ^^^^^^^&calc.exe`. Not entirely sure why.

`foo4.bat` needs `^^^^^^^^^^^^^^^&calc.exe`.

### Escaping `%`

As mentioned [before](#basic-batch-escaping), it's impossible to prevent `%FOO%` variable expansion, neither with quotes (no effect), nor with `^` (`^%FOO^%` doesn't expand `FOO` but does expand `FOO^`).

We can't just ignore this, because you can run arbitrary commands with those, even without needing custom env variables.

[BatBadBut](https://flatt.tech/research/posts/batbadbut-you-cant-securely-execute-commands-on-windows/) shows the following trick: `foo.bat "%CMDCMDLINE:~-1%&calc.exe"`. This runs `calc.exe` despite being quoted. The `CMDCMDLINE` variable returns the `argv` of the process running the script (which is `foo.bat "...."` in this case). The `%  :~-1%` part extracts the last character from it, which is a `"`, which lets this break out from the existing quotes to allow `&` to work. (While you can add whitespace to the end of the command to work around this specific evil argument, the attacker can then change the substring offset to the new position of the quote.)

The same article suggests a clever escaping method for `%`: `%%cd:~,%`. This produces a single `%`.

The `%cd:~,%` part returns an empty string (`%  :~,%` extracts an empty substring from variable `cd` that stores the current directory). And then the preceding `%` is left as is.

This is cool, but problematic for two reasons:

1. As the article itself mentions, it relies on the "command extensions" setting being on. It's on by default, but the user can change it in the registry.

   This can be worked around by prepending `cmd /d /e:on /s /c ` to `foo.bat` when forming the `CreateProcess()` argument. `/e:on` force enables the extensions, and `/d` disables running startup scripts specified in the registry for a good measure, and [see this for `/s`](#special-quoting-rules-of-cmd-c).

2. Not mentioned in the article: Similar to `^`, it can misbehave if the batch file calls another one.

   `foo2.bat "%%cd:~,%CMDCMDLINE:~-1%%cd:~,%&calc.exe"` works fine (passes `&calc.exe` literally),<br/>
   `foo3.bat "%%cd:~,%CMDCMDLINE:~-1%%cd:~,%&calc.exe"` refuses to execute for an unknown reason, and<br/>
   `foo4.bat "%%cd:~,%CMDCMDLINE:~-1%%cd:~,%&calc.exe"` manages to run `calc.exe` (and would need a double escape of `%`).

    Note that, interestingly, `%` are expanded less times than other special symbols (just once per batch file instead of twice like `^`, and not expanded again for `foo2.bat`).

IMO, the trouble is not worth it. `%%cd:~,%` works in some cases but not all, and ultimately who cares about passing `%` to batch files? It's easier to reject any arguments that contain `%`.

### Escaping `!`

`!` does nothing by default, but if you enable a feature called "delayed expansion", it starts behaving similar to `%`, and so is similarly problematic.

Delayed expansion is off by default, but can be enabled with a registry key (or with a certain command in the middle of a batch file, but this doesn't affect us; or with `cmd /v:on`). It can be force disabled by prepending `cmd /d /v:off /s /c ` to the command line, similar to how we force-enabled command extensions in the previous section (or both at the same time: `cmd /d /e:on /v:off /s /c `).

Since customizing those registry keys is evil, and passing `!` might be useful, I think force prepending `cmd /d /e:on /v:off /s /c ` to batch files is a good idea. Alternatively we'd have to ban `!` in arguments, like `%`.

Before you ask, `!!cd:~,!` doesn't seem to work. `cmd /d /v:on /e:on /c foo.bat "!!cd:~,!PATH!!cd:~,!"` does print the expanded `PATH`.

### What characters need to be banned in batch arguments?

In addition to `%` (and optionally `!` if you don't want to prepend `cmd ... /v:off ...` to batch files), we also need to ban:

* `\n` (line break) in batch arguments - it silently deletes itself and everything after in the command line.

* `\r` (carriage return) in batch arguments - it silently deletes itself.

`\n`,`\r` don't seem to be harmful, just weird. Since there's no valid reason to pass them, I'd error on them.

Those need to be banned in the batch file name too, since when it's passed to CMD, it undergoes the same expansion as everything else. E.g. trying to run `foo%FOO%bar.bat` with `FOO=42` will actually run `foo42bar.bat`. (If both `lpApplicationName` and `lpCommandLine` are specified, then `lpApplicationName` is not passed to CMD and doesn't need to be validated.)

Lastly, in all strings check for `\0`, if your language allows them. (For obvious reasons: `CreateProcess()` needs null-terminated strings.)

### What characters need to be quoted in batch arguments?

What characters need to be quoted? (That's different from characters that need [`^` escaping](#basic-batch-escaping), but we [established](#batch-files-running-other-batch-files) that quotes are the superior form of escaping.)

For batch files, we quote the argument if it contains any of: <code> </code> spaces, `\t` tabs, `"` quotes, or any of ``<>&|()[]{}^=;!'+,`~``. This list is approximate, some characters might not be needed. I'm certain about:

* Spaces, tabs, and `"`, since they're argument separators.
* `;,=`, since they are apparently also argument separators in batch?! (For `%1`, `%2`, etc.)
* `<>&|` because they have obvious special meaning (redirection, separating commands).
* `^` because it's the escape character.

Less certain about the rest. [This article](https://learn.microsoft.com/en-us/archive/blogs/twistylittlepassagesallalike/everyone-quotes-command-line-arguments-the-wrong-way) claims those to be special characters: `()%!^"<>&|` (not necessarily quotable). I'm excluding `%` because it ignores quotes. I'd also exclude `!`, but it appears in another list I'll mention in a moment. I have no idea what special effect `()` have.

`cmd /?` has another character list in it: `` &()[]{}^=;!'+,`~``. It's mentioned in a slightly different context, so maybe some of those don't need to be quoted in arguments. They omit `<>|` because they're invalid in filenames (and this list is mentioned in the context of filenames), but we clearly need to quote them. They mention `!` despite not mentioning `%`, so I'll quote it just in case. Not sure why ``()[]{}'+`~`` are listed, but I guess it's safer to quote them too.

### Special quoting rules of `cmd /c`

One more thing. Turns out `cmd ... /c "..."` has a custom behavior of removing the quotes from the rest of the string after `/c` under certain conditions, before doing anything else.

This can happen if the next thing after `/c` (after whitespace) is a `"`. Then it may remove that quote, and remove the final quote in the command (which doesn't have to be at the end of the command).

For example, if given `cmd /c "foo.bat" "&calc.exe"`, it removes the outer quotes and resolves to `foo.bat" "&calc.exe`, which runs `calc.exe`.

This can't be disabled, so we have to lean into it and **always quote the rest of the command after `/c`**.

There is some convoluted corner case where this behavior gets disabled (see `cmd /?`), but we don't want to deal with that, so we **pass `/s`** to get rid of that corner case and unconditionally remove our quotes.

Even if you didn't prepend your own `cmd ... /c`, you still have to deal with this if you specified both `lpApplicationName` and `lpCommandLine`, and `lpApplicationName` is a batch file. Then you have to quote the entire `lpCommandLine`. We have nowhere to pass our `/s` in that case, but it's not an issue, since the quotes are always removed when there are more than two of them, and we'll have 4 because we also always quote the first element in `lpCommandLine` for [unrelated reasons](#quoting-the-executable-name).

## The executable name

Lastly, the executable name passed to `CreateProcess()` has a few quirks that need to be handled.

### Quoting the executable name

First of all, when passed in the second argument (`lpCommandLine`) of `CreateProcess()`, it **must** be quoted regardless of the contents. If not quoted, then `C:\foo bar.exe` is ambiguous between running `C:\foo bar.exe` and running `C:\foo.exe` with argument `bar.exe`. It'll check different paths and run the one that exists (the [docs](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessw) say it tries shorter ones first). Quotes disable this behavior and make it not ambiguous.

### Trailing garbage in executable name

Secondly, you should error if the executable name contains any trailing <code> </code> spaces or `.` dots. Or alternatively remove them yourself. (Do this for `lpApplicationName` if specified, and otherwise for the first part of `lpCommandLine`.)

Trailing <code> </code>,`.` are normally ignored by `CreateProcess()` when looking for the executable, but are propagated to `argv[0]` as is. **BUT** when dealing with batch files, they are very broken. For example:

* Passing `foo.bat.` as the program name errors, but in a weird way. It DOES start `cmd.exe` to run the batch file, but gives it the path `foo.bat.` literally, and it then complains about not being able to find it.

  Note that if you run `foo.bat. &calc.exe`, it manages to run `calc.exe` before erroring.

* Passing `foo.bat .` as the program name errors in `lpCommandLine` (so when quoted). But when done in `lpApplicationName`, it instead passes `.` as the argument (but normally `lpApplicationName` can't be used to pass arguments!). You can even get multiple arguments in: `foo.bat .. . ...`.

Since trailing <code> </code>,`.` are so broken, it's easier to error on them or trim them yourself. They aren't useful anyway.

### How to check if it's a batch file?

After stripping trailing <code> </code>,`.`, check if the name ends with `.bat` or `.cmd` (both case-insensitive).

I also recommend enabling batch logic if the executable name is `cmd` or `cmd.exe` (both case-insensitive). CMD has the same escaping quirks since it's what runs batch files. This is best-effort, since the user can refer to this shell in a bunch of different ways (`C:\Windows\System32\cmd.exe`, etc).

Covering any possible spelling of `cmd` seems unnecessary, since the paranoid escaping logic is important only when the executable name is hardcoded but the arguments are user-provided. If the executable name is user-provided, they can already run anything they want.

## The escaping algorithm

If you're using `CreateProcessW()` ([as you probably should](#unicode)) which accepts wide strings, you can either run this algorithm directly on UTF-16 strings, or on UTF-8 strings and then widen the result.

The inputs are: an optional string `executable`, and an optional array of strings `argv` (that correspond to the [two parameters of `CreateProcess()`](#basic-use-of-createprocess)). Normally you'd only specify `argv`.

For simplicity you can get rid of the `executable` parameter and only allow `argv`. Then you lose the ability to have `argv[0]` differ from the executable path.

1. Check for bad inputs:
   * They can't contain `\0`.
   * At least one parameter of the two must be specified.
   * If `argv` is specified, it can't be empty.

2. Let `exe_name` be `executable` if specified, or `argv[0]` otherwise.<br/>

3. Error if `exe_name` ends with <code> </code> space or `.` dot. [(details)](#trailing-garbage-in-executable-name)<br/>
   Or alternatively remove any trailing spaces and dots from it yourself (could be more than one). If you modify it, propagate the same change to `executable` or `argv[0]`, depending on where you took it from.

4. Check if this is a direct CMD invocation: check if `exe_name` equals `cmd` or `cmd.exe`, case-insensitive. [(details)](#how-to-check-if-its-a-batch-file)<br/>
    This is best-effort, we don't need to catch all possible spellings of CMD, see link.

5. Check if this is a batch file: check if `exe_name` ends with `.bat` or `.cmd`, case-insensitive. [(details)](#how-to-check-if-its-a-batch-file)

6. If this is batch-or-cmd (per steps 4, 5), perform additional argument validation. Error if any element of `argv` (including `0`th) contains any of: `%`, `\n` (line break), `\r` (carriage return). [(details)](#what-characters-need-to-be-banned-in-batch-arguments)

    If `argv` is null, then instead validate `executable` with this.

    You can allow `%`, but it's potentially unsafe. If you allow it, then you should escape it as explained in the next steps. [(details)](#escaping-)

7. If this is batch, prepend the CMD invocation:

    * If `argv` is null, make it empty instead.

    * Prepend those elements to `argv`: `cmd`,`/d`,`/e:on`,`/v:off`,`/s`,`/c`.

    * If `argv` was originally null, add the value of `executable` at the end of it.

    * Reset `executable` to null.

    * Update the bools from steps 4 and 5: now "is cmd" = true, "is batch" = false.

    The entire step 7 can be skipped, but if you skip it, then you should also reject `!` (and `%`) earlier on step 6. [(details)](#escaping--1)

8. Decide if the entire command needs to be quoted: check if `executable` isn't null and this is batch per step 5 (if you executed step 7, it's no longer considered batch). [(details)](#special-quoting-rules-of-cmd-c)

9. To assemble the command string:

    1. If the whole command needs to be quoted per step 8, write opening quote `"`. [(details)](#special-quoting-rules-of-cmd-c)

    2. For each element in `argv`:

        * Write separating space <code> </code> if it's not the `0`th element (and if the separator doesn't need to be skipped because of the preceding `/c`, see below).

        * Decide if this element needs to be quoted:

            * If it's the `0`th element, always quote. [(details)](#quoting-the-executable-name)

            * Quote if the element contains any of: <code> </code> spaces, `\t` tabs, `"` quotes. If this is batch-or-cmd, also check for ``<>&|()[]{}^=;!'+,`~``. [(details)](#what-characters-need-to-be-quoted-in-batch-arguments)

        * Write opening quote `"` if we're quoting this element.

        * Write the modified element string:
            * Replace any `"` with `""`.
            * Replace any sequence of 1+ `\` backslashes with twice as many backslashes **only if**:

                * It's right before `"` in this element, or
                * It's right at the end of the element, and we've decided to quote this element.

                Otherwise leave those `\` unchanged.

                (Note that if this is the last element, and it's not quoted, and the entire command is quoted because of step 8 or `/c`, then `\`s at the end of this element still do **not** need to be duplicated. They only need to be duplicated if this element is quoted individually.)

            * If you decided to allow `%` on step 6, replace those with `%%cd:~,%`.

        * Write closing quote `"` if we're quoting this element.

        * Do the custom handling for for `/c`. [(details)](#special-quoting-rules-of-cmd-c)<br/>
            If all of the following are true: (this happen to be mutually exclusive with step 8)

            * We didn't have a `/c` yet.
            * This element equals `/c` (case insensitive).
            * This is a direct CMD invocation per step 4 (or step 7 was executed).

            Then immediately write <code> "</code> (space and a quote), and then skip writing <code> </code> separator on the next iteration.

    3. Write closing `"` If the whole command needs to be quoted per step 8, or if you handled `/c` as explained earlier. [(details)](#special-quoting-rules-of-cmd-c)

As you can see, this has some knobs for batch files. I'd suggest exposing the following modes as a setting:

Mode|Prepend `cmd /d /e:on /v:off /s /c `|Allow&nbsp;`%`|Allow&nbsp;`!`|Comment
---|---|---|---|---
Default|Yes|No|Yes|Seems to be a good default.
Relaxed|Yes|Yes<br/>(escaped)|Yes|Can be unsafe on some batch files.
Keep registry settings|No|No|No|Safe, but the user messing with registry keys can affect your batch files.
Unsafe|No|Yes (not escaped)|Yes|Unsafe.

This could be exposed as an enum, or perhaps two bools (`keep_registry_settings` and `unsafe`).

&nbsp;

Thanks for reading!
