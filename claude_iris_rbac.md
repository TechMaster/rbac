Certainly! I'll rewrite the RBAC authorization system using the Iris web framework. Iris is a fast, simple, and efficient web framework for Go. Here's the implementation using Iris:

```go
package main

import (
	"github.com/kataras/iris/v12"
)

// Role represents a user role
type Role string

// Rule represents an authorization rule
type Rule struct {
	Path   string
	Method string
	Auth   AuthFunc
}

// AuthFunc is a function type for authorization checks
type AuthFunc func([]Role) bool

// User represents a user with roles
type User struct {
	Username string
	Roles    []Role
}

// Allow creates an AuthFunc that allows only specified roles
func Allow(allowedRoles ...Role) AuthFunc {
	return func(userRoles []Role) bool {
		for _, ur := range userRoles {
			for _, ar := range allowedRoles {
				if ur == ar {
					return true
				}
			}
		}
		return false
	}
}

// Deny creates an AuthFunc that allows all roles except specified ones
func Deny(deniedRoles ...Role) AuthFunc {
	return func(userRoles []Role) bool {
		for _, ur := range userRoles {
			for _, dr := range deniedRoles {
				if ur == dr {
					return false
				}
			}
		}
		return true
	}
}

// AllowAll creates an AuthFunc that allows all roles
func AllowAll() AuthFunc {
	return func([]Role) bool {
		return true
	}
}

// DenyAll creates an AuthFunc that denies all roles
func DenyAll() AuthFunc {
	return func([]Role) bool {
		return false
	}
}

// RBAC middleware
func RBACMiddleware(rules []Rule) iris.Handler {
	return func(ctx iris.Context) {
		user := getUserFromContext(ctx)
		for _, rule := range rules {
			if ctx.Path() == rule.Path && ctx.Method() == rule.Method {
				if !rule.Auth(user.Roles) {
					ctx.StatusCode(iris.StatusForbidden)
					ctx.JSON(iris.Map{"error": "Access denied"})
					return
				}
				break
			}
		}
		ctx.Next()
	}
}

// getUserFromContext retrieves the user from the Iris context
// In a real application, this would typically involve parsing a JWT token or session data
func getUserFromContext(ctx iris.Context) User {
	// This is a placeholder. In a real app, you'd extract user info from the request
	return User{
		Username: "testuser",
		Roles:    []Role{"roleA"},
	}
}

func main() {
	app := iris.New()

	// Define RBAC rules
	rules := []Rule{
		{Path: "/api/book", Method: "GET", Auth: Allow("roleA", "roleB")},
		{Path: "/api/book", Method: "POST", Auth: Deny("roleB", "roleC")},
		{Path: "/api/person", Method: "GET", Auth: AllowAll()},
		{Path: "/api/person", Method: "DELETE", Auth: DenyAll()},
	}

	// Apply RBAC middleware
	app.Use(RBACMiddleware(rules))

	// Define routes
	app.Get("/api/book", func(ctx iris.Context) {
		ctx.JSON(iris.Map{"message": "Get Book"})
	})

	app.Post("/api/book", func(ctx iris.Context) {
		ctx.JSON(iris.Map{"message": "Create Book"})
	})

	app.Get("/api/person", func(ctx iris.Context) {
		ctx.JSON(iris.Map{"message": "Get Person"})
	})

	app.Delete("/api/person", func(ctx iris.Context) {
		ctx.JSON(iris.Map{"message": "Delete Person"})
	})

	// Start the server
	app.Listen(":8080")
}
```

To run this application, you'll need to install the Iris framework:

```
go get github.com/kataras/iris/v12@latest
```

### Explanation:

1. The overall structure remains similar to the Gin version, but we've adapted it to use Iris-specific types and methods.

2. We use `iris.Handler` instead of `gin.HandlerFunc` for our middleware.

3. In the `RBACMiddleware`, we use `ctx.Path()` and `ctx.Method()` to get the current request path and method.

4. For sending responses, we use Iris's `ctx.JSON()` and `ctx.StatusCode()` methods.

5. In the `main` function, we set up the Iris application, define our RBAC rules, apply the RBAC middleware, and define our routes using Iris's routing methods.

6. We use `iris.Map` instead of `gin.H` for JSON responses.

7. The server is started using `app.Listen(":8080")`.

This implementation provides the same RBAC functionality as the Gin version, but it's now using the Iris framework. The RBAC system remains flexible and extensible. You can easily modify the rules by changing the `rules` slice in the `main` function.

To test the application, you can use curl or a tool like Postman to send requests to `http://localhost:8080/api/book` and `http://localhost:8080/api/person` with different HTTP methods. The responses will vary based on the RBAC rules and the user's roles (which are hardcoded in this example for simplicity).

Remember that this is a basic implementation. In a production environment, you would need to add proper user authentication, error handling, logging, and possibly a more sophisticated way of managing and storing RBAC rules.