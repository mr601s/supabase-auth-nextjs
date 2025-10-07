# Supabase Authentication - Next.js Setup

This repository contains a complete Supabase authentication setup for Next.js with email/password login and signup functionality.

## 📁 Project Structure

```
├── lib/
│   └── supabaseClient.ts      # Supabase client initialization
├── app/
│   ├── login/
│   │   └── page.tsx           # Login page with authentication form
│   └── signup/
│       └── page.tsx           # Signup page with user registration
└── .env.local                 # Environment variables (needs configuration)
```

## 🚀 Setup Instructions

### 1. Install Dependencies

First, install the required npm packages:

```bash
npm install @supabase/supabase-js
# or
yarn add @supabase/supabase-js
# or
pnpm add @supabase/supabase-js
```

### 2. Configure Supabase Environment Variables

Update the `.env.local` file with your actual Supabase credentials:

1. Go to your Supabase project dashboard: https://app.supabase.com
2. Navigate to: **Settings** → **API**
3. Copy your **Project URL** and **anon/public key**
4. Replace the placeholder values in `.env.local`:

```env
NEXT_PUBLIC_SUPABASE_URL=https://your-actual-project-id.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-actual-anon-key-here
```

### 3. Configure Supabase Authentication

In your Supabase dashboard:

1. Go to **Authentication** → **Providers**
2. Enable **Email** provider
3. Configure email settings:
   - **Confirm email**: Toggle based on your preference (recommended: ON)
   - **Email templates**: Customize as needed

4. Go to **Authentication** → **URL Configuration**
5. Add your site URL and redirect URLs:
   - **Site URL**: `http://localhost:3000` (for development)
   - **Redirect URLs**: Add `http://localhost:3000/**` for local development

### 4. Run the Development Server

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

## 📝 Features Implemented

### ✅ Supabase Client (`lib/supabaseClient.ts`)
- Centralized Supabase client configuration
- Uses environment variables for secure credential management
- Ready to use across the application

### ✅ Login Page (`app/login/page.tsx`)
- Email/password authentication form
- Error handling and loading states
- Redirect to home page after successful login
- Link to signup page for new users

### ✅ Signup Page (`app/signup/page.tsx`)
- User registration with email/password
- Password confirmation validation
- Minimum password length check (6 characters)
- Email confirmation message
- Auto-redirect to login after successful signup
- Duplicate email detection

## 🧪 Testing Your Setup

### Test Signup Flow:
1. Navigate to `/signup`
2. Enter a valid email and password (minimum 6 characters)
3. Submit the form
4. Check your email for confirmation (if email confirmation is enabled)
5. You should be redirected to `/login` after 3 seconds

### Test Login Flow:
1. Navigate to `/login`
2. Enter your registered email and password
3. Submit the form
4. You should be redirected to `/` (home page)

## 🔐 Security Notes

- **Never commit `.env.local`** to version control (it's already in `.gitignore`)
- The `NEXT_PUBLIC_SUPABASE_ANON_KEY` is safe to expose in the browser
- For sensitive operations, implement Row Level Security (RLS) in Supabase
- Always validate user input on both client and server side

## 🎯 Next Steps for User Onboarding

### 1. **Create a Protected Route/Dashboard**
Add an authenticated home page or dashboard:

```typescript
// app/page.tsx or app/dashboard/page.tsx
'use client';

import { useEffect, useState } from 'react';
import { supabase } from '@/lib/supabaseClient';
import { useRouter } from 'next/navigation';

export default function HomePage() {
  const [user, setUser] = useState(null);
  const router = useRouter();

  useEffect(() => {
    const checkUser = async () => {
      const { data: { user } } = await supabase.auth.getUser();
      if (!user) {
        router.push('/login');
      } else {
        setUser(user);
      }
    };
    checkUser();
  }, [router]);

  const handleLogout = async () => {
    await supabase.auth.signOut();
    router.push('/login');
  };

  if (!user) return <div>Loading...</div>;

  return (
    <div>
      <h1>Welcome, {user.email}!</h1>
      <button onClick={handleLogout}>Logout</button>
    </div>
  );
}
```

### 2. **Add Password Reset Functionality**
Create a password reset page:

```typescript
// app/forgot-password/page.tsx
'use client';

import { useState } from 'react';
import { supabase } from '@/lib/supabaseClient';

export default function ForgotPassword() {
  const [email, setEmail] = useState('');
  const [message, setMessage] = useState('');

  const handleReset = async (e: React.FormEvent) => {
    e.preventDefault();
    const { error } = await supabase.auth.resetPasswordForEmail(email, {
      redirectTo: `${window.location.origin}/reset-password`,
    });
    if (error) {
      setMessage('Error: ' + error.message);
    } else {
      setMessage('Check your email for password reset link!');
    }
  };

  return (
    <form onSubmit={handleReset}>
      <input
        type="email"
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Enter your email"
      />
      <button type="submit">Reset Password</button>
      {message && <p>{message}</p>}
    </form>
  );
}
```

### 3. **Implement User Profile Management**
- Create a profile page to view/edit user information
- Store additional user data in Supabase database tables
- Set up Row Level Security (RLS) policies

### 4. **Add Social Authentication (Optional)**
Extend authentication with OAuth providers:
- Google, GitHub, Facebook, etc.
- Configure in Supabase dashboard under Authentication → Providers

### 5. **Set Up Database Tables**
Create tables for your application data:

```sql
-- Example: User profiles table
CREATE TABLE profiles (
  id UUID REFERENCES auth.users PRIMARY KEY,
  username TEXT UNIQUE,
  full_name TEXT,
  avatar_url TEXT,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

-- Create policies
CREATE POLICY "Users can view their own profile"
  ON profiles FOR SELECT
  USING (auth.uid() = id);

CREATE POLICY "Users can update their own profile"
  ON profiles FOR UPDATE
  USING (auth.uid() = id);
```

### 6. **Production Deployment Checklist**
- [ ] Update Supabase redirect URLs with production domain
- [ ] Set up environment variables in your hosting platform
- [ ] Configure email templates in Supabase for your brand
- [ ] Test authentication flow in production
- [ ] Set up proper error logging and monitoring
- [ ] Implement rate limiting for auth endpoints

## 📚 Additional Resources

- [Supabase Documentation](https://supabase.com/docs)
- [Supabase Auth Documentation](https://supabase.com/docs/guides/auth)
- [Next.js Documentation](https://nextjs.org/docs)
- [Row Level Security Guide](https://supabase.com/docs/guides/auth/row-level-security)

## 🐛 Troubleshooting

**Issue**: "Invalid API Key" error
- Verify your `.env.local` file has the correct credentials
- Restart your development server after updating environment variables

**Issue**: Email confirmation not working
- Check your spam folder
- Verify email settings in Supabase dashboard
- Ensure SMTP is configured (or use Supabase's default email service)

**Issue**: Redirect not working after login
- Check that `useRouter` is imported from `next/navigation` (not `next/router`)
- Verify the route you're redirecting to exists

## 📄 License

MIT License - feel free to use this setup in your projects!

---

**Happy coding!** 🚀
