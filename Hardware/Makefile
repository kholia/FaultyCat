PROJECT := FaultyCat
BOARD := $(PROJECT).kicad_pcb
SCHEMATIC := $(PROJECT).kicad_sch
PCB_TEST_SCRIPT := check_nets_pcbnew_v5.py
PCB_TESTS := pcb_tests.txt
KICAD_PYTHON := /Applications/KiCad/kicad.app/Contents/Frameworks/Python.framework/Versions/Current/bin/python3

KICAD_CLI ?= kicad-cli
REPORT_DIR ?= /tmp/$(PROJECT)-checks
DRC_REPORT ?= $(REPORT_DIR)/drc.rpt
ERC_REPORT ?= $(REPORT_DIR)/erc.rpt
RENDER_3D_OUTPUT ?= 3D-Render.png
RENDER_3D_SIDE ?= top
RENDER_3D_WIDTH ?= 1600
RENDER_3D_HEIGHT ?= 1200
RENDER_3D_ZOOM ?= 0.8
RENDER_3D_BACKGROUND ?= opaque
RENDER_3D_LIGHT_TOP ?= 0.5
RENDER_3D_LIGHT_CAMERA ?= 0.25

.PHONY: all stuff check pcb-tests extract-pcb-tests verify-pcb-tests drc erc 3d-render help

all: check

stuff: all

check: pcb-tests drc erc

pcb-tests:
	$(KICAD_PYTHON) $(PCB_TEST_SCRIPT) $(BOARD) \
		--test-file $(PCB_TESTS) \
		--check-coverage

extract-pcb-tests:
	$(KICAD_PYTHON) $(PCB_TEST_SCRIPT) $(BOARD) \
		--generate-test-file $(PCB_TESTS)

verify-pcb-tests:
	@tmp_file=$$(mktemp /tmp/$(PROJECT)-pcb-tests.XXXXXX); \
	trap 'rm -f "$$tmp_file"' EXIT; \
	$(KICAD_PYTHON) $(PCB_TEST_SCRIPT) $(BOARD) \
		--generate-test-file "$$tmp_file" >/dev/null; \
	cmp -s $(PCB_TESTS) "$$tmp_file" || { \
		echo "$(PCB_TESTS) is stale; run 'make extract-pcb-tests'"; \
		diff -u $(PCB_TESTS) "$$tmp_file"; \
		exit 1; \
	}
	@echo "$(PCB_TESTS) matches $(BOARD)"

drc:
	mkdir -p $(REPORT_DIR)
	$(KICAD_CLI) pcb drc \
		--exit-code-violations \
		--schematic-parity \
		--severity-all \
		--output $(DRC_REPORT) \
		$(BOARD)

erc:
	mkdir -p $(REPORT_DIR)
	$(KICAD_CLI) sch erc \
		--exit-code-violations \
		--severity-all \
		--output $(ERC_REPORT) \
		$(SCHEMATIC)

3d-render:
	$(KICAD_CLI) pcb render \
                --side $(RENDER_3D_SIDE) \
                --background $(RENDER_3D_BACKGROUND) \
                --width $(RENDER_3D_WIDTH) \
                --height $(RENDER_3D_HEIGHT) \
                --zoom $(RENDER_3D_ZOOM) \
                --light-top $(RENDER_3D_LIGHT_TOP) \
                --light-camera $(RENDER_3D_LIGHT_CAMERA) \
		--output $(RENDER_3D_OUTPUT) \
		$(BOARD)

help:
	@echo "make check       Run PCB net assertions, DRC/parity, and ERC"
	@echo "make pcb-tests   Check every footprint pad-to-net assertion"
	@echo "make extract-pcb-tests  Regenerate $(PCB_TESTS) from $(BOARD)"
	@echo "make verify-pcb-tests   Verify $(PCB_TESTS) matches $(BOARD)"
	@echo "make drc         Run PCB DRC and schematic-parity checks"
	@echo "make erc         Run schematic ERC"
	@echo "make 3d-render   Render the board to $(RENDER_3D_OUTPUT)"
