# Basic Setup Guide

This repository contains the basic guidelines and project structure for our backend service.

## Table of Contents
- [Project Overview](#project-overview)
- [Project Structure](#project-structure)
- [Development Guidelines](#development-guidelines)
  - [Code Principles](#code-principles)
    - [DRY (Don't Repeat Yourself)](#1-dry-dont-repeat-yourself)
    - [SOLID Principles](#2-solid-principles)
    - [Clean Code Practices](#3-clean-code-practices)
    - [Error Handling](#4-error-handling)
  - [Creating Pull Requests](#creating-pull-requests)
    - [Branch Naming](#1-branch-naming)
    - [Commit Messages](#2-commit-messages)
    - [PR Description](#3-pr-description)
  - [Code Review Guidelines](#code-review-guidelines)
    - [What to Review](#1-what-to-review)
    - [Review Process](#2-review-process)
    - [Response to Reviews](#3-response-to-reviews)
  - [Testing Guidelines](#testing-guidelines)
    - [Unit Tests](#1-unit-tests)
    - [Integration Tests](#2-integration-tests)
    - [Test Coverage](#3-test-coverage)
  - [Documentation](#documentation)
    - [Code Documentation](#1-code-documentation)
    - [Project Documentation](#2-project-documentation)

## Project Overview

This is an example of our project tree if later on we want to add new service 

## Project Structure
### current example is using go and need to adjust if later on we want use other programming language

```
└── auth-service/
    ├── cmd/                    # Application entry points
    │   ├── app/
    │   │   └── main.go        # Main application entry point
    │   └── server/
    │       ├── router.go      # HTTP router configuration
    │       └── health.go      # Health check endpoints
    │
    ├── configs/               # Configuration files
    │   ├── env.json          # Environment variables
    │   └── config.go         # Configuration loader
    │
    ├── database/             # Database connection and setup
    │   └── database.go       # Database initialization
    │
    ├── middlewares/          # HTTP middlewares
    │   └── authorization.go  # Authorization middleware
    │
    ├── docs/                 # Documentation
    │   └── swagger/         # API documentation
    │       ├── swagger.json
    │       ├── swagger.yml
    │       └── docs.go
    │
    ├── internal/            # Internal packages
    │   └── user/           # User domain
    │       ├── mocks/      # Mock implementations for testing
    │       │   ├── UserRepositoryGen.go
    │       │   └── UserUsecaseGen.go
    │       ├── entity/     # Domain entities
    │       │   └── user.go
    │       ├── delivery/   # Delivery layer (HTTP/gRPC handlers)
    │       │   ├── http/
    │       │   │   └── user_delivery.go
    │       │   └── grpc/
    │       │       └── user_delivery.go
    │       ├── usecase/    # Business logic
    │       │   └── user_usecase.go
    │       └── repository/ # Data access layer
    │           ├── pg/     # PostgreSQL implementation
    │           │   └── user_repository.go
    │           └── redis/  # Redis implementation
    │               └── user_repository.go
    │
    ├── pkg/                # Public packages
    │   └── helpers/       # Utility functions
    │       └── utils.go
    │
    ├── go.mod             # Go module definition
    ├── go.sum             # Go module checksums
    ├── Makefile          # Build and development commands
    └── README.md         # This file
```

## Development Guidelines

### Code Principles

1. **DRY (Don't Repeat Yourself)**
   - Avoid code duplication
   - Create reusable components and utilities
   - Extract common functionality into shared packages

   Example:
   ```go
   // Bad: Duplicated validation logic
   func validateUser(user User) error {
       if user.Name == "" {
           return errors.New("name is required")
       }
       if user.Email == "" {
           return errors.New("email is required")
       }
       return nil
   }

   func validateAdmin(admin Admin) error {
       if admin.Name == "" {
           return errors.New("name is required")
       }
       if admin.Email == "" {
           return errors.New("email is required")
       }
       return nil
   }

   // Good: Reusable validation
   type Validatable interface {
       GetName() string
       GetEmail() string
   }

   func validateEntity(entity Validatable) error {
       if entity.GetName() == "" {
           return errors.New("name is required")
       }
       if entity.GetEmail() == "" {
           return errors.New("email is required")
       }
       return nil
   }
   ```

2. **SOLID Principles**
   - Single Responsibility: Each class/function should have one reason to change
   - Open/Closed: Open for extension, closed for modification
   - Liskov Substitution: Subtypes must be substitutable for their base types
   - Interface Segregation: Many specific interfaces are better than one general interface
   - Dependency Inversion: Depend on abstractions, not concretions

   Example:
   ```go
   // Bad: Violates Single Responsibility
   type UserService struct {
       db *sql.DB
   }

   func (s *UserService) ProcessUser(user User) error {
       // Handles validation, database operations, and email sending
       if err := s.validateUser(user); err != nil {
           return err
       }
       if err := s.saveUser(user); err != nil {
           return err
       }
       return s.sendWelcomeEmail(user)
   }

   // Good: Follows Single Responsibility
   type UserValidator struct{}
   type UserRepository struct{ db *sql.DB }
   type EmailService struct{}

   func (v *UserValidator) Validate(user User) error {
       // Only handles validation
       return validateEntity(user)
   }

   func (r *UserRepository) Save(user User) error {
       // Only handles database operations
       return r.db.Save(user)
   }

   func (e *EmailService) SendWelcome(user User) error {
       // Only handles email sending
       return e.sendEmail(user.Email, "Welcome!")
   }
   ```

3. **Clean Code Practices**
   - Use meaningful and descriptive names
   - Keep functions small and focused
   - Write self-documenting code
   - Follow consistent formatting and style
   - Add comments only when necessary to explain "why" not "what"

   Example:
   ```go
   // Bad: Unclear naming and mixed responsibilities
   func proc(u *User) error {
       if u.n == "" {
           return errors.New("bad")
       }
       // ... more code ...
   }

   // Good: Clear naming and focused function
   func validateUserName(user *User) error {
       if user.Name == "" {
           return errors.New("user name is required")
       }
       return nil
   }
   ```

4. **Error Handling**
   - Handle errors explicitly
   - Use appropriate error types
   - Provide meaningful error messages
   - Log errors with sufficient context

   Example:
   ```go
   // Bad: Generic error handling
   func processUser(user User) error {
       if err := db.Save(user); err != nil {
           return err
       }
       return nil
   }

   // Good: Specific error handling
   type UserError struct {
       Code    string
       Message string
       Err     error
   }

   func (e *UserError) Error() string {
       return fmt.Sprintf("%s: %s", e.Code, e.Message)
   }

   func processUser(user User) error {
       if err := db.Save(user); err != nil {
           return &UserError{
               Code:    "USER_SAVE_FAILED",
               Message: "failed to save user to database",
               Err:     err,
           }
       }
       return nil
   }
   ```

### Creating Pull Requests

1. **Branch Naming**
   - Use descriptive branch names
   - Follow pattern: `type/issue-number-short-description`
   - Types: feature/, bugfix/, hotfix/, refactor/

   Examples:
   ```
   feature/123-add-user-authentication
   bugfix/456-fix-login-validation
   hotfix/789-security-patch
   refactor/101-improve-error-handling
   ```

2. **Commit Messages**
   - Use present tense ("Add feature" not "Added feature")
   - Start with a verb
   - Keep first line under 50 characters
   - Add detailed description after a blank line if needed

   Examples:
   ```
   Add user authentication middleware

   - Implement JWT token validation
   - Add role-based access control
   - Update documentation
   ```

3. **PR Description**
   - Clear title describing the change
   - Link related issues
   - Describe the problem and solution
   - List any breaking changes
   - Include testing instructions
   - Add screenshots for UI changes

   Example:
   ```markdown
   # Add User Authentication

   ## Related Issues
   Closes #123

   ## Problem
   Users need to be authenticated before accessing protected resources.

   ## Solution
   - Implement JWT-based authentication
   - Add middleware for token validation
   - Create login and refresh token endpoints

   ## Breaking Changes
   - All protected endpoints now require authentication
   - New environment variables required: JWT_SECRET

   ## Testing
   1. Set up environment variables
   2. Run `make test`
   3. Test login flow with Postman collection

   ## Screenshots
   [Login UI Screenshot]
   ```

### Code Review Guidelines

1. **What to Review**
   - Code correctness and functionality
   - Design and architecture
   - Security considerations
   - Performance implications
   - Test coverage
   - Documentation updates

   Example Review Comments:
   ```markdown
   ## Security
   - Consider adding rate limiting to prevent brute force attacks
   - JWT secret should be rotated periodically

   ## Performance
   - Database query could be optimized with proper indexing
   - Consider caching frequently accessed data

   ## Testing
   - Add test cases for edge cases
   - Increase test coverage for error scenarios
   ```

2. **Review Process**
   - Be constructive and respectful
   - Focus on the code, not the person
   - Explain the "why" behind suggestions
   - Look for potential bugs and edge cases
   - Check for maintainability and readability

   Example Review:
   ```markdown
   Good:
   - "Consider extracting this validation logic into a separate function for reusability"
   - "The error message could be more descriptive to help with debugging"

   Bad:
   - "This code is terrible"
   - "You should know better than this"
   ```

3. **Response to Reviews**
   - Address all comments
   - Explain if you disagree
   - Make requested changes in new commits
   - Keep the conversation professional

   Example Response:
   ```markdown
   I've addressed the review comments:
   - Extracted validation logic into `validateUserInput`
   - Added more descriptive error messages
   - Added test cases for edge cases

   Regarding the caching suggestion, I disagree because:
   - The data changes frequently
   - The overhead of cache invalidation would be higher than the benefit
   ```

### Testing Guidelines

1. **Unit Tests**
   - Test individual components in isolation
   - Use meaningful test names
   - Follow AAA pattern (Arrange, Act, Assert)
   - Mock external dependencies

   Example:
   ```go
   func TestUserService_CreateUser(t *testing.T) {
       // Arrange
       mockRepo := new(MockUserRepository)
       service := NewUserService(mockRepo)
       user := User{Name: "John", Email: "john@example.com"}

       // Act
       err := service.CreateUser(user)

       // Assert
       assert.NoError(t, err)
       mockRepo.AssertExpectations(t)
   }
   ```

2. **Integration Tests**
   - Test component interactions
   - Use test databases
   - Clean up test data
   - Test error scenarios

   Example:
   ```go
   func TestUserFlow(t *testing.T) {
       // Setup
       db := setupTestDB(t)
       defer cleanupTestDB(t, db)
       
       // Test
       user := createTestUser(t, db)
       token := loginUser(t, user)
       profile := getUserProfile(t, token)
       
       // Assert
       assert.Equal(t, user.ID, profile.ID)
   }
   ```

3. **Test Coverage**
   - Aim for high coverage of critical paths
   - Focus on business logic
   - Don't test implementation details
   - Include edge cases

   Example Coverage Report:
   ```
   user_service.go: 95% coverage
   - Missing coverage for concurrent user creation
   - Missing coverage for database connection failure
   ```

### Documentation

1. **Code Documentation**
   - Document public APIs
   - Keep documentation up to date
   - Use clear and concise language
   - Include examples for complex functions

   Example:
   ```go
   // UserService handles user-related operations
   type UserService struct {
       repo UserRepository
   }

   // CreateUser creates a new user in the system.
   // It validates the user data and ensures the email is unique.
   //
   // Example:
   //   user := User{Name: "John", Email: "john@example.com"}
   //   err := userService.CreateUser(user)
   func (s *UserService) CreateUser(user User) error {
       // Implementation
   }
   ```

2. **Project Documentation**
   - Keep README updated
   - Document setup instructions
   - Include architecture diagrams
   - Maintain changelog

   Example Changelog:
   ```markdown
   # Changelog

   ## [1.1.0] - 2024-03-20
   - Add user authentication
   - Implement role-based access control
   - Update documentation

   ## [1.0.0] - 2024-03-15
   - Initial release
   ```

#### the information will be updated accordingly later on
