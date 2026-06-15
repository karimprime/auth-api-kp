# auth-api-kp

[![npm version](https://img.shields.io/npm/v/auth-api-kp.svg)](https://www.npmjs.com/package/auth-api-kp)
[![Angular](https://img.shields.io/badge/Angular-14%2B-red.svg)](https://angular.dev)

A **plug-and-play Angular authentication library** that wraps common auth REST API endpoints (sign-up, sign-in, forgot/reset password, profile management, etc.) into a single injectable service with **full TypeScript typing**.

---

## Table of Contents

- [Compatibility](#compatibility)
- [Installation](#installation)
- [Setup](#setup)
  - [Angular 15+ (Standalone)](#angular-15-standalone)
  - [Angular 12–14 (NgModule)](#angular-1214-ngmodule)
- [Configuration](#configuration)
  - [Custom Endpoints](#custom-endpoints)
  - [Default Endpoints](#default-endpoints)
- [Usage](#usage)
  - [Inject the Service](#inject-the-service)
  - [Login](#login)
  - [Register](#register)
  - [Forgot Password Flow](#forgot-password-flow)
  - [Change Password](#change-password)
  - [Profile Management](#profile-management)
  - [Upload Photo](#upload-photo)
  - [Logout](#logout)
  - [Delete Account](#delete-account)
  - [Update Role](#update-role)
- [Reactive User State](#reactive-user-state)
- [Response Adaptor](#response-adaptor)
- [Error Handling](#error-handling)
- [API Reference](#api-reference)
  - [Interfaces](#interfaces)
  - [Enums](#enums)
- [License](#license)

---

## Compatibility

| Angular Version | Support |
| --------------- | ------- |
| 21.x            | ✅      |
| 20.x            | ✅      |
| 19.x            | ✅      |
| 18.x            | ✅      |
| 17.x            | ✅      |
| 16.x            | ✅      |
| 15.x            | ✅      |
| 14.x            | ✅      |

**Peer Dependencies:**

- `@angular/common` >= 14.0.0
- `@angular/core` >= 14.0.0
- `rxjs` >= 7.0.0

---

## Installation

```bash
npm install auth-api-kp
```

---

## Setup

### Angular 15+ (Standalone)

In your `app.config.ts`:

```typescript
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient } from '@angular/common/http';
import { provideAuthApiConfig } from 'auth-api-kp';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(),
    provideAuthApiConfig({
      baseUrl: 'https://your-api.com',
      apiVersion: 'v1',
      endpoints: {
        auth: {
          login: 'auth/signin',
          register: 'auth/signup',
          logout: 'auth/logout',
          forgotPassword: 'auth/forgotPassword',
          verifyResetCode: 'auth/verifyResetCode',
          resetPassword: 'auth/resetPassword',
          profileData: 'auth/profile-data',
          editProfile: 'auth/editProfile',
          changePassword: 'auth/change-password',
          deleteMe: 'auth/deleteMe',
          uploadPhoto: 'auth/upload-photo',
        },
      },
    }),
  ],
};
```

### Angular 12–14 (NgModule)

In your `app.module.ts`:

```typescript
import { NgModule } from '@angular/core';
import { HttpClientModule } from '@angular/common/http';
import { provideAuthApiConfig } from 'auth-api-kp';

@NgModule({
  imports: [HttpClientModule],
  providers: [
    provideAuthApiConfig({
      baseUrl: 'https://your-api.com',
      apiVersion: 'v1',
      endpoints: {
        auth: {
          login: 'auth/signin',
          register: 'auth/signup',
          logout: 'auth/logout',
          // ... add other endpoints as needed
        },
      },
    }),
  ],
})
export class AppModule {}
```

---

## Configuration

### Custom Endpoints

You can customize every endpoint path to match your backend API:

```typescript
provideAuthApiConfig({
  baseUrl: 'https://api.myapp.com',
  apiVersion: 'v2', // Optional — omit if your API has no version prefix
  endpoints: {
    auth: {
      login: 'users/login', // → https://api.myapp.com/v2/users/login
      register: 'users/register', // → https://api.myapp.com/v2/users/register
      logout: 'users/logout',
      // Only provide the endpoints your backend supports
    },
  },
});
```

### Default Endpoints

If you don't customize, the library ships with these defaults via `DEFAULT_API_CONFIG`:

```typescript
import { DEFAULT_API_CONFIG } from 'auth-api-kp';

// Default endpoint paths:
// login:           'auth/signin'
// register:        'auth/signup'
// logout:          'auth/logout'
// forgotPassword:  'auth/forgotPassword'
// verifyResetCode: 'auth/verifyResetCode'
// resetPassword:   'auth/resetPassword'
// profileData:     'auth/profile-data'
// editProfile:     'auth/editProfile'
// changePassword:  'auth/change-password'
// deleteMe:        'auth/deleteMe'
// uploadPhoto:     'auth/upload-photo'
```

You can use it as a base and override specific values:

```typescript
provideAuthApiConfig({
  ...DEFAULT_API_CONFIG,
  baseUrl: 'https://api.myapp.com',
});
```

---

## Usage

### Inject the Service

```typescript
import { Component, inject } from '@angular/core';
import { AuthApiKpService } from 'auth-api-kp';

@Component({ ... })
export class LoginComponent {
  private readonly authService = inject(AuthApiKpService);
}
```

### Login

```typescript
this.authService.login({ email: 'user@example.com', password: '123456' }).subscribe({
  next: (res) => {
    if ('token' in res) {
      console.log('Logged in!', res.token);
      console.log('User:', res.user);
    } else {
      console.error('Error:', res.error);
    }
  },
});
```

### Register

```typescript
this.authService
  .register({
    firstName: 'Karim',
    lastName: 'Ashraf',
    email: 'karim@example.com',
    password: 'securePass123',
    rePassword: 'securePass123',
    gender: 'male',
    phone: '01012345678',
  })
  .subscribe({
    next: (res) => {
      if ('token' in res) {
        console.log('Registered!', res.user);
      }
    },
  });
```

### Forgot Password Flow

**Step 1 — Request reset code:**

```typescript
this.authService.forgetPassword({ email: 'user@example.com' }).subscribe({
  next: (res) => {
    if ('message' in res && !('error' in res)) {
      console.log(res.message); // "Reset code sent to your email"
    }
  },
});
```

**Step 2 — Verify the code:**

```typescript
this.authService.verifyCode({ resetCode: '123456' }).subscribe({
  next: (res) => {
    if ('status' in res) {
      console.log(res.status); // "Success"
    }
  },
});
```

**Step 3 — Set new password:**

```typescript
this.authService.resetPassword({ email: 'user@example.com', newPassword: 'newPass123' }).subscribe({
  next: (res) => {
    if ('message' in res && !('error' in res)) {
      console.log('Password reset successfully');
    }
  },
});
```

### Change Password

```typescript
this.authService.changePassword({ password: 'oldPass', newPassword: 'newPass' }).subscribe({
  next: (res) => {
    if ('token' in res) {
      console.log('Password changed, new token:', res.token);
    }
  },
});
```

### Profile Management

**Get profile:**

```typescript
this.authService.getProfileData().subscribe({
  next: (res) => {
    if ('user' in res) {
      console.log('Profile:', res.user);
    }
  },
});

// Force refresh (bypass cache):
this.authService.getProfileData(true).subscribe(/* ... */);
```

**Edit profile:**

```typescript
this.authService
  .editProfile({ firstName: 'Karim', lastName: 'Prime', phone: '01098765432' })
  .subscribe({
    next: (res) => {
      if ('user' in res) {
        console.log('Updated profile:', res.user);
      }
    },
  });
```

### Upload Photo

```typescript
onFileSelected(event: Event): void {
  const file = (event.target as HTMLInputElement).files?.[0];
  if (file) {
    this.authService.uploadPhoto({ photo: file }).subscribe({
      next: (res) => {
        if ('photoUrl' in res) {
          console.log('Photo URL:', res.photoUrl);
        }
      },
    });
  }
}
```

### Logout

```typescript
this.authService.logout().subscribe({
  next: () => console.log('Logged out'),
});
```

### Delete Account

```typescript
this.authService.deleteMe().subscribe({
  next: (res) => {
    if ('message' in res && !('error' in res)) {
      console.log('Account deleted');
    }
  },
});
```

### Update Role

```typescript
this.authService.updateRole({ role: 'admin' }).subscribe({
  next: (res) => {
    if ('user' in res) {
      console.log('Role updated:', res.user.role);
    }
  },
});
```

---

## Reactive User State

The service exposes a `userData$` BehaviorSubject that automatically updates when the user logs in, registers, fetches profile data, edits their profile, or logs out.

```typescript
import { Component, inject } from '@angular/core';
import { AuthApiKpService } from 'auth-api-kp';

@Component({
  template: `
    @if (authService.userData$ | async; as user) {
      <p>Welcome, {{ user.firstName }}!</p>
    } @else {
      <p>Not logged in</p>
    }
  `,
})
export class NavbarComponent {
  readonly authService = inject(AuthApiKpService);
}
```

---

## Response Adaptor

The library includes `AuthAPIAdaptorService` — an adaptor that transforms full `AuthResponse` objects into a simplified format:

```typescript
import { inject } from '@angular/core';
import { AuthAPIAdaptorService } from 'auth-api-kp';

const adaptor = inject(AuthAPIAdaptorService);

// Transform a full auth response:
const adapted = adaptor.adapt(authResponse);
// Result: { message, token, userEmail, userRole }
```

---

## Error Handling

All methods return `Observable<SuccessType | ErrorResponse>`. Check the response shape to handle errors:

```typescript
this.authService.login({ email, password }).subscribe({
  next: (res) => {
    if ('error' in res) {
      // ErrorResponse — show error message
      console.error(res.error);
    } else {
      // Success response
      console.log('Token:', res.token);
    }
  },
});
```

The `ErrorResponse` interface:

```typescript
interface ErrorResponse {
  error: string;
}
```

---

## API Reference

### Interfaces

| Interface                 | Description                   |
| ------------------------- | ----------------------------- |
| `ApiConfig`               | Configuration for the library |
| `SignInRequest`           | Login credentials             |
| `SignInResponse`          | Login response with token     |
| `SignUpRequest`           | Registration data             |
| `SignUpResponse`          | Registration response         |
| `ForgotPasswordRequest`   | Forgot password email         |
| `ForgotPasswordResponse`  | Forgot password response      |
| `VerifyResetCodeRequest`  | Reset code verification       |
| `VerifyResetCodeResponse` | Verification result           |
| `ResetPasswordRequest`    | New password data             |
| `ResetPasswordResponse`   | Reset result                  |
| `ChangePasswordRequest`   | Password change data          |
| `ChangePasswordResponse`  | Password change result        |
| `ProfileDataResponse`     | User profile data             |
| `EditProfileRequest`      | Profile update fields         |
| `EditProfileResponse`     | Updated profile response      |
| `UploadPhotoRequest`      | Photo file for upload         |
| `UploadPhotoResponse`     | Upload result with URL        |
| `DeleteMeResponse`        | Account deletion result       |
| `LogoutResponse`          | Logout result                 |
| `UpdateRoleRequest`       | Role update data              |
| `UpdateRoleResponse`      | Role update result            |
| `User`                    | Full user data model          |
| `ErrorResponse`           | Error object                  |
| `Adaptor<T, R>`           | Generic data adaptor          |

### Enums

**`AuthEndPoint`** — Default endpoint paths:

| Key               | Value                  |
| ----------------- | ---------------------- |
| `LOGIN`           | `auth/signin`          |
| `REGISTER`        | `auth/signup`          |
| `LOGOUT`          | `auth/logout`          |
| `FORGOT_PASSWORD` | `auth/forgotPassword`  |
| `VERIFY_CODE`     | `auth/verifyResetCode` |
| `RESET_PASSWORD`  | `auth/resetPassword`   |
| `PROFILE_DATA`    | `auth/profile-data`    |
| `EDIT_PROFILE`    | `auth/editProfile`     |
| `DELETE_ACCOUNT`  | `auth/deleteMe`        |
| `CHANGE_PASSWORD` | `auth/change-password` |
| `UPLOAD_PHOTO`    | `auth/upload-photo`    |

---

## License

MIT
