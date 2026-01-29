# API Contracts: Overdue Todo Items

**Feature**: 001-overdue-todos  
**Date**: 2026-01-29  
**Phase**: Phase 1 - API Contract Design

## Overview

This feature adds **one new endpoint** to support the hybrid time approach (server time on load, client time for updates). No changes to existing todo endpoints.

## New Endpoint: Get Server Time

### `GET /api/server-time`

Returns the current server timestamp for frontend time synchronization.

**Purpose**: Provide authoritative time source for initial overdue status calculation

**Authentication**: None (single-user app)

**Rate Limiting**: None required (called once on app load)

#### Request

```http
GET /api/server-time HTTP/1.1
Host: localhost:3030
Accept: application/json
```

**Parameters**: None

**Headers**: 
- `Accept: application/json` (optional, default response is JSON)

#### Response - Success (200 OK)

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "timestamp": 1738166400000
}
```

**Response Body**:
```typescript
{
  timestamp: number  // Unix timestamp in milliseconds (Date.now())
}
```

**Example**:
```json
{
  "timestamp": 1738166400000
}
```

**Field Descriptions**:
- `timestamp` (number, required): Current server time in milliseconds since Unix epoch (January 1, 1970 00:00:00 UTC). Equivalent to JavaScript `Date.now()`.

#### Response - Server Error (500)

```http
HTTP/1.1 500 Internal Server Error
Content-Type: application/json

{
  "error": "Failed to retrieve server time"
}
```

**Error Response**:
```typescript
{
  error: string  // Error message describing what went wrong
}
```

#### Usage Example

**JavaScript (Frontend)**:
```javascript
async function getServerTime() {
  try {
    const response = await fetch('/api/server-time');
    
    if (!response.ok) {
      throw new Error('Server time fetch failed');
    }
    
    const { timestamp } = await response.json();
    return new Date(timestamp);
  } catch (error) {
    console.error('Failed to fetch server time, using client time:', error);
    return new Date(); // Fallback to client time
  }
}

// Usage in React component
useEffect(() => {
  getServerTime().then(serverDate => {
    setCurrentDate(serverDate);
  });
}, []);
```

**cURL**:
```bash
curl http://localhost:3030/api/server-time
# Response: {"timestamp":1738166400000}
```

#### Backend Implementation Reference

```javascript
// In src/app.js (Express)
app.get('/api/server-time', (req, res) => {
  try {
    res.json({ timestamp: Date.now() });
  } catch (error) {
    console.error('Error getting server time:', error);
    res.status(500).json({ error: 'Failed to retrieve server time' });
  }
});
```

## Existing Endpoints: No Changes

The following existing endpoints remain **unchanged**:

### `GET /api/todos`
- Returns array of all todos
- No modifications to response structure
- Frontend will calculate overdue status from returned due dates

### `POST /api/todos`
- Creates new todo
- No changes to request/response structure
- Due date field already optional

### `PUT /api/todos/:id`
- Updates existing todo
- No changes to request/response structure
- Can update due date without affecting overdue logic

### `DELETE /api/todos/:id`
- Deletes todo
- No changes needed

## Frontend Service Interface

### New Function: `getServerTime()`

```typescript
/**
 * Fetches current server timestamp for time synchronization
 * @returns Promise<Date> Server's current date/time
 * @throws Error if fetch fails (should be caught and fallback to client time)
 */
async function getServerTime(): Promise<Date> {
  const response = await fetch('/api/server-time');
  
  if (!response.ok) {
    throw new Error(`Server time fetch failed: ${response.status}`);
  }
  
  const { timestamp } = await response.json();
  return new Date(timestamp);
}
```

**Location**: `packages/frontend/src/services/todoService.js`

**Error Handling**: Caller should catch and fallback to client time

**Caching**: Frontend should cache result and only fetch once on app initialization

## Data Flow Diagram

```
┌──────────┐                           ┌──────────┐
│          │  GET /api/server-time     │          │
│ Frontend │──────────────────────────►│ Backend  │
│          │                           │          │
│          │  { timestamp: 1738... }   │          │
│          │◄──────────────────────────│          │
└──────────┘                           └──────────┘
     │
     │ Cache timestamp
     │
     ▼
  ┌─────────────┐
  │ Calculate   │
  │ overdue     │
  │ status for  │
  │ each todo   │
  └─────────────┘
```

## Testing Contract

### Backend Tests

```javascript
describe('GET /api/server-time', () => {
  test('returns current timestamp', async () => {
    const response = await request(app).get('/api/server-time');
    
    expect(response.status).toBe(200);
    expect(response.body).toHaveProperty('timestamp');
    expect(typeof response.body.timestamp).toBe('number');
    expect(response.body.timestamp).toBeGreaterThan(0);
  });
  
  test('timestamp is within reasonable range', async () => {
    const before = Date.now();
    const response = await request(app).get('/api/server-time');
    const after = Date.now();
    
    expect(response.body.timestamp).toBeGreaterThanOrEqual(before);
    expect(response.body.timestamp).toBeLessThanOrEqual(after);
  });
});
```

### Frontend Tests

```javascript
describe('getServerTime', () => {
  test('fetches and returns server time as Date', async () => {
    global.fetch = jest.fn(() =>
      Promise.resolve({
        ok: true,
        json: async () => ({ timestamp: 1738166400000 })
      })
    );
    
    const date = await getServerTime();
    
    expect(date).toBeInstanceOf(Date);
    expect(date.getTime()).toBe(1738166400000);
  });
  
  test('throws error on fetch failure', async () => {
    global.fetch = jest.fn(() =>
      Promise.resolve({ ok: false, status: 500 })
    );
    
    await expect(getServerTime()).rejects.toThrow('Server time fetch failed');
  });
});
```

## Versioning & Compatibility

**API Version**: No versioning needed (new endpoint, backward compatible)

**Breaking Changes**: None - existing endpoints unchanged

**Deprecation**: N/A

## Performance Characteristics

**Response Time**: <5ms (trivial operation, just returns timestamp)

**Payload Size**: ~25 bytes JSON

**Caching**: 
- **Browser**: No caching headers needed (called once per session)
- **CDN**: Not applicable (dynamic endpoint)
- **Application**: Frontend caches in memory for session duration

## Security Considerations

**Authentication**: None required (single-user app, time is not sensitive)

**Authorization**: N/A

**Data Validation**: None needed (no user input)

**Rate Limiting**: Not required (low-frequency endpoint)

**CORS**: Already handled by existing CORS configuration

## Summary

✅ **One new endpoint**: `GET /api/server-time`  
✅ **Simple contract**: Single number response  
✅ **No breaking changes**: Existing todo endpoints unchanged  
✅ **Error handling**: Graceful fallback to client time  
✅ **Testable**: Clear contract for unit/integration tests  
✅ **Performant**: <5ms response time, minimal payload

**Next**: Proceed to quickstart guide
