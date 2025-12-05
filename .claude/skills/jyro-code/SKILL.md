---
name: jyro-code
description: Expert assistance for writing and debugging Jyro scripts with perfect syntax and idioms
---

# Jyro Code Expert Skill

You are an expert Jyro programmer. Your role is to help users write perfect, idiomatic Jyro scripts that solve their business problems efficiently and correctly.

## Language Overview

Jyro is an imperative, dynamically-typed scripting language designed for secure, sandboxed data transformation. Key characteristics:

- **Data-centric**: All input/output happens through the `Data` context object
- **JSON-native**: Works directly with JSON-compatible types (number, string, boolean, object, array, null)
- **Sandboxed**: No file I/O, network access (by default), or system calls
- **Resource-limited**: Enforced limits on execution time, statements, loops, and stack depth
- **Type-safe at runtime**: Dynamic typing with explicit type checking capabilities

## Core Syntax Reference

### 1. The Data Context

**CRITICAL**: The `Data` object is the ONLY way to pass data in and out of scripts.

```jyro
# Data is always available - never declare it
Data.result = "Hello, World!"

# Read from Data (provided by host)
var input = Data.userInput
var count = Data.items.length

# Write to Data (returned to host)
Data.processed = true
Data.timestamp = Now()

# Nested properties - create intermediate objects first!
Data.level1 = {}
Data.level1.level2 = {}
Data.level1.level2.value = "nested"

# Local variables are discarded after execution
var temp = 100  # NOT returned to host
Data.output = temp  # This IS returned
```

**Rules**:
- `Data` is ALWAYS an object, never null
- `Data` cannot be reassigned or shadowed
- Only modifications to `Data` persist after execution
- Local variables (`var`) exist only during script execution

### 2. Variables and Types

**Declaration**:
```jyro
var count = 0
var name = "Alice"
var isActive = true
var result = null
var items = []
var config = {}

# Uninitialized variables default to null
var unset  # null
```

**Types** (5 primitive + 2 structural):
- `number` - IEEE 754 double (42, 3.14, -100)
- `string` - Double-quoted text ("hello", "world")
- `boolean` - true or false
- `null` - absence of value
- `object` - Key-value pairs { "key": "value" }
- `array` - Ordered collections [1, 2, 3]

**Type Checking**:
```jyro
# Using 'is' operator (preferred for conditions)
if Data.value is number then
    Data.doubled = Data.value * 2
end

# Using TypeOf function (returns string)
var typeName = TypeOf(Data.value)  # "number", "string", etc.

# Null checking
if Data.optional is null then
    Data.optional = "default"
end

if IsNull(Data.optional) then
    Data.optional = "default"
end

# Property existence (checks defined AND not null)
if Exists(Data.optionalField) then
    Data.result = Data.optionalField
end
```

**Truthiness** (IMPORTANT for conditions):

Falsy values:
- `false`
- `null`
- `0`
- `""` (empty string)

Truthy values (everything else, including):
- All non-zero numbers
- All non-empty strings
- **ALL arrays** (even empty `[]`)
- **ALL objects** (even empty `{}`)
- `true`

```jyro
# WRONG - empty arrays are truthy!
if Data.items then
    # Executes even if array is empty
end

# RIGHT - check length explicitly
if Data.items is not null and Length(Data.items) > 0 then
    # Only executes if array has elements
end
```

### 3. Operators

**Arithmetic**:
```jyro
var sum = 10 + 5        # 15
var diff = 10 - 5       # 5
var product = 10 * 5    # 50
var quotient = 10 / 5   # 2.0 (always float!)
var remainder = 10 % 3  # 1
var negative = -value   # Unary negation
```

**CRITICAL**: Division by zero throws exception - always validate!
```jyro
if divisor != 0 then
    Data.result = numerator / divisor
else
    Data.error = "Division by zero"
end
```

**String Concatenation**:
```jyro
var msg = "Count: " + 42         # "Count: 42" (auto-converts)
var label = "Status: " + true    # "Status: true"

# Other operators DO NOT auto-convert
var error = "10" - 5  # ERROR: cannot subtract from string
```

**Comparison**:
```jyro
var isEqual = a == b
var notEqual = a != b
var less = a < b
var lessOrEq = a <= b
var greater = a > b
var greaterOrEq = a >= b
```

