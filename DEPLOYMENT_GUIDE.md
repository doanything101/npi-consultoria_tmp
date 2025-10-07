# Deployment Guide - NPi Consultoria

## 🚨 Build Error Fix

The build error you encountered was caused by the MongoDB connection check happening at **module import time** instead of **runtime**. This has been fixed in `src/app/lib/mongodb.ts`.

### What was changed:
- Moved the `MONGODB_URI` environment variable check from the module level into the `connectToDatabase()` function
- This allows Next.js to build successfully even when the environment variable isn't available during the build phase
- The variable will still be validated at runtime when the database connection is actually needed

## 📋 Deployment Checklist

### 1. Configure Vercel Environment Variables

You **must** add the following environment variables in your Vercel project settings:

**Path:** Vercel Dashboard → Your Project → Settings → Environment Variables

Add these variables for **Production**, **Preview**, and **Development** environments:

#### Required Variables:

```bash
# Database
MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/database?retryWrites=true&w=majority

# Firebase Admin
FIREBASE_PROJECT_ID=your-project-id
FIREBASE_CLIENT_EMAIL=firebase-adminsdk-xxxxx@your-project-id.iam.gserviceaccount.com
FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\nYour-Private-Key-Here\n-----END PRIVATE KEY-----\n"

# AWS S3
AWS_ACCESS_KEY_ID=your-aws-access-key-id
AWS_SECRET_ACCESS_KEY=your-aws-secret-access-key
AWS_REGION=sa-east-1
AWS_BUCKET_NAME=npi-imoveis

# Site Configuration
NEXT_PUBLIC_SITE_URL=https://www.npiconsultoria.com.br
```

#### Optional Variables:

```bash
# Google Maps API
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=your-google-maps-api-key

# Google Vision API (for image analysis)
GOOGLE_APPLICATION_CREDENTIALS=path/to/service-account-key.json
```

### 2. Special Note: Firebase Private Key

The `FIREBASE_PRIVATE_KEY` requires special handling:

1. In Vercel, paste the **entire** private key including the header and footer
2. Make sure it's on a **single line** with `\n` representing newlines
3. Wrap it in **double quotes**

Example format:
```
"-----BEGIN PRIVATE KEY-----\nMIIEvQIBADANBg...(your key)...=\n-----END PRIVATE KEY-----\n"
```

### 3. How to Add Environment Variables in Vercel

**Via Vercel Dashboard:**
1. Go to https://vercel.com/dashboard
2. Select your project
3. Click on "Settings" tab
4. Click on "Environment Variables" in the sidebar
5. For each variable:
   - Enter the **Name** (e.g., `MONGODB_URI`)
   - Enter the **Value** (e.g., your MongoDB connection string)
   - Select which environments (Production, Preview, Development)
   - Click "Save"

**Via Vercel CLI:**
```bash
# Install Vercel CLI if you haven't
npm i -g vercel

# Login to Vercel
vercel login

# Link your project
vercel link

# Add environment variables
vercel env add MONGODB_URI production
vercel env add FIREBASE_PROJECT_ID production
# ... repeat for all variables

# Pull environment variables to local
vercel env pull .env.local
```

### 4. MongoDB URI Configuration

If you don't have a MongoDB URI yet:

1. Go to https://www.mongodb.com/cloud/atlas
2. Create a free cluster or use existing one
3. Click "Connect" → "Connect your application"
4. Copy the connection string
5. Replace `<password>` with your actual database password
6. Replace `<database>` with `npi` (or your database name)

Example:
```
mongodb+srv://npi_user:yourpassword123@cluster0.abc123.mongodb.net/npi?retryWrites=true&w=majority
```

### 5. Firebase Configuration

If you need to set up Firebase Admin:

1. Go to https://console.firebase.google.com/
2. Select your project (or create a new one)
3. Go to Project Settings → Service Accounts
4. Click "Generate New Private Key"
5. Download the JSON file
6. Extract the values:
   - `project_id` → `FIREBASE_PROJECT_ID`
   - `client_email` → `FIREBASE_CLIENT_EMAIL`
   - `private_key` → `FIREBASE_PRIVATE_KEY` (keep the `\n` characters)

