# Makefile
# A 'better' Makefile template for Linux system programming.
#
# Meant for the case where a single C source file is built with various useful targets.
# If you require to compile/link multiple C source files into one executable, refer the
# <src>/makefile_templ/hdr_var/Makefile.
#
# Besides the 'usual' targets to build production and debug versions of the
# code and cleanup, we incorporate targets to do useful (and indeed required)
# stuff like:
#  - prod_2part: build a '2-part' production target 'prod_2part'; it's
#     -O${PROD_OPTLEVEL}, no debug symbolic info, strip-debug;
#     Excellent for production as it gives ability to debug as and when required!
#  - indent: adhering to (Linux kernel) coding style guidelines (indent+checkpatch)
#  - sa: static analysis target (via flawfinder, cppcheck)
#  - dynamic analysis target: via valgrind
#  - code coverage via gcov/lcov
#  - a packaging (.tar.xz) target and
#  - a help target.
#
# You will require these utils installed:
#  indent, flawfinder, cppcheck, valgrind, kernel-headers package -or- simply the
#  checkpatch.pl script, gcov, lcov, tar; + libasan
#  The lcov_gen.sh wrapper script
#   (DEV NOTE: careful of dependencies between this Makefile and script)
#
# To get started, just type:
#  make help
#
# (c) 2020 Kaiwan N Billimoria, kaiwanTECH
# License: MIT

## Pl check and keep or remove <foo>_dbg_[asan|ub|msan] targets
## (where <foo> is the program name) as desired.
ALL :=  prod prod_2part

###
# Update as required
# Simply replace the variable ${FNAME_C} below (currently set to 'killer'),
# with your program name!
# We also provide the ../../mk script to trivially replace the value of FNAME_C.
# Of course, if you have >1 C program to build, you must add it manually.
# Also, it's recommended to keep one Makefile per program in separate directories.
###
FNAME_C := ioctl_llkd_userspace
#--- CHECK: manually add params as required
# Populate any required cmdline arguments to the process here:
CMDLINE_ARGS=


# Can remove the 'msan' one if it doesn't build - requires clang
ALL_NM :=  ${FNAME_C} ${FNAME_C}_dbg ${FNAME_C}_dbg_asan ${FNAME_C}_dbg_ub ${FNAME_C}_dbg_lsan ${FNAME_C}_gcov ${FNAME_C}_dbg_msan ${FNAME_C}_dbg_tsan

# Decide which compiler to use; GCC doesn't support MSAN, clang does
CC := gcc
ifeq (, $(shell which clang))
  $(warning === WARNING! No clang (compiler) in PATH (reqd for MSAN); consider doing 'sudo apt install clang' or equivalent ===)
else
  CC := clang
  LINKIN := -static-libsan
endif
#CC := gcc
#LINKIN := -static-libasan
$(info Compiler set to $(CC))

STRIP=${CROSS_COMPILE}strip
OBJCOPY=${CROSS_COMPILE}objcopy

CSTD=-ansi -std=c99 -std=c11 -std=c18  # the last one wins; else if unsupported, earlier ones...
# For the meaning of the following feature test macros, see feature_test_macros(7)
POSIX_STD=201112L
STD_DEFS=-D_DEFAULT_SOURCE -D_GNU_SOURCE

# Add this option switch to CFLAGS / CFLAGS_DBG if you want ltrace to work on Ubuntu!
LTRACE_ENABLE=-z lazy

PROD_OPTLEVEL=2
  # -O<N>; N can be 0-3 or s (optimize for speed & size);
  #  in the -D_FORTIFY_SOURCE=N option, you must set N to the same number

# TODO : verify..
SECURITY_CFLAGS=-fstack-protector-strong -Wformat-security \
	-O${PROD_OPTLEVEL} -D_FORTIFY_SOURCE=${PROD_OPTLEVEL} \
	-fPIE \
	-fsanitize=bounds -fsanitize-undefined-trap-on-error

#ifeq (gcc, $(CC))
#  SECURITY_CFLAGS=${SECURITY_CFLAGS} -fstrict-flex-arrays
#endif