**Logical** (with short-circuit evaluation):
```jyro
# and - both must be true
if Data.age >= 18 and Data.hasPermission then
    Data.canAccess = true
end

# or - at least one must be true
if Data.isAdmin or Data.isEditor then
    Data.canEdit = true
end

# not - negation
if not Data.isActive then
    Data.status = "inactive"
end

# Short-circuit prevents errors
if Data.user != null and Data.user.age >= 18 then
    # Data.user.age only evaluated if Data.user is not null
end
```

**Ternary Conditional**:
```jyro
var status = Data.isActive ? "active" : "inactive"
var discount = Data.isPremium ? 0.2 : 0.1
var label = count == 1 ? "item" : "items"
```

**Operator Precedence** (high to low):
1. `()` - Parentheses
2. `.`, `[]` - Member access
3. Function calls
4. `not`, `-` (unary)
5. `*`, `/`, `%`
6. `+`, `-`
7. `<`, `<=`, `>`, `>=`, `is`
8. `==`, `!=`
9. `and`
10. `or`
11. `? :`
12. `=`

### 4. Control Flow

**if/else if/else**:
```jyro
if condition then
    # statements
else if otherCondition then
    # statements
else
    # statements
end
```

**switch/case**:
```jyro
switch Data.action
case "create"
    Data.status = "created"
case "update"
    Data.status = "updated"
case "delete"
    Data.status = "deleted"
default
    Data.status = "unknown"
end
```

**while loop**:
```jyro
var count = 0
while count < 10 do
    count = count + 1
    Data.result = count
end
```

**foreach loop** (for arrays):
```jyro
var total = 0
foreach item in Data.items do
    total = total + item.price
end
Data.total = total

# Modifying objects in array
foreach user in Data.users do
    user.processed = true
    user.timestamp = Now()
end

# CRITICAL: Do NOT modify the array being iterated!
# This will cause "Collection was modified" error:
# foreach item in Data.items do
#     Append(Data.items, "new")  # ERROR!
# end

# Instead, use a temporary array:
var additions = []
foreach item in Data.items do
    if item.shouldDuplicate then
        Append(additions, item)
    end
end
foreach addition in additions do
    Append(Data.items, addition)
end
```

**Loop Control**:
```jyro
# break - exit loop
foreach item in items do
    if item.isTarget then
        Data.found = item
        break
    end
end

# continue - skip to next iteration
foreach item in items do
    if item.skip then
        continue
    end
    # Process item
end

# return - exit script immediately
if Data.error then
    return
end
```

### 5. Arrays

**Creation and Access**:
```jyro
var numbers = [1, 2, 3, 4, 5]
var names = ["Alice", "Bob", "Carol"]
var mixed = [1, "text", true, null, {}, []]
var empty = []

# Zero-based indexing
var first = items[0]
var second = items[1]

# Out of bounds returns null
var missing = items[999]  # null

# Length
var count = Length(items)

# First and last elements
var first = First(items)
var last = Last(items)
var firstN = Take(items, 4)  # returns the first four members from `items`
```

**Mutation vs Immutable Functions**:

**Functions that MUTATE the original array**:
```jyro
var items = [1, 2, 3]

Append(items, 4)           # items = [1, 2, 3, 4]
Insert(items, 1, "x")      # items = [1, "x", 2, 3, 4]
RemoveAt(items, 0)         # items = ["x", 2, 3, 4]
RemoveLast(items)          # items = ["x", 2, 3]
Pop(items)                 # returns 3, items = ["x", 2]
Clear(items)               # items = []
```

**Functions that RETURN NEW arrays**:
```jyro
var numbers = [5, 2, 8, 1, 9]

var sorted = Sort(numbers)              # [1, 2, 5, 8, 9], numbers unchanged
var reversed = Reverse(numbers)         # [9, 1, 8, 2, 5], numbers unchanged

var users = [
    { "name": "Alice", "age": 30 },
    { "name": "Bob", "age": 25 }
]

var filtered = Filter(users, "age", ">=", 30)  # New array
var byAge = SortByField(users, "age", "desc")  # New array

var combined = MergeArrays([1, 2], [3, 4])  # [1, 2, 3, 4]
```

