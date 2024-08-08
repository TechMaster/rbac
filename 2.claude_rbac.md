Certainly! I'll create a more structured Go web application that uses RBAC for authorization based on URL paths, HTTP methods, and roles. We'll use the `gin` framework for routing and middleware, and implement the authorization logic as specified.

Here's the complete code:

```go
package main

import (
	"net/http"

	"github.com/gin-gonic/gin"
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
func RBACMiddleware(rules []Rule) gin.HandlerFunc {
	return func(c *gin.Context) {
		user := getUserFromContext(c)
		for _, rule := range rules {
			if c.FullPath() == rule.Path && c.Request.Method == rule.Method {
				if !rule.Auth(user.Roles) {
					c.JSON(http.StatusForbidden, gin.H{"error": "Access denied"})
					c.Abort()
					return
				}
				break
			}
		}
		c.Next()
	}
}

// getUserFromContext retrieves the user from the Gin context
// In a real application, this would typically involve parsing a JWT token or session data
func getUserFromContext(c *gin.Context) User {
	// This is a placeholder. In a real app, you'd extract user info from the request
	return User{
		Username: "testuser",
		Roles:    []Role{"roleA"},
	}
}

func main() {
	r := gin.Default()

	// Define RBAC rules
	rules := []Rule{
		{Path: "/api/book", Method: http.MethodGet, Auth: Allow("roleA", "roleB")},
		{Path: "/api/book", Method: http.MethodPost, Auth: Deny("roleB", "roleC")},
		{Path: "/api/person", Method: http.MethodGet, Auth: AllowAll()},
		{Path: "/api/person", Method: http.MethodDelete, Auth: DenyAll()},
	}

	// Apply RBAC middleware
	r.Use(RBACMiddleware(rules))

	// Define routes
	r.GET("/api/book", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"message": "Get Book"})
	})

	r.POST("/api/book", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"message": "Create Book"})
	})

	r.GET("/api/person", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"message": "Get Person"})
	})

	r.DELETE("/api/person", func(c *gin.Context) {
		c.JSON(http.StatusOK, gin.H{"message": "Delete Person"})
	})

	r.Run(":8080")
}
```

To run this application, you'll need to install the Gin framework:

```
go get -u github.com/gin-gonic/gin
```

### Explanation:

1. We define `Role`, `Rule`, `AuthFunc`, and `User` types to represent our RBAC structure.

2. We implement `Allow`, `Deny`, `AllowAll`, and `DenyAll` functions that return `AuthFunc` as specified in the requirements.

3. The `RBACMiddleware` function checks each request against the defined rules and allows or denies access accordingly.

4. In the `main` function, we set up the Gin router, define our RBAC rules, apply the RBAC middleware, and define our routes.

5. The `getUserFromContext` function is a placeholder for user authentication. In a real application, you would implement proper user authentication, possibly using JWT tokens.

This implementation provides a flexible and extensible RBAC system. You can easily add new rules or modify existing ones by changing the `rules` slice in the `main` function.

To test the application, you can use curl or a tool like Postman to send requests to `http://localhost:8080/api/book` and `http://localhost:8080/api/person` with different HTTP methods. The responses will vary based on the RBAC rules and the user's roles (which are hardcoded in this example for simplicity).

Remember that this is a basic implementation. In a production environment, you would need to add proper user authentication, error handling, logging, and possibly a more sophisticated way of managing and storing RBAC rules.