CFLAGS=-Wall -UDEBUG ${CSTD} -D_POSIX_C_SOURCE=${POSIX_STD} ${STD_DEFS} ${SECURITY_CFLAGS}
# Dynamic analysis includes the compiler itself!
# Especially the powerful Address Sanitizer (ASAN) toolset
#CFLAGS_DBG=-g -ggdb -gdwarf-4 -Og -Wall -Wextra -DDEBUG -fno-omit-frame-pointer -Werror=format-security ${CSTD} -D_POSIX_C_SOURCE=${POSIX_STD} ${STD_DEFS}
CFLAGS_DBG=-g -ggdb -gdwarf-4 -Og -Wall -Wextra -Wpedantic -DDEBUG -fno-omit-frame-pointer ${CSTD} -D_POSIX_C_SOURCE=${POSIX_STD} ${STD_DEFS} ${SECURITY_CFLAGS}
# Why -Og and not -O0?
# man gcc
# '... Optimize debugging experience.  -Og should be the optimization level of
# choice for the standard edit-compile-debug cycle, offering a reasonable level
# of  optimization  while maintaining  fast  compilation  and a good debugging
# experience. It is a better choice than -O0 for producing debuggable code
# because some compiler passes that collect debug information are disabled at -O0.'

# For MSan, don't use the -D_FORTIFY_SOURCE option
CFLAGS_DBG_ASAN=${CFLAGS_DBG} -fsanitize=address -fsanitize-address-use-after-scope
CFLAGS_DBG_UB=${CFLAGS_DBG} -fsanitize=undefined
CFLAGS_DBG_MSAN=${CFLAGS_DBG} -fsanitize=memory -fPIE -pie
CFLAGS_DBG_LSAN=${CFLAGS_DBG} -fsanitize=leak
CFLAGS_DBG_TSAN=${CFLAGS_DBG} -fsanitize=thread

CFLAGS_GCOV=${CFLAGS_DBG} -fprofile-arcs -ftest-coverage -lgcov
LINK=  #-pthread

# Required vars
all: ${ALL}
SRC_FILES := *.[ch]
INDENT := indent
FLAWFINDER := flawfinder
CPPCHECK := cppcheck
VALGRIND := valgrind
# update as required
PKG_NAME := ${FNAME_C}
CHECKPATCH := /lib/modules/$(shell uname -r)/build/scripts/checkpatch.pl
GCOV := gcov
LCOV := lcov
GENINFO := geninfo
GENHTML := genhtml
LCOV_SCRIPT := lcov_gen.sh

# Targets and their rules
# Three types:
# 1. 'regular' production target 'prod': -O${PROD_OPTLEVEL}, no debug symbolic info, stripped
# 2. '2-part' production target 'prod_2part': -O${PROD_OPTLEVEL}, no debug symbolic info, strip-debug;
#     excellent for production as it gives ability to debug as and when required!
#     (internally invokes the 'debug' target as it requires the debug binary as well
# 3. 'debug' target(s): -Og, debug symbolic info (-g -ggdb), not stripped
prod: ${FNAME_C}.c
	@echo
	@echo "--- building 'production'-ready target (-O${PROD_OPTLEVEL}, no debug, stripped) ---"
	@echo " glibc (and NPTL) version: $(shell getconf GNU_LIBPTHREAD_VERSION|cut -d' ' -f2)"
	@echo
	${CC} ${CFLAGS} ${FNAME_C}.c -o ${FNAME_C} ${LINK}
	${STRIP} --strip-debug ./${FNAME_C}

# The '2-part executable' solution : use strip and objcopy to generate a
# binary executable that has the ability to retrieve debug symbolic information
# from the 'debug' binary!
prod_2part: ${FNAME_C}.c
	@echo
	@echo "--- building 'production'-ready 2-part target (-O${PROD_OPTLEVEL}, no debug, strip-debug) ---"
	@echo " glibc (and NPTL) version: $(shell getconf GNU_LIBPTHREAD_VERSION|cut -d' ' -f2)"
	@echo