**Searching**:
```jyro
var items = ["apple", "banana", "cherry"]
var index = IndexOf(items, "banana")  # 1
var notFound = IndexOf(items, "grape")  # -1

# IndexOf uses DEEP EQUALITY (value comparison)
var users = [
    { "id": 1, "name": "Alice" },
    { "id": 2, "name": "Bob" }
]
var idx = IndexOf(users, { "id": 2, "name": "Bob" })  # 1 (found!)

# Checking if array contains value
var hasApple = IndexOf(items, "apple") >= 0  # true

# Cryptographically secure random selection from array
var colors = ["red", "blue", "green", "yellow"]
var randomColor = RandomChoice(colors)  # Randomly selects one element

var prizes = [100, 50, 25, 10, 5]
var winner = RandomChoice(prizes)       # Randomly selects one prize
```

**Filtering** (arrays of objects only):
```jyro
var users = [
    { "name": "Alice", "age": 30, "active": true },
    { "name": "Bob", "age": 25, "active": false },
    { "name": "Carol", "age": 35, "active": true }
]

# Filter(array, fieldName, operator, value)
var adults = Filter(users, "age", ">=", 30)
var active = Filter(users, "active", "==", true)

# Supports dot notation for nested properties
var employees = [
    { "name": "Alice", "address": { "city": "Boston" } },
    { "name": "Bob", "address": { "city": "New York" } }
]
var bostonEmp = Filter(employees, "address.city", "==", "Boston")

# Operators: ==, !=, <, <=, >, >=
```

**Counting**:
```jyro
var users = [
    { "name": "Alice", "age": 30 },
    { "name": "Bob", "age": 25 },
    { "name": "Carol", "age": 35 }
]

var total = Length(users)  # 3
var adultCount = CountIf(users, "age", ">=", 30)  # 2
```

**Sorting**:
```jyro
# Primitives
var numbers = [5, 2, 8, 1, 9]
var sorted = Sort(numbers)  # [1, 2, 5, 8, 9]

# Objects by field
var users = [
    { "name": "Carol", "age": 35 },
    { "name": "Alice", "age": 30 },
    { "name": "Bob", "age": 25 }
]
var byName = SortByField(users, "name", "asc")
var byAge = SortByField(users, "age", "desc")

# Reverse order
var reversed = Reverse([1, 2, 3])  # [3, 2, 1]
```

**Grouping**:
```jyro
var employees = [
    { "name": "Alice", "department": "Engineering" },
    { "name": "Bob", "department": "Sales" },
    { "name": "Carol", "department": "Engineering" },
    { "name": "Dave", "department": "Sales" }
]

# Group by field - returns object with arrays
var byDept = GroupBy(employees, "department")
# Result:
# {
#     "Engineering": [{ "name": "Alice", ... }, { "name": "Carol", ... }],
#     "Sales": [{ "name": "Bob", ... }, { "name": "Dave", ... }]
# }

# Access a group
var engineers = byDept["Engineering"]  # array of 2 employees

# Supports nested field paths with dot notation
var people = [
    { "name": "Alice", "address": { "city": "Boston" } },
    { "name": "Bob", "address": { "city": "New York" } },
    { "name": "Carol", "address": { "city": "Boston" } }
]
var byCity = GroupBy(people, "address.city")

# Items with null/missing field values are grouped under "null" key
# Non-object elements in the array are skipped
```

### 6. Objects

**Creation and Access**:
```jyro
var user = {
    "name": "Alice",
    "age": 30,
    "email": "alice@example.com"
}

# Dot notation
var name = user.name
user.age = 31

# Bracket notation
var field = "email"
var email = user[field]
user["status"] = "active"

# Nested objects
var person = {
    "name": "Alice",
    "address": {
        "city": "Boston",
        "state": "MA"
    }
}
var city = person.address.city

# Non-existent properties return null
var missing = user.nonExistent  # null
```

**Building Objects Dynamically**:
```jyro
# Start with empty object
var result = {}

# Add properties
result.status = "success"
result.count = 10
result.items = []

# Build nested - create each level
var config = {}
config.settings = {}
config.settings.theme = "dark"
config.settings.notifications = {}
config.settings.notifications.email = true
```

### 7. Strings

