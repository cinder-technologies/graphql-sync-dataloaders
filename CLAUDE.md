# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is `graphql-sync-dataloaders`, a Python library that enables the use of DataLoaders in GraphQL servers running in synchronous contexts (primarily Django). It solves the problem of efficient data loading through batching in environments where async/await cannot be used.

## Common Development Commands

### Setup
```bash
# Install dependencies using Poetry
poetry install
```

### Testing
```bash
# Run all tests
poetry run pytest

# Run only non-Django tests
poetry run python -m pytest tests/ --showlocals -vv -m "not django"

# Run only Django tests
poetry run python -m pytest --showlocals -vv -m django

# Run a specific test file
poetry run pytest tests/test_dataloader.py -v
```

### Code Formatting
```bash
# Format code with Black
poetry run black .

# Check formatting without changes
poetry run black --check .
```

## Architecture Overview

The library implements a synchronous version of the DataLoader pattern through four main components:

1. **SyncDataLoader** (`sync_dataloader.py`): Main class that batches multiple load requests into a single batch function call. Supports caching and custom cache key functions.

2. **SyncFuture** (`sync_future.py`): A synchronous implementation of futures/promises that allows chaining operations with `.then()`. This enables deferred execution without async/await.

3. **DeferredExecutionContext** (`execution_context.py`): Custom GraphQL execution context that intercepts SyncFuture objects returned from resolvers and processes them after the initial execution pass, enabling the batching behavior.

4. **Integration Points**: The library integrates with both Strawberry and Graphene-Django by passing `DeferredExecutionContext` as the execution context class when setting up the GraphQL schema/view.

## Key Implementation Details

- **Batching Flow**: When `loader.load(key)` is called, it returns a SyncFuture immediately. The actual batch function is called later when the execution context processes all accumulated futures.

- **Chaining**: SyncFutures support `.then()` for transforming results, allowing complex data loading patterns while maintaining synchronous execution.

- **Cache Key Functions**: Recent addition (commit f63e5aa) allows custom cache key generation for more flexible caching strategies.

- **Django Integration**: The library is designed specifically for Django's synchronous ORM, with tests demonstrating integration with both Strawberry and Graphene Django GraphQL libraries.

## Testing Approach

Tests are organized into:
- Unit tests for core functionality (`test_dataloader.py`, `test_sync_future.py`)
- Integration tests with GraphQL libraries (`test_graphene_django.py`, `test_strawberry_django.py`)
- Django-specific tests are marked with `@pytest.mark.django`

The test Django app in `tests/django/app/` provides models and schemas for integration testing.