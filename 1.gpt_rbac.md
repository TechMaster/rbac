Creating a Go web application that uses Role-Based Access Control (RBAC) to allow or deny HTTP requests based on URL paths, methods, and roles involves several steps. Below is a simplified example to get you started. This example uses the `net/http` package for the web server and a custom middleware for authorization.

First, let's define the necessary structures and middleware for handling RBAC:

```go
package main

import (
	"fmt"
	"net/http"
	"strings"
)

// Role-based access control rules
type Rule struct {
	Path    string
	Method  string
	Allow   []string
	Deny    []string
	AllowAll bool
	DenyAll  bool
}

// User structure
type User struct {
	Username string
	Roles    []string
}

// Middleware for RBAC
func RBACMiddleware(rules []Rule, next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		user := getUserFromRequest(r) // Assume this function extracts user info from the request

		for _, rule := range rules {
			if strings.HasPrefix(r.URL.Path, rule.Path) && r.Method == rule.Method {
				if rule.DenyAll {
					http.Error(w, "Access Denied", http.StatusForbidden)
					return
				}
				if rule.AllowAll {
					next.ServeHTTP(w, r)
					return
				}
				if containsAny(user.Roles, rule.Deny) {
					http.Error(w, "Access Denied", http.StatusForbidden)
					return
				}
				if containsAny(user.Roles, rule.Allow) {
					next.ServeHTTP(w, r)
					return
				}
				http.Error(w, "Access Denied", http.StatusForbidden)
				return
			}
		}
		next.ServeHTTP(w, r)
	})
}

// Helper function to check if any role is in the list
func containsAny(roles, list []string) bool {
	for _, role := range roles {
		for _, item := range list {
			if role == item {
				return true
			}
		}
	}
	return false
}

// Dummy function to get user from request
func getUserFromRequest(r *http.Request) User {
	// In a real application, you would extract user information from the request (e.g., from a JWT token)
	return User{
		Username: "testuser",
		Roles:    []string{"roleA"},
	}
}

func main() {
	rules := []Rule{
		{Path: "/api/book", Method: "GET", Allow: []string{"roleA", "roleB"}},
		{Path: "/api/book", Method: "POST", Deny: []string{"roleB", "roleC"}},
		{Path: "/api/person", Method: "GET", AllowAll: true},
		{Path: "/api/person", Method: "DELETE", DenyAll: true},
	}

	mux := http.NewServeMux()
	mux.HandleFunc("/api/book", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Book API")
	})
	mux.HandleFunc("/api/person", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "Person API")
	})

	http.ListenAndServe(":8080", RBACMiddleware(rules, mux))
}
```

### Explanation:

1. **Rule Structure**: Defines the RBAC rules with fields for path, method, allowed roles, denied roles, and flags for allowing or denying all roles.

2. **User Structure**: Represents a user with a username and roles.

3. **RBAC Middleware**: This middleware checks the incoming request against the defined rules. It allows or denies access based on the rules.

4. **Helper Functions**: 
   - `containsAny` checks if any of the user's roles are in the allowed or denied lists.
   - `getUserFromRequest` is a placeholder function to extract user information from the request. In a real application, this would typically involve parsing a JWT token or session data.

5. **Main Function**: 
   - Defines the RBAC rules.
   - Sets up the HTTP server with the RBAC middleware and handlers for the `/api/book` and `/api/person` endpoints.

This example provides a basic framework for implementing RBAC in a Go web application. In a real-world application, you would need to handle user authentication, possibly using JWT tokens, and securely manage user roles and permissions.