**Literals** (double-quotes only):
```jyro
var text = "Hello, World!"
var empty = ""

# Escape sequences
var withQuote = "She said, \"Hello!\""
var withNewline = "Line 1\nLine 2"
var withTab = "Col1\tCol2"
var withBackslash = "C:\\Users\\Alice"
var unicode = "\u0048\u0065\u006C\u006C\u006F"  # "Hello"
```

**Standard Library Functions**:
```jyro
# Case conversion
var upper = Upper("hello")     # "HELLO"
var lower = Lower("HELLO")     # "hello"

# Trimming whitespace
var trimmed = Trim("  hello  ")  # "hello"

# Searching
var contains = Contains("Hello, World", "World")  # true
var starts = StartsWith("Hello", "Hel")           # true
var ends = EndsWith("Hello", "lo")                # true

# Transformation
var replaced = Replace("Hello, World", "World", "Jyro")  # "Hello, Jyro"
var parts = Split("a,b,c", ",")                          # ["a", "b", "c"]
var joined = Join(["a", "b", "c"], ",")                  # "a,b,c"

# Character access (zero-based)
var text = "Jyro"
var first = text[0]   # "J"
var last = text[3]    # "o"
var oob = text[10]    # null

# Length
var len = Length("Hello")  # 5

# Type conversion
var num = ToNumber("42")       # 42
var decimal = ToNumber("3.14") # 3.14
var invalid = ToNumber("abc")  # 0 (returns 0 for invalid!)
var empty = ToNumber("")       # 0

# Cryptographically secure random string generation
var chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
var randomCode = RandomString(chars, 8)     # Random 8-char string from charset
var alphaNum = RandomString("abcdefghijklmnopqrstuvwxyz0123456789", 16)
var pin = RandomString("0123456789", 6)     # Random 6-digit PIN
```

**CRITICAL**: `ToNumber` returns 0 for invalid input - validate if 0 is a valid value!
```jyro
var text = Data.input
var number = ToNumber(text)
if number == 0 and text != "0" then
    Data.error = "Invalid number"
end
```

### 8. Math Functions

```jyro
# Basic math
var absolute = Abs(-42)           # 42
var rounded = Round(3.14159, 2)   # 3.14
var noDecimals = Round(3.7, 0)    # 4

# Aggregation
var total = Sum(1, 2, 3, 4, 5)    # 15 (variadic!)
var sum2 = Sum(10, 20)            # 30
var single = Sum(100)             # 100

# CRITICAL: Sum IGNORES non-numeric arguments
var partial = Sum(1, "text", 3, null, 5)  # 9 (only 1, 3, 5)

# Min/Max take EXACTLY TWO arguments (NOT variadic)
var minimum = Min(5, 2)     # 2
var maximum = Max(5, 2)     # 5

# For multiple values, nest or loop:
var min1 = Min(5, 2)        # 2
var min2 = Min(min1, 8)     # 2
var min3 = Min(min2, 1)     # 1

# Or use a loop for arrays:
var numbers = [5, 2, 8, 1, 9]
var minimum = numbers[0]
foreach num in numbers do
    if num < minimum then
        minimum = num
    end
end

# Cryptographically secure random integer
var randomDice = RandomInt(1, 6)        # Random int between 1-6 inclusive
var randomId = RandomInt(1000, 9999)    # Random int between 1000-9999 inclusive
var coin = RandomInt(0, 1)              # Random 0 or 1
```

### 9. Date and Time Functions

```jyro
# Current date/time
var now = Now()          # "2025-10-31T14:30:00" (ISO 8601)
var today = Today()      # "2025-10-31"

# Parsing
var parsed = ParseDate("2025-10-31")

# Formatting
var date = Now()
var formatted = FormatDate(date, "yyyy-MM-dd")        # "2025-10-31"
var withTime = FormatDate(date, "yyyy-MM-dd HH:mm")   # "2025-10-31 14:30"

# Format specifiers:
# yyyy - 4-digit year
# MM   - 2-digit month
# dd   - 2-digit day
# HH   - 2-digit hour (24-hour)
# mm   - 2-digit minute
# ss   - 2-digit second

# Date arithmetic
var tomorrow = DateAdd(Today(), "days", 1)
var nextWeek = DateAdd(Today(), "days", 7)
var nextMonth = DateAdd(Today(), "months", 1)
var yesterday = DateAdd(Today(), "days", -1)

# Units: "days", "weeks", "months", "years", "hours", "minutes", "seconds"

# Date difference
var start = ParseDate("2025-01-01")
var end = ParseDate("2025-12-31")
var daysBetween = DateDiff(end, start, "days")  # 364

# Extracting components
var date = Now()
var year = DatePart(date, "year")
var month = DatePart(date, "month")
var day = DatePart(date, "day")
```

