CC = gcc
CYTHON = cython
CYTHON_FREEZE = ../../bin/cython_freeze
PYTHON = python
RST2HTML = rst2html

PY_LDFLAGS = $(shell $(PYTHON) -c 'from distutils.sysconfig import get_config_var as g; import sys; sys.stdout.write(" ".join([g("LINKFORSHARED"), "-L"+g("LIBPL")]) + "\n")')
PY_CPPFLAGS = $(shell $(PYTHON) -c 'from distutils.sysconfig import *; import sys; sys.stdout.write("-I"+get_python_inc() + "\n")')
PY_LDLIBS = $(shell $(PYTHON) -c 'from distutils.sysconfig import get_config_var as g; import sys; sys.stdout.write(" ".join(["-lpython"+g("VERSION"), g("SYSLIBS"), g("LIBS"), g("LOCALMODLIBS")]) + "\n")')

CFLAGS = -fPIC -fno-strict-aliasing -g -O2 -Wall -Wextra
CPPFLAGS = $(PY_CPPFLAGS)
LDFLAGS = $(PY_LDFLAGS)
LDLIBS = $(PY_LDLIBS)

default : all test

test : test_main_module test_interpreter test_imported
all : main_module interpreter imported

imported : imported.o importer.o testlib/some_module.o
imported.c :
	$(CYTHON_FREEZE) importer testlib/some_module.pyx -o $@
test_imported : imported
	./imported

main_module : main_module.o testlib/some_module.o
main_module.c :
	$(CYTHON_FREEZE) testlib/some_module.pyx -o $@
test_main_module : main_module
	./main_module

interpreter : interpreter.o testlib/some_module.o
interpreter.c :
	$(CYTHON_FREEZE) --pymain testlib/some_module.pyx -o $@
test_interpreter : interpreter
	./interpreter -c 'import testlib.test'

# Hack for now - eventually we should patch Cython to do this.
testlib/some_module.c : testlib/some_module.pyx
	$(CYTHON) $(CYTHONFLAGS) $^
	sed -i -e 's/"some_module"/"testlib.some_module"/g' $@

%.c : %.pyx
	$(CYTHON) $(CYTHONFLAGS) $^

clean:
	$(RM) imported.c importer.c main_module.c interpreter.c testlib/some_module.c
	$(RM) imported.o importer.o main_module.o interpreter.o testlib/some_module.o
	$(RM) imported main_module interpreter
	$(RM) *.pyc testlib/*.pyc

.PHONY: clean all test_main_module test_interpreter test_imported
.SECONDARY:
