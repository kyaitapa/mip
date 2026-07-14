# Auth Testing Playbook

## Test Google Auth Flow
1. Navigate to /login
2. Click "Login with Google" button
3. Verify redirect to auth.emergentagent.com
4. After Google login, verify redirect to /dashboard/mip with session_id in URL fragment
5. Verify session_token cookie is set (httpOnly, secure, sameSite=none)
6. Verify /api/auth/me returns user data
7. Reload page - should stay logged in

## Manual DB Session Setup (if needed)
```bash
mongosh --eval "
use('toyota_mip_db');
var userId = 'user_' + Math.random().toString(36).slice(2,14);
var sessionToken = 'sess_' + Date.now();
db.users.insertOne({
  user_id: userId,
  email: 'test.google@example.com',
  name: 'Test Google User',
  picture: 'https://via.placeholder.com/150',
  role: 'partman',
  branch: 'PLAZA TOYOTA KYAI TAPA',
  auth_provider: 'google',
  created_at: new Date()
});
db.user_sessions.insertOne({
  user_id: userId,
  session_token: sessionToken,
  expires_at: new Date(Date.now() + 7*24*60*60*1000),
  created_at: new Date()
});
"
```

## Endpoints
- POST /api/auth/session - Exchange session_id for session_token + user data
- GET /api/auth/me - Get current user (works with JWT or session_token cookie)
- POST /api/auth/logout - Logout (clears both types)

## Test Users
- Existing password users still work (admin/manager/partman @ toyota.com)
- Google users created on first login with role='partman', no branch (admin must assign)
