---
title: 'Sign in with Google'
description: 'Sign in with Google'
pubDate: 'Feb 03 2025'
heroImage: '/google-placeholder.jpg'
---

## How I implemented Google Sign-In for my flight tracking app

When I started thinking about user authentication, I knew I wanted it to be simple and fast. Users shouldn't have to go through complicated signup forms if they don’t want to. So, I decided to implement Google Sign-In as one of the main authentication options in the app.

There are tons of reasons why Google Sign-In is such a great choice:

**Many people already have a Google account, so they don't need to create a new username and password.**

**It feels secure since users trust Google's login system.**

**It's fast and lets users start using the app right away.**

### Setting up Supabase for Google OAuth

Since I’m using Supabase for authentication, the process wasn’t too complicated. Supabase supports OAuth for multiple providers, including Google. Here’s how I got started:

- First, I enabled Google OAuth on my Supabase project dashboard.

- I generated a Google Client ID from the Google Cloud Console and linked it to Supabase.

- I added my app’s URL as a redirect for OAuth, so that after login, users are sent back to the app.


### Modifying my app to handle OAuth

The next step was to update the code in my SupabaseService to support OAuth authentication. I created a function to trigger the Google sign-in process.

Here’s what it looks like:

```typescript

async signInWithGoogle() {
  const redirectTo = Capacitor.isNativePlatform()
    ? 'myapp://auth/callback'  // Custom deep link for mobile
    : window.location.origin + '/auth/callback';  // For web browsers

  const { error } = await this.supabase.auth.signInWithOAuth({
    provider: 'google',
    options: {
      redirectTo,
    },
  });

  if (error) {
    console.error('Google sign-in failed:', error.message);
    throw error;
  }

  console.log('Redirecting to Google OAuth...');
}

```
I also made sure to handle cases where the user might abandon the login halfway through or run into errors. If something goes wrong, I show a toast notification with the error message.

### Handling OAuth on mobile devices

OAuth can get tricky when dealing with mobile apps. For web apps, redirecting back to the app is easy since the browser handles it. But for mobile apps, I needed to use a custom URL scheme to ensure the app reopens after authentication.

Here’s what I added to my AndroidManifest file to support deep linking:

```xml
<intent-filter>
  <action android:name="android.intent.action.VIEW" />
  <category android:name="android.intent.category.DEFAULT" />
  <category android:name="android.intent.category.BROWSABLE" />
  <data android:scheme="myapp" android:host="auth" />
</intent-filter>
```

I also made sure to configure GoogleAuth in my Capacitor config file to support both Android and iOS platforms.

```javascript
const config: CapacitorConfig = {
  appId: 'io.ionic.starter',
  appName: 'Flight Tracker App',
  webDir: 'www',
  plugins: {
    GoogleAuth: {
      scopes: ['profile', 'email'],
      serverClientId: 'YOUR_GOOGLE_CLIENT_ID',
      forceCodeForRefreshToken: true,
    },
  },
};
```

### Restoring user sessions

One annoying issue I faced was that after users logged in and closed the app, they had to log in again when reopening it. That’s obviously not a great experience. To fix this, I used Ionic Storage to persist the user session.

Here’s how it works:

After logging in, I save the session data to local storage:

```typescript
this.supabase.auth.onAuthStateChange(async (event, session) => {
  if (session) {
    console.log('Updating session in storage.');
    await this.storage.set('supabase_session', JSON.stringify(session));
  } else {
    console.log('Session cleared.');
    await this.storage.remove('supabase_session');
  }
});
```

When the app starts, I check if there’s a session in storage and restore it:

```typescript
async restoreSession() {
  const storedSession = await this.storage.get('supabase_session');

  if (storedSession) {
    const session: Session = JSON.parse(storedSession);
    const { error } = await this.supabase.auth.setSession(session);
    if (error) {
      console.error('Failed to restore session:', error);
      await this.storage.remove('supabase_session');
    } else {
      console.log('Session restored successfully.');
    }
  } else {
    console.log('No session found in storage.');
  }
}
```