# We require the 'debug' build for the 2part, so do that first
	make --ignore-errors debug
	${CC} ${CFLAGS} ${FNAME_C}.c -o ${FNAME_C} ${LINK}
	${STRIP} --strip-debug ./${FNAME_C}
# Most IMP, setup the 'debug link' in the release binary, pointing to the debug
# info file; this, in effect, is the 2part solution!
	${OBJCOPY} --add-gnu-debuglink=./${FNAME_C}_dbg ./${FNAME_C}

# 'debug' target: -Og, debug symbolic info (-g -ggdb), not stripped.
# Generates the regular debug build, debug+ASAN, debug+UB, debug+LSAN, debug+MSAN builds'
# MSAN requires clang.
# When using clang on Debian/Fedora-type distros, use -static-libsan (LINKIN is
# set to this value); with GCC, and on Fedora-type distros, other libraries
# (libasan, libubsan, liblsan) are required (pkg: lib<name>-<gcc ver>-...)
debug: ${FNAME_C}.c
	@echo
	@echo "--- building 'debug'-ready targets (with debug symbolic info, not stripped) ---"
	@echo " glibc (and NPTL) version: $(shell getconf GNU_LIBPTHREAD_VERSION|cut -d' ' -f2)"
	@echo
	${CC} ${CFLAGS_DBG} ${FNAME_C}.c -o ${FNAME_C}_dbg ${LINK}
#-- Sanitizers (use clang or GCC)
	${CC} ${CFLAGS_DBG_ASAN} ${FNAME_C}.c -o ${FNAME_C}_dbg_asan ${LINK} ${LINKIN}
	${CC} ${CFLAGS_DBG_UB} ${FNAME_C}.c -o ${FNAME_C}_dbg_ub ${LINK} ${LINKIN}
# GCC doesn't support MSAN, clang does
ifeq (gcc, $(CC))
	$(warning === WARNING! No clang (compiler) in PATH (reqd for MSAN); consider doing 'sudo apt install clang' or equivalent ===)
else
	${CC} ${CFLAGS_DBG_MSAN} ${FNAME_C}.c -o ${FNAME_C}_dbg_msan ${LINK}
endif
	${CC} ${CFLAGS_DBG_LSAN} ${FNAME_C}.c -o ${FNAME_C}_dbg_lsan ${LINK} ${LINKIN}
# ThreadSanitizer (TSan):
# For clang ver < 18.1.0 (Mar '24), need to set vm.mmap_rnd_bits sysctl to 28 (default is 32)
# else it bombs on execution (ref: https://stackoverflow.com/a/77856955/779269)
	sudo sysctl vm.mmap_rnd_bits=28
	${CC} ${CFLAGS_DBG_TSAN} ${FNAME_C}.c -o ${FNAME_C}_dbg_tsan ${LINK} ${LINKIN}


#--------------- More (useful) targets! -------------------------------

# indent- "beautifies" C code - to conform to the the Linux kernel
# coding style guidelines.
# Note! original source file(s) is overwritten, so we back it up.
# code-style : "wrapper" target over the following kernel code style targets
code-style:
	make --ignore-errors indent
	make --ignore-errors checkpatch

indent: ${SRC_FILES}
ifeq (, $(shell which ${INDENT}))
	$(warning === WARNING! ${INDENT} not installed; consider doing 'sudo apt install indent' or equivalent ===)
else
	make clean
	@echo
	@echo "--- applying Linux kernel code-style indentation with indent ---"
	@echo
	mkdir bkp 2>/dev/null; cp -f ${SRC_FILES} bkp/
	${INDENT} -linux ${SRC_FILES}
endif
# RELOOK
# !WARNING!
# I came across this apparent bug in indent when using it on Ubuntu 20.04:
#  realloc(): invalid next size
#  Aborted (core dumped)
# Worse, it TRUNCATED the source file to 0 bytes !!! So backing them up - as we
# indeed do - is good.