### 10. Utility Functions

```jyro
# Deep equality (for objects and arrays)
var obj1 = { "name": "Alice", "age": 30 }
var obj2 = { "name": "Alice", "age": 30 }
var equal = Equal(obj1, obj2)        # true
var notEqual = NotEqual(obj1, obj2)  # false

# Note: == operator uses reference equality for objects
var same = obj1 == obj2  # false (different references)

# Type inspection
var type = TypeOf(Data.value)  # "number", "string", "boolean", etc.

# Null checking
var isNull = IsNull(Data.optional)  # true if null
var exists = Exists(Data.field)  # true if defined and not null

# Object keys - get all property names as an array
var user = { "name": "Alice", "age": 30, "email": "alice@example.com" }
var keys = Keys(user)  # ["name", "age", "email"]

# Useful for iterating over dynamic objects
var lookup = { "a": 1, "b": 2, "c": 3 }
foreach key in Keys(lookup) do
    Log("Info", key + " = " + lookup[key])
end

# Returns empty array for objects with no properties
var emptyKeys = Keys({})  # []

# Base64 encoding
var text = "Hello, World!"
var encoded = Base64Encode(text)     # "SGVsbG8sIFdvcmxkIQ=="
var decoded = Base64Decode(encoded)  # "Hello, World!"

# GUID generation
var id = NewGuid()  # "a1b2c3d4-e5f6-7g8h-9i0j-k1l2m3n4o5p6"

# HTTP requests (requires host opt-in!)
var headers = { "Content-Type": "application/json" }
var body = { "name": "Alice", "email": "alice@example.com" }
var response = InvokeRestMethod("https://api.example.com/users", "POST", headers, body)

# Simple GET
var data = InvokeRestMethod("https://api.example.com/data", "GET", null, null)
```

## Common Patterns and Best Practices

### Pattern: Input Validation
```jyro
# Validate required fields
if IsNull(Data.userId) or Data.userId == "" then
    Data.error = "User ID is required"
    return
end

if Data.age is not number then
    Data.error = "Age must be a number"
    return
end

if Data.age < 0 or Data.age > 150 then
    Data.error = "Invalid age"
    return
end

# Proceed with valid data
Data.valid = true
```

### Pattern: Safe Property Access
```jyro
# Check each level of nested properties
if Data.user != null and Data.user.profile != null then
    Data.userName = Data.user.profile.name
else
    Data.userName = "Guest"
end

# Or use Exists for cleaner code
if Exists(Data.user.profile) and Exists(Data.user.profile.name) then
    Data.userName = Data.user.profile.name
else
    Data.userName = "Guest"
end
```

### Pattern: Array Processing Pipeline
```jyro
# Filter -> Sort -> Extract
var employees = Data.employees

# Step 1: Filter active employees
var active = Filter(employees, "active", "==", true)

# Step 2: Filter by department
var engineering = Filter(active, "department", "==", "Engineering")

# Step 3: Sort by salary
var sorted = SortByField(engineering, "salary", "desc")

# Step 4: Extract top 5
Data.topEarners = []
var count = 0
foreach emp in sorted do
    if count >= 5 then
        break
    end
    Append(Data.topEarners, emp)
    count = count + 1
end
```

### Pattern: Accumulation
```jyro
# Sum values
var total = 0
foreach item in Data.items do
    if item.price is number then
        total = total + item.price
    end
end
Data.total = total

# Build result array
var results = []
foreach item in Data.items do
    if item.active then
        Append(results, item)
    end
end
Data.activeItems = results
```

### Pattern: Transformation
```jyro
# Transform each element
foreach user in Data.users do
    user.fullName = user.firstName + " " + user.lastName
    user.ageGroup = user.age >= 18 ? "adult" : "minor"
    user.processedAt = Now()
end
```

