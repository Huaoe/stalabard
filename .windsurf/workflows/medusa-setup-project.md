---
description: Set up a new Medusa project with initial configuration and structure
---

# Setup Medusa Project

This workflow guides you through setting up a new Medusa project.

## Prerequisites

- Node.js 18+ installed
- npm or yarn package manager

## Steps

1. **Create new Medusa project**
   - Run: `npx create-medusa-app@latest`
   - Follow the prompts to configure your project
   - Choose your preferred database (PostgreSQL recommended)

2. **Install dependencies**
   - Navigate to project directory
   - Run: `npm install` or `yarn install`

3. **Configure environment variables**
   - Create `.env` file in project root
   - Set database connection string
   - Set JWT secret for authentication
   - Configure any third-party services

4. **Initialize database**
   - Run: `npx medusa db:migrate`
   - This creates initial database schema

5. **Create initial admin user**
   - Run: `npx medusa user:create`
   - Follow prompts to set email and password

6. **Start development server**
   - Run: `npm run dev` or `yarn dev`
   - Backend runs on http://localhost:9000
   - Admin dashboard on http://localhost:7001

7. **Verify setup**
   - Visit http://localhost:7001 and log in
   - Check that admin dashboard loads correctly
   - Test basic product creation

## Project Structure

```
medusa-project/
├── src/
│   ├── api/              # API routes
│   ├── modules/          # Custom modules
│   ├── workflows/        # Workflows
│   └── index.ts
├── medusa-config.ts      # Main configuration
├── package.json
└── .env                  # Environment variables
```

## Configuration Files

**medusa-config.ts** - Main configuration file:
- Database settings
- Module registration
- Plugin configuration
- Admin settings

**package.json** - Dependencies and scripts:
- Medusa framework
- Database drivers
- Development tools

**.env** - Environment variables:
- DATABASE_URL
- JWT_SECRET
- NODE_ENV
- API_ROUTE_PREFIX

## Validation Checklist

- [ ] Project created successfully
- [ ] Dependencies installed
- [ ] Environment variables configured
- [ ] Database initialized
- [ ] Admin user created
- [ ] Development server starts without errors
- [ ] Admin dashboard accessible and functional

## Next Steps

After setup:
1. Create your first custom module (see medusa-create-custom-module workflow)
2. Create API routes for your features
3. Build your storefront integration
4. Configure authentication and authorization

## Common Issues

**Database connection error**
- Verify DATABASE_URL in .env is correct
- Ensure PostgreSQL is running
- Check database credentials

**Admin dashboard won't load**
- Verify development server is running
- Check browser console for errors
- Clear browser cache and try again

**Port already in use**
- Change port in package.json dev script
- Or kill process using the port
