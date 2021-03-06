dash := /bin/dash
ifeq ($(wildcard $(dash)), $(dash))
    SHELL=$(dash)
endif

.PHONY: help
help:
	@echo "BUILD/INSTALL INSTRUCTIONS"
	@echo
	@echo "to build Firetray, just:"
	@echo "  make build"
	@echo
	@echo "by default, debug calls are stripped from js files and DEBUG_MODE"
	@echo "is off (performance). If you want to keep debug calls:"
	@echo "  DEBUG=on make build"
	@echo
	@echo "to create the dev profile:"
	@echo "  firefox -no-remote -P		# then create '$(profile_id)'"
	@echo "  thunderbird -no-remote -P	# then create '$(profile_id)'"
	@echo
	@echo "to test the extension with the dev profile:"
	@echo "  cd ~/.mozilla/firefox/mozilla-dev/extensions"
	@echo "  ln -s ../../path/to/firetray/src '{9533f794-00b4-4354-aa15-c2bbda6989f8}'"
	@echo "  cd ~/.thunderbird/mozilla-dev/extensions"
	@echo "  ln -s ../path/to/firetray/src '{9533f794-00b4-4354-aa15-c2bbda6989f8}'"
	@echo
	@echo "to test with dev profile:"
	@echo "  firefox -no-remote -P mozilla-dev"
	@echo "  thunderbird -no-remote -P mozilla-dev"
	@echo
	@echo "Have fun !"

# The UUID of the extension.
extension_name := firetray

# The name of the profile dir where the extension can be installed.
profile_id := mozilla-dev

# The zip application to be used. NOTE: symlinks seem not supported in XPI
# ("could not be installed because Firefox cannot modify the needed file")
ZIP := zip # --symlinks

ifdef DEBUG
build_debug := -debug
copy_and_strip_maybe =				\
@mkdir -p $(dir $(1));				\
cp -f $(2) $(1);
else
copy_and_strip_maybe =				\
@mkdir -p $(dir $(1));				\
echo "Stripping debug calls from JS file $<";	\
sed '/log.debug(/d' $(2) > $(1);
endif

# The target location of the build and build files.
SCM-REVISION = $(shell git rev-parse --short HEAD)
build_dir := ../build-$(SCM-REVISION)$(build_debug)


# The location of the extension profile. (this extension is intended for Linux only)
profile_locations := \
  ~/.mozilla/firefox/$(profile_id)/extensions \
  ~/.thunderbird/$(profile_id)/extensions

# The license file
license := LICENSE

# The install.rdf file.
install_rdf := install.rdf

# Version fetched from install.rdf
VERSION := $(shell awk -F '[<>]' '/em:version/{print $$3}' $(install_rdf))

# The target XPI files.
xpi_file := $(extension_name)-$(VERSION).xpi
xpi_built := $(build_dir)/$(xpi_file)
# Since we use <em:unpack>false, we need the same name across versions
xpi_deployed := $(extension_name).xpi

# The chrome.manifest file.
chrome_manifest := chrome.manifest

# The preferences dir.
preferences_dir := defaults/preferences

# The root of the chrome sources.
chrome_source_root := chrome