### Pattern: Conditional Aggregation
```jyro
var activeCount = 0
var inactiveCount = 0
var totalRevenue = 0

foreach account in Data.accounts do
    if account.active then
        activeCount = activeCount + 1
        if account.revenue is number then
            totalRevenue = totalRevenue + account.revenue
        end
    else
        inactiveCount = inactiveCount + 1
    end
end

Data.summary = {
    "active": activeCount,
    "inactive": inactiveCount,
    "totalRevenue": totalRevenue
}
```

## Critical Gotchas and Debugging

### 1. Empty Arrays/Objects Are Truthy!
```jyro
# WRONG
if Data.items then
    # Executes even if items is []
end

# RIGHT
if Data.items is not null and Length(Data.items) > 0 then
    # Only executes if items has elements
end
```

### 2. ToNumber Returns 0 for Invalid Input
```jyro
# WRONG - can't distinguish invalid from zero
var num = ToNumber(Data.input)
Data.result = num

# RIGHT - validate before using
var num = ToNumber(Data.input)
if num == 0 and Data.input != "0" then
    Data.error = "Invalid number: " + Data.input
    return
end
Data.result = num
```

### 3. Division by Zero Crashes Script
```jyro
# WRONG - crashes if divisor is 0
Data.average = Data.total / Data.count

# RIGHT - check first
if Data.count != 0 then
    Data.average = Data.total / Data.count
else
    Data.average = 0
end
```

### 4. Cannot Modify Array During Iteration
```jyro
# WRONG - runtime error
foreach item in Data.items do
    Append(Data.items, item)  # ERROR!
end

# RIGHT - use temporary array
var additions = []
foreach item in Data.items do
    if item.shouldDuplicate then
        Append(additions, item)
    end
end
foreach addition in additions do
    Append(Data.items, addition)
end
```

### 5. Nested Properties Need Intermediate Objects
```jyro
# WRONG - error if level1 doesn't exist
Data.level1.level2.value = "x"  # ERROR!

# RIGHT - create each level
Data.level1 = {}
Data.level1.level2 = {}
Data.level1.level2.value = "x"
```

### 6. Mutation vs Immutable Functions
```jyro
# Mutating functions change original
var items = [1, 2, 3]
Append(items, 4)  # items is now [1, 2, 3, 4]

# Immutable functions return new array
var items = [3, 1, 2]
Sort(items)  # items is STILL [3, 1, 2] - Sort returns new array!
var sorted = Sort(items)  # NOW sorted is [1, 2, 3]
```

### 7. IndexOf Uses Deep Equality
```jyro
# This works (deep comparison)
var users = [{ "id": 1 }, { "id": 2 }]
var idx = IndexOf(users, { "id": 2 })  # 1 (found!)

# But this doesn't (reference comparison)
var obj = { "id": 2 }
var same = users[1] == obj  # false (different references)
```

## Debugging Strategies

### 1. Add Intermediate Variables
```jyro
# Hard to debug
Data.result = SortByField(Filter(Data.items, "active", "==", true), "price", "desc")

# Easy to debug
var active = Filter(Data.items, "active", "==", true)
Data.debugActive = active  # Inspect this
var sorted = SortByField(active, "price", "desc")
Data.debugSorted = sorted  # Inspect this
Data.result = sorted
```

### 2. Add Validation Checkpoints
```jyro
# Validate inputs
Data.debug = {
    "inputType": TypeOf(Data.input),
    "inputValue": Data.input,
    "isNull": IsNull(Data.input)
}

if Data.input is not array then
    Data.error = "Expected array, got " + TypeOf(Data.input)
    return
end

Data.debug.arrayLength = Length(Data.input)
```

### 3. Test with Minimal Data
Start with the simplest possible input:
```json
{
    "items": [
        { "id": 1, "price": 10 }
    ]
}
```

Then add complexity:
```json
{
    "items": [
        { "id": 1, "price": 10 },
        { "id": 2, "price": null },
        { "id": 3 }
    ]
}
```

### 4. Use Type Checks Liberally
```jyro
foreach item in Data.items do
    if item is not object then
        Data.warning = "Skipping non-object item"
        continue
    end

    if not Exists(item.price) then
        Data.warning = "Item missing price: " + item.id
        continue
    end

    if item.price is not number then
        Data.warning = "Invalid price type for item: " + item.id
        continue
    end

    # Safe to process
    Data.total = Data.total + item.price
end
```

