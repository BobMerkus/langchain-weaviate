.PHONY: all format lint test tests integration_tests docker_tests help extended_tests update-weaviate-image

# Default target executed when no arguments are given to make.
all: help

update-weaviate-image:
	$(eval DOCKER_COMPOSE_FILE := tests/docker-compose.yml)

	@echo "Fetching the latest Weaviate version..."
	$(eval LATEST_VERSION := $(shell curl -s https://api.github.com/repos/weaviate/weaviate/releases/latest | jq -r '.tag_name | ltrimstr("v")'))
	@echo "Latest Weaviate version fetched: $(LATEST_VERSION)"
	
	$(eval CURRENT_VERSION := $(shell grep "semitechnologies/weaviate:" $(DOCKER_COMPOSE_FILE) | sed -E 's/.*semitechnologies\/weaviate:(.*)/\1/'))

	@if [ "$(CURRENT_VERSION)" != "$(LATEST_VERSION)" ]; then \
		echo "Updating Weaviate version from $(CURRENT_VERSION) to $(LATEST_VERSION)..."; \
		if sed -i "s|semitechnologies/weaviate:$(CURRENT_VERSION)|semitechnologies/weaviate:$(LATEST_VERSION)|" $(DOCKER_COMPOSE_FILE); then \
			echo "Update successful. Weaviate version is now $(LATEST_VERSION)"; \
		else \
			echo "Update failed." >&2; \
		fi \
	else \
		echo "No update required. Current Weaviate version is already $(LATEST_VERSION)"; \
	fi

TEST_FILE ?= tests/unit_tests/

integration_test integration_tests: TEST_FILE=tests/integration_tests/

test tests:
	uv run pytest $(TEST_FILE) --cov=langchain_weaviate --cov-report term-missing --cov-fail-under=96.44

integration_test integration_tests: update-weaviate-image
	uv run pytest $(TEST_FILE) --cov=langchain_weaviate --cov-report term-missing

######################
# LINTING AND FORMATTING
######################

# Define a variable for Python and notebook files.
PYTHON_FILES=.
MYPY_CACHE=.mypy_cache
lint format: PYTHON_FILES=.
lint_diff format_diff: PYTHON_FILES=$(shell git diff --name-only --diff-filter=d master | grep -E '\.py$$|\.ipynb$$')
lint_package: PYTHON_FILES=langchain_weaviate
lint_tests: PYTHON_FILES=tests
lint_tests: MYPY_CACHE=.mypy_cache_test

lint lint_diff lint_package lint_tests:
	uv run ruff check .
	uv run ruff format $(PYTHON_FILES) --diff
	uv run ruff check --select I $(PYTHON_FILES)
	mkdir $(MYPY_CACHE); uv run mypy $(PYTHON_FILES) --cache-dir $(MYPY_CACHE)

format format_diff:
	uv run ruff format $(PYTHON_FILES)
	uv run ruff check --fix $(PYTHON_FILES)

spell_check:
	uv run codespell --toml pyproject.toml

spell_fix:
	uv run codespell --toml pyproject.toml -w

check_imports: $(shell find langchain_weaviate -name '*.py')
	uv run python ./scripts/check_imports.py $^

######################
# HELP
######################

help:
	@echo '----'
	@echo 'check_imports				- check imports'
	@echo 'format                       - run code formatters'
	@echo 'lint                         - run linters'
	@echo 'test                         - run unit tests'
	@echo 'tests                        - run unit tests'
	@echo 'test TEST_FILE=<test_file>   - run all tests in file'
