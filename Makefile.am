## Process this file with automake to produce Makefile.in

if BUILD_DSELECT
  MAYBE_DSELECT = dselect
endif

SUBDIRS = \
	lib \
	dpkg-deb \
	dpkg-split \
	src \
	utils \
	$(MAYBE_DSELECT) \
	scripts \
	t-func \
	po \
	man

ACLOCAL_AMFLAGS = -I m4


dist_pkgdata_DATA = \
	data/pie-compile.specs \
	data/pie-link.specs \
	data/no-pie-compile.specs \
	data/no-pie-link.specs \
	data/cputable \
	data/ostable \
	data/abitable \
	data/tupletable \
	$(nil)

EXTRA_DIST = \
	.mailmap \
	ChangeLog.old \
	README.l10n \
	get-version \
	run-script \
	doc/README.api \
	doc/README.feature-removal-schedule \
	doc/coding-style.txt \
	doc/frontend.txt \
	doc/lcov-inject.pl \
	doc/rootless-builds.txt \
	doc/triggers.txt \
	debian/changelog \
	debian/compat \
	debian/control \
	debian/copyright \
	debian/dpkg-dev.docs \
	debian/dpkg-dev.install \
	debian/dpkg-dev.manpages \
	debian/dpkg-dev.lintian-overrides \
	debian/dpkg.alternatives.logrotate \
	debian/dpkg.cfg \
	debian/dpkg.cron.daily \
	debian/dpkg.docs \
	debian/dpkg.install \
	debian/dpkg.manpages \
	debian/dpkg.postinst \
	debian/dpkg.postrm \
	debian/dpkg.logrotate \
	debian/dpkg.lintian-overrides \
	debian/dselect.cfg \
	debian/dselect.docs \
	debian/dselect.install \
	debian/dselect.manpages \
	debian/dselect.lintian-overrides \
	debian/libdpkg-dev.docs \
	debian/libdpkg-dev.install \
	debian/libdpkg-dev.lintian-overrides \
	debian/libdpkg-perl.docs \
	debian/libdpkg-perl.install \
	debian/libdpkg-perl.lintian-overrides \
	debian/source/lintian-overrides \
	debian/source/format \
	debian/usertags \
	debian/rules \
	debian/shlibs.default \
	debian/shlibs.override \
	po/its/polkit.its \
	po/its/polkit.loc \
	$(test_scripts) \
	$(test_data) \
	$(nil)

.PHONY: doc

doc: doc/Doxyfile
	$(DOXYGEN) doc/Doxyfile

doc-clean:
	rm -rf doc/html/

# Code coverage support

.PHONY: coverage coverage-clean

if COVERAGE_ENABLED
LCOV_OPTS = -q --rc geninfo_checksum=1 --rc lcov_branch_coverage=1
LCOV_CAPTURE_OPTS = $(LCOV_OPTS) --no-recursion \
	-d $(top_builddir)/lib/dpkg \
	-d $(top_builddir)/src \
	-d $(top_builddir)/utils
LCOV_INJECT = $(PERL) -i $(top_srcdir)/doc/lcov-inject.pl

coverage: all
	: # Remove coverage data from any previous run
	rm -f *.lcov
	find -name '*.gcda' -o -name '*.gcov' | xargs rm -f

	: # Initialize data
	$(LCOV) $(LCOV_CAPTURE_OPTS) -c -o dpkg_base.lcov -i
	: # Run test cases
	$(MAKE) -C lib/dpkg check
	$(MAKE) -C src check
	$(MAKE) -C utils check
	: # Merge test coverage data
	$(LCOV) $(LCOV_CAPTURE_OPTS) -c -o dpkg_test.lcov
	$(LCOV) $(LCOV_OPTS) -a dpkg_base.lcov -a dpkg_test.lcov \
	  -o dpkg_merge.lcov
	$(LCOV) $(LCOV_OPTS) -r dpkg_merge.lcov '/usr/include/*' -o dpkg.lcov
	: # Generate reports
	$(LCOV) $(LCOV_OPTS) -l dpkg.lcov
	$(LCOV_GENHTML) $(LCOV_OPTS) \
	  --legend --title "dpkg $(VERSION) C code coverage" \
	  -o doc/coverage dpkg.lcov

	$(MAKE) -C scripts $@

	: # XXX: Inject perl coverage into lcov index files. This is a fragile
	: # hack which might break depending on the html output generated.
	$(LCOV_INJECT) doc/coverage/index-sort-b.html
	$(LCOV_INJECT) doc/coverage/index-sort-f.html
	$(LCOV_INJECT) doc/coverage/index-sort-l.html
	$(LCOV_INJECT) doc/coverage/index.html

coverage-clean:
	rm -rf doc/coverage/
	find -name '*.gcno' -o -name '*.gcda' -o \
	     -name '*.gcov' -o -name '*.lcov' | xargs rm -f
	$(MAKE) -C scripts $@
else
coverage:
	@echo "Need to reconfigure with --enable-coverage"

coverage-clean:
endif

test_scripts = \
	t/pod.t \
	t/pod-spell.t \
	t/pod-coverage.t \
	t/synopsis.t \
	t/minimum-version.t \
	t/module-version.t \
	t/syntax.t \
	t/strict.t \
	t/critic.t \
	$(nil)

test_data = \
	t/critic/perlcriticrc \
	$(nil)

# We need to use absolute paths here due to strict.t invoking a sub-perl,
# while changing dir to source dir on out-of-tree builds.
TEST_ENV_VARS = \
	DPKG_DATADIR=$(abs_top_srcdir)/data \
	$(nil)

include $(top_srcdir)/check.am

.PHONY: update-po

update-po:
	$(MAKE) -C po update-po
	$(MAKE) -C scripts/po update-po
	$(MAKE) -C dselect/po update-po
	$(MAKE) -C man update-po

# If we create the dist tarball from the git repository, make sure
# that we're not forgetting some files...
dist-hook:
	echo $(VERSION) >$(distdir)/.dist-version
	if [ -e .git ]; then \
	  for file in `git ls-files | grep -v .gitignore`; do \
	    if [ ! -e "$(distdir)/$$file" ]; then \
	      echo "$$file is missing in $(distdir)" >&2 ; \
	      exit 1 ; \
	    fi ; \
	  done ; \
	  XDG_CONFIG_HOME= HOME= \
	  git log -C --stat 1.15.0.. >$(distdir)/ChangeLog; \
	fi

clean-local: doc-clean coverage-clean check-clean