### 6. Redeploy Your Project

After adding all environment variables:

**Via Vercel Dashboard:**
1. Go to your project
2. Click on "Deployments" tab
3. Click the "..." menu on the latest deployment
4. Select "Redeploy"

**Via Vercel CLI:**
```bash
vercel --prod
```

**Via Git Push:**
```bash
git add .
git commit -m "Fix: MongoDB connection for build process"
git push origin main
```

Vercel will automatically trigger a new deployment.

### 7. Post-Deployment Steps

After successful deployment:

1. **Verify the deployment:**
   - Visit your production URL
   - Check if the homepage loads correctly

2. **Run city migration (first deployment only):**
   ```bash
   curl -X POST https://your-domain.com/api/cities/migrate
   ```

3. **Test admin login:**
   - Go to https://your-domain.com/admin/login
   - Login with your credentials

4. **Check API routes:**
   - Test a few API endpoints to ensure they're working
   - Example: `https://your-domain.com/api/imoveis`

## 🔍 Troubleshooting

### Build still fails with environment variable error

**Solution:** Make sure you've added **all required** environment variables to Vercel and they're enabled for the **Production** environment.

### MongoDB connection errors at runtime

**Possible causes:**
1. Incorrect MongoDB URI format
2. Database password contains special characters (URL encode them)
3. IP whitelist in MongoDB Atlas (add `0.0.0.0/0` to allow all IPs)

**Check MongoDB Atlas:**
- Network Access → Add IP Address → Allow Access from Anywhere (0.0.0.0/0)

### Firebase authentication errors

**Solution:**
1. Verify `FIREBASE_PRIVATE_KEY` is correctly formatted with `\n` characters
2. Make sure the private key is wrapped in double quotes
3. Check that `FIREBASE_PROJECT_ID` matches your Firebase project

### AWS S3 upload errors

**Solution:**
1. Verify AWS credentials are correct
2. Check that the S3 bucket exists and the region matches
3. Ensure the IAM user has S3 upload permissions

## 📝 Environment Variables Quick Reference

| Variable | Required | Purpose |
|----------|----------|---------|
| `MONGODB_URI` | ✅ Yes | Database connection |
| `FIREBASE_PROJECT_ID` | ✅ Yes | Firebase project identifier |
| `FIREBASE_CLIENT_EMAIL` | ✅ Yes | Firebase service account email |
| `FIREBASE_PRIVATE_KEY` | ✅ Yes | Firebase authentication key |
| `AWS_ACCESS_KEY_ID` | ✅ Yes | AWS S3 access |
| `AWS_SECRET_ACCESS_KEY` | ✅ Yes | AWS S3 secret |
| `AWS_REGION` | ✅ Yes | AWS S3 region |
| `NEXT_PUBLIC_SITE_URL` | ✅ Yes | Public site URL |
| `AWS_BUCKET_NAME` | ⚠️ Recommended | S3 bucket name |
| `NEXT_PUBLIC_GOOGLE_MAPS_API_KEY` | ❌ Optional | Google Maps integration |

## 🎯 Success Criteria

Your deployment is successful when:

- ✅ Build completes without errors
- ✅ Homepage loads correctly
- ✅ Property search works
- ✅ Admin panel is accessible
- ✅ Images load from S3
- ✅ Database queries return data

## 📞 Need Help?

If you continue to experience issues:

1. Check Vercel deployment logs: Dashboard → Deployments → Click deployment → View Function Logs
2. Verify all environment variables are set correctly
3. Test the build locally: `npm run build`
4. Check MongoDB Atlas network access settings

## 🔐 Security Note

**Never commit `.env.local` or any file with actual credentials to your repository!**

The `.env.example` file is a template only - it should contain placeholder values, not real credentials.