# The chrome sources.
chrome_sources_js := $(wildcard $(chrome_source_root)/content/*.js)
chrome_sources := $(chrome_sources_js)								\
               $(wildcard $(chrome_source_root)/content/*.xul)					\
               $(wildcard $(chrome_source_root)/content/*.xml)					\
               $(wildcard $(chrome_source_root)/content/*.css)					\
               $(wildcard $(chrome_source_root)/skin/*.css)					\
               $(wildcard $(chrome_source_root)/skin/icons/*.gif)				\
               $(wildcard $(chrome_source_root)/skin/icons/*.png)				\
               $(wildcard $(chrome_source_root)/skin/icons/*.svg)				\
               $(wildcard $(chrome_source_root)/skin/icons/linux/hicolor/22x22/*/*.png)		\
               $(wildcard $(chrome_source_root)/skin/icons/winnt/*.bmp)				\
               $(wildcard $(chrome_source_root)/skin/icons/winnt/*.ico)				\
               $(wildcard $(chrome_source_root)/locale/*/*.dtd)					\
               $(wildcard $(chrome_source_root)/locale/*/*.properties)

# The modules (JSM) dir.
modules_dir := modules

# The sources for the module files.
modules_sources := $(wildcard $(modules_dir)/*.js)			\
		$(wildcard $(modules_dir)/*.jsm)			\
		$(wildcard $(modules_dir)/ctypes/*.jsm)			\
		$(wildcard $(modules_dir)/ctypes/linux/*.jsm)		\
		$(wildcard $(modules_dir)/ctypes/linux/gtk?/*.jsm)	\
		$(wildcard $(modules_dir)/ctypes/winnt/*.jsm)		\
		$(wildcard $(modules_dir)/linux/*.jsm)			\
		$(wildcard $(modules_dir)/winnt/*.jsm)

# The components (JSM) dir.
components_dir := components

# The JS component source files
components_sources := $(wildcard $(components_dir)/*.js)

# The sources for the XPI file. Uses variables defined in the included
# Makefiles.
xpi_includes := $(license) \
             $(install_rdf) \
             $(chrome_manifest) \
             $(preferences_dir)/prefs.js \
             $(chrome_sources) \
             $(modules_sources) \
             $(components_sources)

# Destination files
build_includes := $(foreach f,$(xpi_includes),$(build_dir)/$(f))


$(xpi_built): check_version check_loglevel $(build_dir) $(build_includes)
	@echo "Creating XPI file."
	@cd $(build_dir); $(ZIP) $(xpi_file) $(xpi_includes)
	@echo "Creating XPI file. Done!"

# This builds the extension XPI file.
.PHONY: build
build: $(xpi_built)
	@echo
	@echo "Build finished successfully in $(build_dir)."
	@echo

# This cleans all temporary files and directories created by 'make'.
.PHONY: clean
clean: clean_build
	@echo "Cleanup is done."

# called via $(build_includes)
$(build_dir)/%: %
	@mkdir -p $(dir $@)
	@cp -f $< $@ # -d for symlinks

$(build_dir)/$(license): ../$(license)
	@cp -f $< $@

# Inject SCM revision stamp. This may be useful in case we mess up with tags
$(build_dir)/$(install_rdf): $(install_rdf)
	@sed 's/\/\* SCM-REVISION:/\/* SCM-REVISION: $(SCM-REVISION) /' $< > $@

# Debug calls are removed for performance.
# NOTE: we could also use m4 for filtering source files...
$(build_dir)/$(chrome_source_root)/%.js: $(chrome_source_root)/%.js
	$(call copy_and_strip_maybe,$@,$<)

$(build_dir)/$(modules_dir)/%: $(modules_dir)/%
	$(call copy_and_strip_maybe,$@,$<)

$(build_dir)/$(components_dir)/%.js: $(components_dir)/%.js
	$(call copy_and_strip_maybe,$@,$<)

$(build_dir):
	@if [ ! -x $(build_dir) ];	\
  then					\
    mkdir -p $(build_dir);		\
  fi

$(profile_locations):
	@echo "Creating extension folder: $(profile_locations)"
	@for p in $(profile_locations) ; do	\
  if [ ! -x "$$p" ];				\
  then						\
    mkdir -p $$p;				\
  fi;						\
  done

clean_build:
	@echo "Removing build dirs: $(build_dir)*"
	@rm -rf $(build_dir)*

VERSION_HARD_CODED := $(shell awk -F\" '/const[ \t]+FIRETRAY_VERSION/ \
  {print $$2}' modules/commons.js)
check_version:
	@echo "checking version consistency"
	@[ "$(VERSION)" = "$(VERSION_HARD_CODED)" ]

FIRETRAY_LOG_LEVEL := $(shell awk -F\" '/const[ \t]+FIRETRAY_LOG_LEVEL/ \
  {print $$2}' modules/logging.jsm)
check_loglevel:
	@echo "checking loglevel"
	@[ "Warn" = "$(FIRETRAY_LOG_LEVEL)" ]