## Script Composition with CallScript

For complex workflows, break logic into smaller scripts:

```jyro
# Main script orchestrates
var validationScript = @"
    if not Exists(Data.email) or Data.email == "" then
        Data.isValid = false
        Data.error = "Email required"
    else
        Data.isValid = true
    end
"

var processingScript = @"
    Data.processedAt = Now()
    Data.status = "completed"
"

# Validate first
var validated = CallScript(validationScript, Data)

if validated.isValid then
    var processed = CallScript(processingScript, validated)
    Data.result = processed
else
    Data.error = validated.error
end
```

**Limits**:
- Max 5 levels of nested CallScript calls
- Cycle detection prevents infinite recursion
- Resource limits shared across entire call chain

## Performance Considerations

1. **Minimize array passes**: Each function processes the entire array
   ```jyro
   # Two passes
   var filtered = Filter(Data.items, "active", "==", true)
   var sorted = Sort(filtered)

   # Still two passes, but explicit
   var result = Sort(Filter(Data.items, "active", "==", true))
   ```

2. **Filter early**: Reduce data volume before sorting
   ```jyro
   # Better - filter first (smaller array to sort)
   var result = SortByField(Filter(Data.items, "active", "==", true), "price", "desc")

   # Worse - sort first (larger array to sort)
   var result = Filter(SortByField(Data.items, "price", "desc"), "active", "==", true)
   ```

3. **Avoid redundant operations**:
   ```jyro
   # Bad - calls Length every iteration
   var i = 0
   while i < Length(Data.items) do
       i = i + 1
   end

   # Good - call Length once
   var count = Length(Data.items)
   var i = 0
   while i < count do
       i = i + 1
   end
   ```

## When Writing Jyro Scripts

**ALWAYS**:
1. Start with `Data` - all input comes from here, all output goes here
2. Check for null before accessing nested properties
3. Validate divisors before division or modulo
4. Check array length explicitly (empty arrays are truthy!)
5. Use short-circuit evaluation to prevent errors
6. Create intermediate objects for nested property assignment
7. Remember which functions mutate vs return new values

**NEVER**:
1. Declare a variable named `Data`
2. Modify an array while iterating over it
3. Assume `ToNumber` will error on invalid input (it returns 0)
4. Use single quotes for strings (only double quotes work)
5. Rely on empty array/object being falsy (they're truthy!)
6. Use compound assignment operators (+=, -=, etc. - not supported)
7. Forget to capture return values from immutable functions (Sort, Filter, etc.)

## Example: Complete Script Template

```jyro
# 1. Validate inputs
if IsNull(Data.items) or Data.items is not array then
    Data.error = "Invalid input: items must be an array"
    return
end

if Length(Data.items) == 0 then
    Data.result = []
    Data.summary = { "count": 0, "total": 0 }
    return
end

# 2. Initialize output
Data.result = []
Data.errors = []
var total = 0
var count = 0

# 3. Process items
foreach item in Data.items do
    # Validate item structure
    if item is not object then
        Append(Data.errors, "Non-object item skipped")
        continue
    end

    if not Exists(item.price) or item.price is not number then
        Append(Data.errors, "Item missing valid price: " + item.id)
        continue
    end

    if item.price < 0 then
        Append(Data.errors, "Negative price for item: " + item.id)
        continue
    end

    # Process valid item
    total = total + item.price
    count = count + 1

    # Transform and add to results
    var processed = {
        "id": item.id,
        "price": item.price,
        "priceWithTax": Round(item.price * 1.1, 2),
        "processedAt": Now()
    }
    Append(Data.result, processed)
end

# 4. Generate summary
Data.summary = {
    "totalItems": count,
    "totalPrice": Round(total, 2),
    "averagePrice": count > 0 ? Round(total / count, 2) : 0,
    "errorCount": Length(Data.errors)
}

# 5. Sort results if needed
if count > 0 then
    Data.result = SortByField(Data.result, "price", "desc")
end
```

This skill provides you with complete, accurate Jyro syntax and idioms. When helping users, always write syntactically perfect code that follows these patterns and avoids the documented gotchas.
