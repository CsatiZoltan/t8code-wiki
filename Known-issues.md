# Known issues

On this page we collect known issues/bugs related to t8code

## Pure g++ installation not possible with gcc >=v8 

Using `CC=g++` or `CC=mpicxx` or similar will not configure when the gcc Version is 8 or larger.
This particularly concerns Ubuntu 22.04 and beyond.

The behavior is caused by a bug in autotools regarding the AC_CHECK_LIBRARY macro that causes an error message in the C++ compiler (which previously was only a warning).
See the discussion in a close PR https://github.com/holke/t8code/pull/257 and the autotools discussion https://www.mail-archive.com/bug-autoconf@gnu.org/msg04294.html