checkpatch:
	make clean
	@echo
	@echo "--- applying Linux kernel code-style checking with checkpatch.pl ---"
	@echo
	${CHECKPATCH} -f --no-tree --max-line-length=95 ${SRC_FILES}

# sa : "wrapper" target over the following static analyzer targets
sa:   # static analysis
	make --ignore-errors sa_flawfinder
	make --ignore-errors sa_cppcheck

# static analysis with flawfinder
sa_flawfinder:
ifeq (, $(shell which ${FLAWFINDER}))
	$(warning === WARNING! ${FLAWFINDER} not installed; consider doing 'sudo apt install flawfinder' or equivalent ===)
else
	make clean
	@echo
	@echo "--- static analysis with flawfinder ---"
	@echo
	${FLAWFINDER} --neverignore --context *.[ch]
endif

# static analysis with cppcheck
sa_cppcheck:
ifeq (, $(shell which ${CPPCHECK}))
	$(warning === WARNING! ${CPPCHECK} not installed; consider doing 'sudo apt install cppcheck' or equivalent ===)
else
	make clean
	@echo
	@echo "--- static analysis with cppcheck ---"
	@echo
	${CPPCHECK} -v --force --enable=all -i bkp/ --suppress=missingIncludeSystem .
endif

# Dynamic Analysis
# dynamic analysis with valgrind
valgrind:
ifeq (, $(shell which ${VALGRIND}))
	$(warning === WARNING! ${VALGRIND} not installed; consider doing 'sudo apt install valgrind' or equivalent ===)
else
	make --ignore-errors debug
	@echo
	@echo "--- dynamic analysis with Valgrind memcheck ---"
	@echo
#--- CHECK: have you populated the above CMDLINE_ARGS var with the required cmdline?
	@if test -z "${CMDLINE_ARGS}"; then echo "\n@@@ (Possible) Warning: no parameters being passed to the program under test ${FNAME_C}_gcov ? @@@\n"; fi
	${VALGRIND} --tool=memcheck --trace-children=yes ./${FNAME_C}_dbg ${CMDLINE_ARGS}
endif

# dynamic analysis with the Sanitizer tooling
san:
	make --ignore-errors debug
	@echo
	@echo "--- dynamic analysis with the Address Sanitizer (ASAN) ---"
	@echo
	@if test -z "${CMDLINE_ARGS}"; then echo "\n@@@ (Possible) Warning: no parameters being passed to the program under test ${FNAME_C}_gcov ? @@@\n"; fi
	./${FNAME_C}_dbg_asan ${CMDLINE_ARGS}

	@echo
	@echo "--- dynamic analysis with the Undefined Behavior Sanitizer (UBSAN) ---"
	@echo
	./${FNAME_C}_dbg_ub ${CMDLINE_ARGS}

	@echo
	@echo "--- dynamic analysis with the Memory Sanitizer (MSAN) ---"
	@echo
	./${FNAME_C}_dbg_msan ${CMDLINE_ARGS}

	@echo
	@echo "--- dynamic analysis with the Thread Sanitizer (TSAN) ---"
	@echo
	./${FNAME_C}_dbg_tsan ${CMDLINE_ARGS}

# dynamic analysis run with LSan binary not done here (as ASan typically covers leakage)

#----- Testing: line coverage with gcov(1), lcov(1)
# ref: https://backstreetcoder.com/code-coverage-using-gcov-lcov-in-linux/
covg:
	@echo
	@echo "=== Code coverage (funcs/lines/branches) with gcov+lcov ==="
	@echo

ifeq (,$(wildcard /etc/lcovrc))
	$(error ERROR: install lcov first)
endif
# Set up the ~/.lcovrc to include branch coverage
# ref: https://stackoverflow.com/questions/12360167/generating-branch-coverage-data-for-lcov
ifneq (,$(wildcard ~/.lcovrc))
	@echo "~/.lcovrc in place"
else
	cp /etc/lcovrc ~/.lcovrc
	sed -i 's/^#genhtml_branch_coverage = 1/genhtml_branch_coverage = 1/' ~/.lcovrc
	sed -i 's/^lcov_branch_coverage = 0/lcov_branch_coverage = 1/' ~/.lcovrc
