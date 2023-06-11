---
title: "Go: Bad table-driven tests"
summary: "When to avoid table-driven tests"
date: 2022-05-22
series: []
categories: ["Go"]
tags: ["go", "golang", "unit-testing"]
weight: 99
aliases: []
author: "Shivam Mamgain"
---

[Table-driven tests](https://dave.cheney.net/2019/05/07/prefer-table-driven-tests) are great because they remove the need for duplicate testing setup for similar test cases. In following this approach, the input and the expected output of these test cases are conveniently placed in a list (the table).

I’ve come across folks trying to write a table-driven test even though it increases the complexity of the test (anecdotal?). For such instances, separate tests for the test cases would’ve resulted in easier-to-understand code.

To demonstrate what I seen, here’s a sample HTTP handler that accepts a POST JSON request to create a TODO:

```go
func (c *Controller) CreateTODO(w http.ResponseWriter, r *http.Request) {
	// Is method allowed
	if r.Method != http.MethodPost {
		http.Error(w, "method is not POST", http.StatusMethodNotAllowed)
		return
	}

	// Authentication call
	token := r.Header.Get("AuthToken")
	if !c.auth.IsAuthenticated(token) {
		http.Error(w, "unauthenticated", http.StatusUnauthorized)
		return
	}

	// Decoding and validation
	todo := &TODO{}
	if err := json.NewDecoder(r.Body).Decode(todo); err != nil {
		http.Error(w, "invalid json: "+err.Error(), http.StatusBadRequest)
		return
	}
	if err := todo.Validate(); err != nil {
		http.Error(w, "invalid todo: "+err.Error(), http.StatusBadRequest)
		return
	}

	// Database call
	if err := c.db.CreateTODO(todo); err != nil {
		http.Error(w, "db error: "+err.Error(), http.StatusInternalServerError)
		return
	}

	respond(w, 201, "todo created")
}
```

There are broadly 4 steps here before a TODO is considered to be successfully created:

1. Validation of allowed method (POST)
2. Authentication via an auth client
3. Decoding of JSON from request body, and validation of fields
4. Database INSERT call via a DB client/ORM

The auth client `c.auth` and db client `c.db` are fields of the `Controller` struct.

```go
type Controller struct {
	auth Authenticator
	db   Database
}

type Authenticator interface {
	IsAuthenticated(token string) bool
}

type Database interface {
	CreateTODO(todo *TODO) error
}
```

Mocks can be generated for these interfaces to be used in unit tests.


## Good table-driven test

The 3rd step (Decoding of JSON from request body, and validation of fields) from the above is a great candidate for table-driven tests because there could be multiple test inputs that can share the same testing setup. Here’s a small list of these:
* Invalid JSON in the request body
* Empty TODO name
* Empty TODO category

All of these will result in Bad Requests, albeit different response bodies.

A table-driven test for it would look something like this:

```go
func TestController_CreateTODO_BadRequestErrors(t *testing.T) {
	testCases := []struct {
		name             string
		requestBody      string
		expectedResponse string
	}{
		{"invalid json", `{"name"}`, "invalid json: invalid character '}' after object key\n"},
		{"empty name", `{"name": ""}`, "invalid todo: empty name\n"},
		{"empty category", `{"name": "task1", "category": ""}`, "invalid todo: empty category\n"},
	}
	for _, tc := range testCases {
		t.Run(tc.name, func(t *testing.T) {
			// Setup mocks
			c := gomock.NewController(t)
			defer c.Finish()
			auth := mock.NewMockAuthenticator(c)
			db := mock.NewMockDatabase(c)
			ctrl := api.NewController(auth, db)

			// Setup response recorder and request
			w := httptest.NewRecorder()
			rBody := bytes.NewBufferString(tc.requestBody)
			r := httptest.NewRequest(http.MethodPost, "http://example.com/todos", rBody)
			r.Header.Add("AuthToken", testToken)

			// Expectations from mocks
			auth.EXPECT().IsAuthenticated(testToken).Return(true)

			// Call HTTP handler
			ctrl.CreateTODO(w, r)
			resp := w.Result()

			// Assertions
			assertEqual(t, 400, resp.StatusCode)
			assertEqual(t, tc.expectedResponse, responseBody(resp))
		})
	}
}
```

Full code for this snippet can be found in this file: [controller_test.go](https://github.com/shivamMg/table-driven-tests-go/blob/master/api/controller_test.go).


## Bad table-driven test

If you try to write a table-driven test that covers 1st, 2nd, and 4th step, and the success case, it would look like this:

```go
func TestController_CreateTODO_BadTableDrivenTest(t *testing.T) {
	testCases := []struct {
		name string

		requestMethod string

		expectAuthCall bool
		authCallReturn bool

		expectDBCall bool
		dbCallReturn error

		expectedStatusCode int
		expectedResponse   string
	}{
		{"method not allowed", http.MethodGet, false, false, false, nil, 405, "method is not POST\n"},
		{"unauthenticated", http.MethodPost, true, false, false, nil, 401, "unauthenticated\n"},
		{"db error", http.MethodPost, true, true, true, errors.New("failed to commit txn"), 500, "db error: failed to commit txn\n"},
		{"success", http.MethodPost, true, true, true, nil, 201, "todo created"},
	}
	for _, tc := range testCases {
		t.Run(tc.name, func(t *testing.T) {
			// Setup mocks …
			// Setup response recorder and request …

			if tc.expectAuthCall {
				auth.EXPECT().IsAuthenticated(testToken).Return(tc.authCallReturn)
			}
			if tc.expectDBCall {
				db.EXPECT().CreateTODO(&api.TODO{"task1", "cat1"}).Return(tc.dbCallReturn)
			}

			ctrl.CreateTODO(w, r)
			resp := w.Result()

			assertEqual(t, tc.expectedStatusCode, resp.StatusCode)
			assertEqual(t, tc.expectedResponse, responseBody(resp))
		})
	}
}
```

Notice in the table:
1. `requestMethod` is really needed only for the “method not allowed” case but has to be present even for other test cases.
2. `expectAuthCall` and `expectDBCall` are needed to set mock expectations (`EXPECT()`) for certain test cases.
3. `authCallReturn` and `dbCallReturn` are needed by `EXPECT()` calls.

Overall, trying to fit multiple dissimilar cases into a table-driven test has resulted in a more complicated test.

Contrarily, if you were to separate these test cases into separate tests, then granted, there will be more code, but the tests will be easier to understand. For instance, if the “method not allowed” case was its own separate test, it would look something like this:

```go
func TestController_CreateTODO_MethodNotAllowed(t *testing.T) {
	// Setup mocks …
	// Setup response recorder and request …

	ctrl.CreateTODO(w, r)
	resp := w.Result()

	assertEqual(t, 405, resp.StatusCode)
	assertEqual(t, "method is not POST\n", responseBody(resp))
}
```

The test case itself is simple because no client calls are expected, but adding it as a part of the table-driven test made it more complicated.

If this HTTP handler doesn’t look complex enough, then understand that most HTTP handlers are way more sophisticated. For instance, they might use more clients (besides auth and db) for external services (e.g. cache). Also, `EXPECT()` calls might require different arguments for different test cases, which will then need to be part of the table - making the table bigger.


## Conclusion

Obvious to many, but not to some: Avoid writing a table-driven test where test cases have different testing setups.

Working sample code for everything here can be found in this repo: [github.com/shivamMg/table-driven-tests-go](https://github.com/shivamMg/table-driven-tests-go).

All tests shown here can be found in this file: [controller_test.go](https://github.com/shivamMg/table-driven-tests-go/blob/master/api/controller_test.go).

Thank you [Avinash](https://avi.im/) for reviewing the post's draft.

---
