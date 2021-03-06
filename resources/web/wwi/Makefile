# Copyright 1996-2020 Cyberbotics Ltd.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

JS_SOURCES = \
  shaders/hdr_resolve.js \
  shaders/bright_pass.js \
  shaders/blend_bloom.js \
  shaders/gaussian_blur_13_tap.js \
  passes/bloom.js \
  gpu_picker.js \
  util.js \
  system_info.js \
  dialog_window.js \
  console.js \
  help_window.js \
  robot_window.js \
  toolbar.js \
  editor.js \
  context_menu.js \
  viewpoint.js \
  selector.js \
  mouse_events.js \
  multimedia_client.js \
  texture_loader.js \
  default_url.js \
  server.js \
  stream.js \
  x3d.js \
  x3d_scene.js \
  animation.js \
  webots.js

BABEL_JS_SOURCES = \
  node_modules/@babel/polyfill/dist/polyfill.js

ifneq ($(MAKECMDGOALS),debug)
THREEJS_SUFFIX = .min
endif

DEPENDENCIES_PATH = dependencies
THEEEJS_VERSION = 104
THREEJS_SOURCE = $(DEPENDENCIES_PATH)/three$(THREEJS_SUFFIX).js
THREEJS_URL = https://cdnjs.cloudflare.com/ajax/libs/three.js/$(THEEEJS_VERSION)/three$(THREEJS_SUFFIX).js

null :=
space := $(null) $(null)
WEBOTS_HOME_PATH=$(subst $(space),\ ,$(strip $(subst \,/,$(WEBOTS_HOME))))
include $(WEBOTS_HOME_PATH)/resources/Makefile.os.include

ifeq ($(OSTYPE),darwin)
 WGET=LANG=en_US.UTF-8 wget -qN -O
else
 WGET=wget -qN -O
endif

# external sources to be downloaded and minified
THREEJS_EXAMPLE_URL = https://cdn.jsdelivr.net/gh/mrdoob/three.js@r$(THEEEJS_VERSION)/examples/js/
THREEJS_EXAMPLE_SOURCES = \
  loaders/RGBELoader.js \
  pmrem/PMREMCubeUVPacker.js \
  pmrem/PMREMGenerator.js \
  shaders/CopyShader.js \
  shaders/FXAAShader.js \
  postprocessing/EffectComposer.js \
  postprocessing/RenderPass.js \
  postprocessing/ShaderPass.js
LOCAL_THREEJS_EXAMPLE_SOURCES = $(addprefix $(DEPENDENCIES_PATH)/,$(THREEJS_EXAMPLE_SOURCES))

# Token file used to determine when the last successful installation occurred.
INSTALL_TOKEN = install.token

.PHONY: release debug profile update cleanse clean

NPM_EXISTS = $(shell which npm 2> /dev/null)

ifeq ($(NPM_EXISTS),)
release debug profile update:
	@echo "# npm not found in PATH:" $(PATH)
	@echo "# Please install nodejs to build 'webots.min.js' using the instructions on the GitHub wiki."
	@echo -e "# \033[0;33mnpm not installed, skipping webots.min.js\033[0m"
else
release profile update: webots.min.js webots.lib.js
debug: webots.debug.js
endif

webots.min.js: $(JS_SOURCES) $(LOCAL_THREEJS_EXAMPLE_SOURCES) $(THREEJS_SOURCE) $(INSTALL_TOKEN)
	@echo "# compressing webots.min.js"
	@node_modules/.bin/babel $(LOCAL_THREEJS_EXAMPLE_SOURCES) $(JS_SOURCES) $(BABEL_JS_SOURCES) -o webots.tmp1.js --presets "@babel/preset-env"
	@cat $(THREEJS_SOURCE) webots.tmp1.js > webots.tmp2.js
	@node_modules/.bin/minify webots.tmp2.js -o $@
	@rm webots.tmp2.js webots.tmp1.js

webots.debug.js: $(JS_SOURCES) $(LOCAL_THREEJS_EXAMPLE_SOURCES) $(THREEJS_SOURCE) $(INSTALL_TOKEN)
	@echo "# creating webots.debug.js"
	@cat $(THREEJS_SOURCE) $(LOCAL_THREEJS_EXAMPLE_SOURCES) $(JS_SOURCES) $(BABEL_JS_SOURCES) > $@

# JS module intended to be included in another module.
# webots.lib.js is not processed with babel or minify and does not include three.js.
webots.lib.js: $(LOCAL_THREEJS_EXAMPLE_SOURCES) $(JS_SOURCES) $(INSTALL_TOKEN)
	@echo "# compressing webots.lib.js"
	@cat $(LOCAL_THREEJS_EXAMPLE_SOURCES) $(JS_SOURCES) > $@

$(THREEJS_SOURCE):
	@echo "# downloading $(THREEJS_URL)"
	@mkdir -p $(DEPENDENCIES_PATH)
	@$(WGET) $@ $(THREEJS_URL)

$(LOCAL_THREEJS_EXAMPLE_SOURCES):
	$(eval FILENAME=$(@:$(DEPENDENCIES_PATH)/%=%))
	@echo "# downloading three.js $@"
	@mkdir -p $(dir $@)
	@$(WGET) $@ $(THREEJS_EXAMPLE_URL)$(FILENAME)

$(INSTALL_TOKEN): package.json
	@echo "# installing npm dependencies"
	@npm --silent install 1> /dev/null
	@touch $(INSTALL_TOKEN)

cleanse: clean
	@echo "# uninstalling npm dependencies"
	@rm -fr node_modules $(INSTALL_TOKEN) package-lock.json
ifneq ($(NPM_EXISTS),)
	@npm --silent uninstall 1> /dev/null
endif

clean:
	@rm -fr *.min.js webots.debug.js webots.lib.js $(DEPENDENCIES_PATH)