endif
ifeq (, $(shell which ${LCOV_SCRIPT}))
	$(error ERROR: ensure our ${LCOV_SCRIPT} wrapper script's installed and in your PATH first; location: https://github.com/kaiwan/usefulsnips/blob/master/lcov_gen.sh)
endif

#--- Build for coverage testing; this generates the binary executable named
# ${FNAME_C}_covg and the .gcno ('notes') files as well
	make clean
	@echo "___"
	@echo "> Forcing compiler to GCC for coverage, as gcov/lcov seem to require it"
	$(eval CC := gcc)
# For coverage analysis, gcov/lcov seems to require compilation via GCC (not clang)
	${CC} ${CFLAGS_GCOV} ${FNAME_C}.c -o ${FNAME_C}_gcov ${LINK}
	@if test -z "${CMDLINE_ARGS}"; then echo "\n@@@ (Possible) Warning: no parameters being passed to the program under test ${FNAME_C}_gcov ? @@@\n"; fi

	@echo "-------------- Running via our wrapper ${LCOV_SCRIPT} --------------"
	${LCOV_SCRIPT} ${FNAME_C}_gcov ${CMDLINE_ARGS}
# lcov_gen.sh Notes:
#  - If you want a cumulative / merged code coverage report, run your next coverage
#    test case via this script. In effect, simply adjust the CMDLINE_ARGS variable here
#    and run 'make covg' again
#  - If you want to start from scratch, *wiping* previous coverage data, then
#    add the -r (reset) option when invoking this script (above) -OR-
#    simply invoke the 'clean_lcov' target (which deletes all the lcov meta dirs)
	
# exit unconditionally
%:
	@true

# Testing all
# Limitation:
# When the PUT (Prg Under Test) runs in an infinite loop or forever (eg. servers/daemons),
# you may have to manually run a client process (or whatever) and exit the main process
# programatically; else, a signal like ^C does abort it BUT make doesn't continue (even
# when run with --ignore-errors).
test:
	@echo
	@echo "=== Test All ==="
	@echo "-------------------------------------------------------------------------------"
	make --ignore-errors code-style
	@echo "-------------------------------------------------------------------------------"
	make --ignore-errors sa
	@echo "-------------------------------------------------------------------------------"
	make --ignore-errors valgrind
	@echo "-------------------------------------------------------------------------------"
	make --ignore-errors san
	@echo "-------------------------------------------------------------------------------"
	make --ignore-errors covg

# packaging
package:
	@echo
	@echo "--- packaging ---"
	@echo
	rm -f ../${PKG_NAME}.tar.xz
	make clean
	tar caf ../${PKG_NAME}.tar.xz *
	ls -l ../${PKG_NAME}.tar.xz
	@echo "=== $(PKG_NAME).tar.xz package created ==="
	@echo 'Tip: when extracting, to extract into a dir of the same name as the tar file,'
	@echo ' do: tar -xvf ${PKG_NAME}.tar.xz --one-top-level'

clean:
	@echo
	@echo "--- cleaning ---"
	@echo
	rm -vf ${ALL_NM} core* vgcore* *.o *~
# rm some of the code coverage metadata
	rm -rfv ${FNAME_C}_gcov *.[ch].gcov *.gcda *.gcno *.info

	@echo "Note: Special case wrt the 'clean' target and the code coverage target (covg):"
	@echo " It deliberately does NOT delete the LCOV metadata, intermediate and final LCOV coverage"
	@echo " report folders - the ones named 0lcov_meta/, lcov_onerun_html/ and lcov_merged_html/ resp,"
	@echo " as they're required to generate a merged / cumulative code coverage report."
	@echo "To start code coverage analysis from scratch, you can either:"
	@echo "- invoke the special 'make clean_lcov' (it manually delete these 3 folders), OR,"
	@echo "- change the invocation of the lcov_gen.sh script in the Makefile, passing the -r option"

clean_lcov:
	@echo
	@echo "--- cleaning LCOV metadata ---"
	@echo
	# NOTE! depedency on the lcov_gen.sh script, on these folder names
	rm -rvf 0lcov_meta/  lcov_merged_html/  lcov_onerun_html/

help:
	@echo '=== Makefile Help : additional targets available ==='
	@echo
	@echo 'This Makefile is appropriate for single-sourcefile program builds'
	@echo 'If your purpose is to build an app with multiple source files (across'
	@echo 'multiple dirs), then pl use the <src>/makefile_templ/hdr_var/Makefile template).'
	@echo
	@echo 'TIP: type make <tab><tab> to show all valid targets'
	@echo

	@echo 'Regular targets ::'
	@echo ' 1. 'prod'  : regular production target: -O${PROD_OPTLEVEL}, no debug symbolic info, stripped'
	@echo ' 2. 'debug' : -Og, debug symbolic info (-g -ggdb), not stripped. Generates the regular debug build, debug+ASAN, debug+UB, debug+LSAN, debug+MSAN builds'
	@echo ' 3. 'prod_2part': production target : -O${PROD_OPTLEVEL}, no debug symbolic info, strip-debug; \
    Excellent for production as it gives ability to debug as and when required! \
    (shown as third option as it *requires* the 'debug' build as a step'
	@echo
	@echo 'Doing a 'make' will build all three shown above.'

	@echo
	@echo '--- code style targets ---'
	@echo 'code-style : "wrapper" target over the following kernel code style targets'
	@echo ' indent     : run the $(INDENT) utility on source file(s) to indent them as per the kernel code style'
	@echo ' checkpatch : run the kernel code style checker tool on source file(s)'

	@echo
	@echo '--- static analyzer targets ---'
	@echo 'sa          : "wrapper" target over the following static analyzer targets'
	@echo ' sa_flawfinder : run the static analysis flawfinder tool on the source file(s)'
	@echo ' sa_cppcheck   : run the static analysis cppcheck tool on the source file(s)'

	@echo
	@echo '--- dynamic analysis targets ---'
	@echo ' valgrind   : run the dynamic analysis tool ($(VALGRIND)) on the binary executable'
	@echo ' san        : run dynamic analysis via ASAN, UBSAN, MSAN and TSAN tooling on the binary executable'

	@echo
	@echo '--- code coverage ---'
	@echo ' covg       : run the gcov+lcov code coverage tooling on the source (generates html output!). NOTE: this target requires our ${LCOV_SCRIPT} wrapper script installed (location: https://github.com/kaiwan/usefulsnips/blob/master/lcov_gen.sh)'
	@echo " Note: Special case wrt the 'clean' target and the code coverage target (covg):"
	@echo "  It deliberately does NOT delete the LCOV metadata, intermediate and final LCOV coverage"
	@echo "  report folders - the ones named 0lcov_meta/, lcov_onerun_html/ and lcov_merged_html/ resp,"
	@echo "  as they're required to generate a merged / cumulative code coverage report."
	@echo " To start code coverage analysis from scratch, you can either:"
	@echo " - invoke the special 'make clean_lcov' (it manually delete these 3 folders), OR,"
	@echo " - change the invocation of the lcov_gen.sh script in the Makefile, passing the -r option"

	@echo
	@echo '--- TEST all ---'
	@echo ' test       : run all targets (it runs them in this order): code-style, sa, valgrind, san, covg'
	@echo '              Tip: run "make -i test > out 2>&1" to save all output to a file "out".'
	@echo '                             -i = --ignore-errors'

	@echo
	@echo '--- misc targets ---'
	@echo ' clean      : cleanup - remove all the binaries, core files, etc'
	@echo '              See special note wrt code coverage'
	@echo ' clean_lcov : cleanup the LCOV metadata folders; implies code coverage starts from scratch'
	@echo ' package    : tar and compress the source files into the dir above'
	@echo '  Tip: when extracting, to extract into a dir of the same name as the tar file, do:'
	@echo '       tar -xvf ${PKG_NAME}.tar.xz --one-top-level'

	@echo ' help       : this 'help' target'
