# API Design Patterns

## API-001: Use Proper HTTP Status Codes

**Impact**: HIGH (Clear API contracts and better client error handling)

Returning the wrong HTTP status code confuses clients and makes debugging difficult. Proper status codes enable clients to handle different scenarios appropriately.

### Incorrect

```csharp
// ❌ BAD: Always returns 200 OK regardless of outcome
[HttpGet("/api/users/{id}")]
public async Task<IActionResult> GetUser(int id)
{
    var user = await context.Users.FindAsync(id);
    if (user == null)
        return Ok(new { error = "User not found" }); // Wrong status!

    return Ok(user);
}

[HttpPost("/api/users")]
public async Task<IActionResult> CreateUser(CreateUserDto dto)
{
    if (!ModelState.IsValid)
        return Ok(new { error = "Invalid input" }); // Wrong status!

    var user = _mapper.Map<User>(dto);
    context.Users.Add(user);
    await context.SaveChangesAsync();

    return Ok(user); // Should be 201 Created
}
```

### Correct

```csharp
// ✅ GOOD: Proper status codes with documentation
[HttpGet("/api/users/{id}")]
[ProducesResponseType(StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<ActionResult<UserDto>> GetUser(int id)
{
    var user = await context.Users.FindAsync(id);
    if (user == null)
        return NotFound(); // 404 Not Found

    return Ok(_mapper.Map<UserDto>(user)); // 200 OK
}

[HttpPost("/api/users")]
[ProducesResponseType(StatusCodes.Status201Created)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
public async Task<ActionResult<UserDto>> CreateUser(CreateUserDto dto)
{
    if (!ModelState.IsValid)
        return BadRequest(ModelState); // 400 Bad Request

    var user = _mapper.Map<User>(dto);
    context.Users.Add(user);
    await context.SaveChangesAsync();

    return CreatedAtAction( // 201 Created with Location header
        nameof(GetUser),
        new { id = user.Id },
        _mapper.Map<UserDto>(user));
}

[HttpPut("/api/users/{id}")]
[ProducesResponseType(StatusCodes.Status204NoContent)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
public async Task<IActionResult> UpdateUser(int id, UpdateUserDto dto)
{
    var user = await context.Users.FindAsync(id);
    if (user == null)
        return NotFound(); // 404 Not Found

    _mapper.Map(dto, user);
    await context.SaveChangesAsync();

    return NoContent(); // 204 No Content
}
```

### Additional Context

Common status codes:
- **200 OK**: Successful GET/PUT requests
- **201 Created**: Successful POST that creates a resource
- **204 No Content**: Successful operation with no response body
- **400 Bad Request**: Invalid input/validation errors
- **401 Unauthorized**: Missing or invalid authentication
- **403 Forbidden**: Authenticated but not authorized
- **404 Not Found**: Resource doesn't exist
- **409 Conflict**: Request conflicts with current state
- **500 Internal Server Error**: Unhandled server error

### Reference

- [REST API Status Codes](https://learn.microsoft.com/en-us/aspnet/core/web-api/)
- [HTTP Status Code Definitions](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
