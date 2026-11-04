---
name: go-vercel-handlers
description: Guides the design, implementation, and refactoring of Go Vercel serverless functions using Pattern A (separating pure business logic from HTTP transport).
---

# Go Vercel Serverless Handlers (Pattern A)

## Purpose & Architecture
When writing Go serverless functions deployed to Vercel (or similar serverless environments), handlers commonly mix HTTP transport concerns (CORS, method checks, header parsing, body deserialization, status codes) with core domain logic.

**Pattern A: Separate Business Logic from HTTP Transport** enforces a clean architecture where:
1. **Business logic** is a pure, typed function returning domain response data and typed domain errors:
   `func(ctx context.Context, cfg *config.Config, req Req) (Res, error)`
2. **HTTP transport** is managed by a generic adapter (`Handle[Req, Res]`) that handles:
   - CORS and OPTIONS preflight
   - HTTP method enforcement
   - Configuration loading (`config.Load()`)
   - JSON decoding (skipped for `GET` requests)
   - Status code mapping and structured JSON responses (`APIError` -> status/type, default -> 500)
3. **Exported handler function** is a concise one-line adapter call matching Vercel's signature `func(w http.ResponseWriter, r *http.Request)`:
   `httputil.Handle(http.MethodPost, myBusinessLogic)(w, r)`

---

## When to Use This Skill
- Creating new Go serverless functions in Vercel (`api/*.go`).
- Refactoring existing handlers that mix HTTP status handling, JSON decoding, and business logic into unreadable monolithic functions.
- Adding unit test coverage to Go API endpoints without needing to mock `http.ResponseWriter` or HTTP request streams.

---

## Pattern Implementation Guide

### 1. The Generic Adapter (`internal/httputil/adapter.go`)

```go
package httputil

import (
	"context"
	"errors"
	"net/http"

	"your-project/internal/config"
)

// HandlerFunc is a typed handler that returns a response or an error.
// The error, if non-nil, should be an *APIError to control status code and type.
type HandlerFunc[Req any, Res any] func(ctx context.Context, cfg *config.Config, req Req) (Res, error)

// APIError carries HTTP status and error type for the adapter to render.
type APIError struct {
	Status  int
	Type    string
	Message string
	Err     error
}

func (e *APIError) Error() string {
	if e.Message != "" {
		return e.Message
	}
	if e.Err != nil {
		return e.Err.Error()
	}
	return ""
}

func (e *APIError) Unwrap() error { return e.Err }

// NewAPIError creates a new APIError with a custom message.
func NewAPIError(status int, errType, message string) *APIError {
	return &APIError{
		Status:  status,
		Type:    errType,
		Message: message,
	}
}

// WrapAPIError wraps an existing error with an HTTP status and type identifier.
func WrapAPIError(status int, errType string, err error) *APIError {
	if err == nil {
		return nil
	}
	return &APIError{
		Status:  status,
		Type:    errType,
		Message: err.Error(),
		Err:     err,
	}
}

// Handle wires a typed HandlerFunc into an http.HandlerFunc.
// It handles CORS, method enforcement, config loading, JSON decode, and JSON response.
func Handle[Req any, Res any](method string, fn HandlerFunc[Req, Res]) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		if SetCORS(w, r) {
			return
		}
		if r.Method != method {
			WriteError(w, http.StatusMethodNotAllowed, errors.New("method not allowed"), "MethodNotAllowed")
			return
		}
		cfg := config.Load()
		var req Req
		if method != http.MethodGet {
			if err := DecodeJSON(r, &req); err != nil {
				WriteError(w, http.StatusBadRequest, err, "BadRequest")
				return
			}
		}
		res, err := fn(r.Context(), &cfg, req)
		if err != nil {
			var apiErr *APIError
			if errors.As(err, &apiErr) {
				WriteError(w, apiErr.Status, apiErr, apiErr.Type)
			} else {
				WriteError(w, http.StatusInternalServerError, err, "ServerError")
			}
			return
		}
		_ = WriteJSON(w, http.StatusOK, res)
	}
}
```

---

### 2. Structuring Handlers (`api/<endpoint>.go`)

Each endpoint file should follow this structure:

```go
package handler

import (
	"context"
	"net/http"

	"your-project/internal/config"
	"your-project/internal/httputil"
)

// 1. Request and Response DTOs
type searchRequest struct {
	Query string `json:"query"`
	Limit int    `json:"limit,omitempty"`
}

type searchResponse struct {
	Items []Item `json:"items"`
}

// 2. Pure Business Logic Function
// - Accepts context, config, and typed request.
// - Returns typed response and error (*httputil.APIError for controlled HTTP responses).
// - Independent of http.ResponseWriter.
func searchItems(ctx context.Context, cfg *config.Config, req searchRequest) (searchResponse, error) {
	if req.Query == "" {
		return searchResponse{}, httputil.NewAPIError(http.StatusBadRequest, "BadRequest", "query cannot be empty")
	}

	items, err := service.Find(ctx, req.Query, req.Limit)
	if err != nil {
		return searchResponse{}, httputil.WrapAPIError(http.StatusInternalServerError, "SearchError", err)
	}

	return searchResponse{Items: items}, nil
}

// 3. Exported Vercel Handler Function
// Single one-liner delegating to httputil.Handle
func Search(w http.ResponseWriter, r *http.Request) {
	httputil.Handle(http.MethodPost, searchItems)(w, r)
}
```

---

### 3. GET Requests Without a Body
For `GET` or bodyless endpoints, use `struct{}` as the `Req` type:

```go
func getStatus(_ context.Context, cfg *config.Config, _ struct{}) (statusResponse, error) {
	return statusResponse{Status: "ok"}, nil
}

func Status(w http.ResponseWriter, r *http.Request) {
	httputil.Handle(http.MethodGet, getStatus)(w, r)
}
```

---

### 4. Unit Testing Pure Business Functions
Because the business logic functions are pure Go functions, you can test validation and business flows without constructing HTTP requests or mock recorders:

```go
func TestSearchItemsValidation(t *testing.T) {
	cfg := &config.Config{}
	_, err := searchItems(context.Background(), cfg, searchRequest{Query: ""})
	if err == nil {
		t.Fatal("expected error for empty query")
	}
	var apiErr *httputil.APIError
	if !errors.As(err, &apiErr) || apiErr.Status != http.StatusBadRequest {
		t.Fatalf("expected 400 BadRequest, got %v", err)
	}
}
```

---

## Best Practices & Checklist
- [ ] Keep request and response structs typed with `json:` tags.
- [ ] For missing or optional slices, return empty initialized slices (`[]Item{}`) instead of `nil` to ensure JSON encodes as `[]` rather than `null`.
- [ ] Use `r.Context()` from the adapter to support request cancellation and deadline propagation.
- [ ] Map known business/client errors to `httputil.NewAPIError` or `httputil.WrapAPIError` with distinct `Type` strings for frontend error discrimination.
- [ ] Keep the exported function name matching Vercel's endpoint conventions (`api/<endpoint>.go` exporting `func EndpointName(w http.ResponseWriter, r *http.Request)`).

