# Waste Collection MVP - Database and API Design

## 1. Introduction

The ezzday MVP is a route planner application for scheduling waste material collection. It facilitates the planning of routes and allocation of resources (trucks, drivers, loaders) for daily operations.

This document outlines the current database structure and API design. It covers both implemented features and future work. Recommendations and feedback on existing implementations are welcome.

Key focus areas include:

- Implementation using Go (Pagoda), Postgres, and Ent ORM
- Existing data model and API endpoints
- Implemented features and future work

## 2. Data Model

The ezzday MVP utilizes a Postgres database with the following key entities, implemented using the Ent ORM:

### 2.1 Core Entities

- **MvpRoute**: Represents predefined collection routes.
  - Fields: name (unique), day_of_week
- **MvpStaff**: Represents drivers and loaders.
  - Fields: name, role (DRIVER or LOADER), email (optional), phone (optional), last_name, birthday (optional)
  - Relationships: driven_routes, loaded_routes (to MvpPlannedRoute)
- **MvpTruck**: Represents collection vehicles.
  - Fields: name
  - Relationships: planned_routes (to MvpPlannedRoute)
- **MvpMaterial**: Represents types of waste materials.
  - Fields: name (unique)
  - Relationships: planned_routes (to MvpPlannedRoute)
- **MvpPlannedRoute**: Represents scheduled collection routes.
  - Fields: date, route_name, status (planned or edited)
  - Relationships: trucks, drivers, loaders, materials

### 2.2 Junction Entities

To manage many-to-many relationships, the following junction entities are implemented:

- MvpPlannedRouteDrivers
- MvpPlannedRouteLoaders
- MvpPlannedRouteMaterials
- MvpPlannedRouteTrucks

These entities facilitate the connections between planned routes and their associated resources.

## 3. API Design

The ezzday MVP exposes the following RESTful API endpoints to support route planning and resource management:

### 3.1 Calendar and Route Planning

#### GET /api/calendar/current-month

- Purpose: Retrieve calendar data for the current month, including planning status for each day.
- Response: JSON object with month, year, and an array of days with their planning status.

#### GET /api/routes-by-day

- Purpose: Fetch routes for a specific date.
- Query Parameters: date (YYYY-MM-DD)
- Response: Array of route objects with ID, name, day of week, and planning status.

### 3.2 Resource Availability

#### GET /api/available/trucks
#### GET /api/available/drivers
#### GET /api/available/loaders

- Purpose: Retrieve available resources (trucks, drivers, or loaders) for a given date.
- Query Parameters: date (YYYY-MM-DD)
- Response: Array of available resource objects.

#### GET /api/available/materials

- Purpose: Fetch all available materials.
- Response: Array of material objects.

### 3.3 Route Scheduling

#### POST /api/route-schedule

- Purpose: Create or update a route schedule.
- Request Body: JSON object with date, route_name, truck_ids, driver_ids, loader_ids, and material_ids.
- Response: Created or updated planned route object.

#### GET /api/route-schedule

- Purpose: Retrieve details of a specific route schedule.
- Query Parameters: date (YYYY-MM-DD), route_name
- Response: Detailed planned route object including assigned resources.

### 3.4 Planned Routes Overview

#### GET /api/planned-routes

- Purpose: Fetch a list of planned routes with pagination and filtering options.
- Query Parameters:
  - page, limit (for pagination)
  - week (ISO week date format), date (YYYY-MM-DD)
  - truck, driver, loader, material (IDs for filtering)
  - route (route name for filtering)
- Response: JSON object with an array of planned routes and pagination details.

### 3.5 Error Handling

All endpoints follow a consistent error handling approach:

- 400 Bad Request: For invalid input parameters
- 404 Not Found: When requested resources don't exist
- 500 Internal Server Error: For server-side issues

Responses include appropriate error messages to facilitate client-side handling and user feedback.

### 3.6 Authentication and Authorization

Note: Authentication and authorization mechanisms are being implemented by another team member (Alauddin). These APIs will be secured appropriately once the auth system is in place.

## 4. Implementation Details

The ezzday MVP backend is implemented using the Pagoda full-stack development kit, leveraging Go for server-side logic, Postgres for data storage, and Ent as the ORM for database operations.

### 4.1 Pagoda Framework

- Utilized as the foundation for the full-stack application.
- Provides a structured approach for routing, middleware, and service container management.
- Echo framework is used for HTTP routing and handling.

### 4.2 Database Operations with Ent ORM

- Ent ORM is used for database schema definition, query building, and data manipulation.
- Entity schemas (MvpRoute, MvpStaff, MvpTruck, etc.) are defined in separate files under the ent/schema/ directory.
- Ent generates type-safe Go code for database operations, ensuring compile-time checks and reducing runtime errors.
- Complex queries, like fetching available resources, are constructed using Ent's fluent API.

### 4.3 Handler Structure

The main handler for the route planner API is implemented in mvprouteplanner_api_handler.go:

- MvpRoutePlannerAPI struct encapsulates the handler methods and dependencies.
- Init method initializes the handler with the service container.
- Routes method defines the API endpoints and maps them to handler methods.
- Individual handler methods (e.g., GetCurrentMonthData, GetRoutesByDay) implement the business logic for each endpoint.

### 4.4 Request Processing

- Echo's context (echo.Context) is used for accessing request data and sending responses.
- Request parameters and query strings are parsed using Echo's built-in methods.
- JSON responses are sent using c.JSON() method.

### 4.5 Error Handling

- HTTP-specific errors are created using echo.NewHTTPError().
- Database errors are caught and translated into appropriate HTTP responses.

### 4.6 Transaction Management

- Database transactions are used for operations that modify multiple records (e.g., creating/updating route schedules).
- Transactions ensure data consistency and allow for rollback in case of errors.

### 4.7 Date Handling

- Go's time.Time type is used for handling dates.
- Date parsing from string inputs is done using time.Parse() with appropriate format strings.

### 4.8 Filtering and Pagination

- The GetPlannedRoutes endpoint demonstrates implementation of filtering and pagination.
- Query parameters are used to specify filters and pagination options.
- Ent's query builder is used to construct dynamic queries based on these parameters

## 5. Future Tasks

The following tasks are planned for the upcoming development. Though not clear on implementation approach yet, but after development, implementations will be shared for review.

### 5.1 Staff Scheduling and Communication using Email Notification System

- Develop API endpoints for emailing staff their biweekly schedules.
- Implement email template generation with personalized schedule information.
- Ensure compliance with email delivery best practices and regulations.

### 5.2 Advanced Schedule Editing

- Create functionality to re-assign constrained resources, such as allowing drivers to work on multiple routes for the same day.
- Implement conflict detection and resolution mechanisms for resource allocation.

### 5.3 Enhanced Planned Routes Endpoint

- Test and expand the /api/planned-routes endpoint with additional filtering options to get results filtered by week, date, driver, loader, material and route name.

### 5.4 Resource Management APIs

- Develop CRUD operations for trucks, staff, and materials.
- Ensure proper validation and error handling for these operations.

### 5.5 System Integration and Resource Management

Discussion on Centralized Resource Management:
- Investigate and plan for the integration of a centralized resource system.
- Explore options for sharing resources (e.g., materials, trucks) between ezzday and ezzton products.
- Design API interfaces for fetching shared resources from a central repository.

### 5.6 Authentication and Authorization

- Enhance the authentication mechanism for API access.
- Implement fine-grained authorization controls for different user roles.
- Implement encryption for data in transit and at rest where necessary.

## 6. API collection (Postman)

APIs can be reviewed from basecamp project eZzDay. It's in "Docs & Files"
