# LIRC tips for contributors

So, you are considering contributing to lirc? You are most welcome! Some points:

## Code layout

We use the kernel coding standards, as described in [1]. Many lines in the code
are actually too long according to these, but stick to the rules for new code.

In the git-tools directory there is a pre-commit hook aimed to be installed in
.git/hooks/pre-commit (a symlink works fine). This handles most of the boring
tabs and trailing whitespace problems on checkin. Here is also a
fix-whitespace script which can be used to filter a range of commits.

## Git branches

We basically use the branching scheme described in [2]. However, what is called
'devel' in that document we call 'master'. Likewise, what is called 'master' in [2]
we call 'release'. In short:

    - master is the current development, from time to time unstable.
    - release contains the last stable version, and also all tagged releases.
    - When a release is upcoming, we fork a release branch from master. This
      is kept stable, only bugfixes are allowed. Eventually it is merged into
      release and tagged.
    - Other branches are feature branches for test and review. They can not
      be trusted, and are often rewritten.

## New remote configuration files

There is a document contrib/remote-checklist.txt describing how to submit
new remotes.

## Testing and and bug reporting

Non-trivial changes should be checked using the lirc-codecs-regression-test.sh
in contrib/. Structural and  build system changes should be tested with
'make distcheck'. New testing tools are more than welcome...

Please report bugs, RFE.s etc at sourceforge [3]; either the mailing list or the
issue tracker.

## Running from the source tree.

You can run the lirc programs directly from their source directory after a
successful 'make' without installing things. The gotcha is that you need
to provide the plugin directory on the command line. E. g., to run lircd:

    $ make
    $ cd daemons
    $ ./lircd --nodaemon --plugindir=../plugins/.libs

Note that the gnu tools places the generated so-files in the hidden .libs
directory.

## Generating a stacktrace.

A stacktrace is extremely useful if a lirc program crashes. The common way
is to use gdb. If possible, try to start the program you are testing from
the command line rather than from e. g.,  a systemd script.

To generate a core and show the stacktrace:

    $ cd daemons
    $ ulimit -c unlimited
    $ ./lircd --nodaemon  --plugindir=../plugins/.libs [other options]
    ---> crash
    $ ls -lt core*   # will list newest core first.
    $ libtool --mode=execute gdb lircd core.12345
    [... lots of gdb welcome talk]
    (gdb) bt
    #12 0xb7fbba63 in for_each_driver [...] at hw-types.c:167
    #13 0xb7fbbad3 in hw_choose_driver [...] at hw-types.c:206
    #14 0x08049d39 in main (argc=2, argv=0xbffff034) at lircd.c:2250

You dont need a core file. You can also start your program inside gdb and
let gdb trap the crash. Something like

    $ cd daemons
    $ libtool --mode=execute gdb lircd
    (gdb) set args --nodaemon --plugindir=../plugins/.libs [more options]
    (gdb) run
    ----> program crashed
    (gdb) bt
    [ stack trace displayed ]


## References

[1] https://www.kernel.org/doc/Documentation/CodingStyle
[2] http://nvie.com/posts/a-successful-git-branching-model
[3] http://sourceforge.net/projects/lirc/