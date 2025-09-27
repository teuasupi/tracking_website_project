# Sprint 1: Foundation & Authentication
## 🎯 Detailed Implementation Guide for Development Team

**Sprint Duration:** September 30 - October 6, 2025 (7 days)  
**Sprint Goal:** Establish working backend foundation and connect authentication to frontend  
**Success Criteria:** Users can register, login, and see their profile data from the database

---

## 📋 Sprint Overview

### **The Big Picture**
Right now we have a beautiful frontend with 18 complete pages, but it's completely disconnected from the backend. Sprint 1 is about building the foundation that makes everything else possible.

### **Current Status**
- ✅ **Frontend**: Complete UI, forms, components - but using mock data
- ❌ **Backend**: Fresh NestJS setup with no actual endpoints
- ❌ **Database**: No connection, no tables, no data
- ❌ **Integration**: Frontend and backend don't communicate

### **Sprint 1 End Goal**
- ✅ **Working authentication**: Users can actually register and login
- ✅ **Database connected**: Real user data stored in MariaDB
- ✅ **API endpoints**: Backend serves data that frontend can consume
- ✅ **Integration**: Frontend forms talk to backend APIs

---

## 🔥 Daily Breakdown & Tasks

### **Day 1 (Monday) - Database Foundation**
**Goal:** Get database connected and basic user table working

#### **Task 1: Configure MariaDB + TypeORM Connection**
**Owner:** Backend Developer  
**Priority:** 🚨 CRITICAL  
**Time Estimate:** 3-4 hours

**What to do:**
1. **Install required packages:**
   ```bash
   cd backend
   npm install @nestjs/typeorm typeorm mysql2 @nestjs/config
   ```

2. **Create database configuration:**
   ```typescript
   // backend/src/config/database.config.ts
   import { TypeOrmModuleOptions } from '@nestjs/typeorm';
   
   export const databaseConfig: TypeOrmModuleOptions = {
     type: 'mysql',
     host: process.env.DB_HOST || '127.0.0.1',
     port: parseInt(process.env.DB_PORT) || 3306,
     username: process.env.DB_USER || 'root',
     password: process.env.DB_PASSWORD || 'root',
     database: process.env.DB_NAME || 'TEUAS',
     entities: [__dirname + '/../**/*.entity{.ts,.js}'],
     synchronize: process.env.NODE_ENV !== 'production',
     logging: true
   };
   ```

3. **Create User entity:**
   ```typescript
   // backend/src/entities/user.entity.ts
   import { Entity, PrimaryGeneratedColumn, Column, CreateDateColumn, UpdateDateColumn } from 'typeorm';
   
   @Entity('users')
   export class User {
     @PrimaryGeneratedColumn()
     id: number;
   
     @Column({ unique: true })
     email: string;
   
     @Column()
     fullName: string;
   
     @Column()
     password: string;
   
     @Column({ nullable: true })
     graduationYear: number;
   
     @Column({ nullable: true })
     nim: string;
   
     @Column({ default: 'user' })
     role: string;
   
     @Column({ nullable: true })
     phoneNumber: string;
   
     @Column({ nullable: true })
     currentCompany: string;
   
     @Column({ nullable: true })
     position: string;
   
     @CreateDateColumn()
     createdAt: Date;
   
     @UpdateDateColumn()
     updatedAt: Date;
   }
   ```

4. **Update app.module.ts:**
   ```typescript
   // backend/src/app.module.ts
   import { Module } from '@nestjs/common';
   import { TypeOrmModule } from '@nestjs/typeorm';
   import { ConfigModule } from '@nestjs/config';
   import { databaseConfig } from './config/database.config';
   import { User } from './entities/user.entity';
   
   @Module({
     imports: [
       ConfigModule.forRoot(),
       TypeOrmModule.forRoot(databaseConfig),
       TypeOrmModule.forFeature([User])
     ],
   })
   export class AppModule {}
   ```

**Acceptance Criteria:**
- [ ] Backend starts without errors
- [ ] Database connection established
- [ ] Users table created in MariaDB
- [ ] Can see "Database connection established" in console logs

**Testing:**
```bash
npm run start:dev
# Should see: "Database connection established" and no TypeORM errors
```

---

### **Day 2 (Tuesday) - Authentication Endpoints**
**Goal:** Build working JWT authentication API

#### **Task 2: Build Authentication API**
**Owner:** Backend Developer  
**Priority:** 🚨 CRITICAL  
**Time Estimate:** 4-5 hours

**What to do:**

1. **Install auth packages:**
   ```bash
   npm install @nestjs/jwt @nestjs/passport passport passport-jwt bcryptjs
   npm install -D @types/passport-jwt @types/bcryptjs
   ```

2. **Create Auth Module structure:**
   ```bash
   mkdir src/auth
   touch src/auth/auth.module.ts
   touch src/auth/auth.controller.ts  
   touch src/auth/auth.service.ts
   touch src/auth/jwt.strategy.ts
   touch src/auth/jwt-auth.guard.ts
   ```

3. **Implement Auth Service:**
   ```typescript
   // backend/src/auth/auth.service.ts
   import { Injectable, UnauthorizedException } from '@nestjs/common';
   import { InjectRepository } from '@nestjs/typeorm';
   import { Repository } from 'typeorm';
   import { JwtService } from '@nestjs/jwt';
   import * as bcrypt from 'bcryptjs';
   import { User } from '../entities/user.entity';
   
   @Injectable()
   export class AuthService {
     constructor(
       @InjectRepository(User)
       private userRepository: Repository<User>,
       private jwtService: JwtService,
     ) {}
   
     async register(registerDto: any) {
       const hashedPassword = await bcrypt.hash(registerDto.password, 12);
       const user = this.userRepository.create({
         ...registerDto,
         password: hashedPassword,
       });
       await this.userRepository.save(user);
       
       const { password, ...result } = user;
       return result;
     }
   
     async login(email: string, password: string) {
       const user = await this.userRepository.findOne({ where: { email } });
       if (!user || !(await bcrypt.compare(password, user.password))) {
         throw new UnauthorizedException('Invalid credentials');
       }
   
       const payload = { email: user.email, sub: user.id };
       return {
         access_token: this.jwtService.sign(payload),
         user: { id: user.id, email: user.email, fullName: user.fullName }
       };
     }
   }
   ```

4. **Create Auth Controller:**
   ```typescript
   // backend/src/auth/auth.controller.ts
   import { Controller, Post, Body } from '@nestjs/common';
   import { AuthService } from './auth.service';
   
   @Controller('auth')
   export class AuthController {
     constructor(private authService: AuthService) {}
   
     @Post('register')
     async register(@Body() registerDto: any) {
       return this.authService.register(registerDto);
     }
   
     @Post('login')  
     async login(@Body() loginDto: { email: string; password: string }) {
       return this.authService.login(loginDto.email, loginDto.password);
     }
   }
   ```

**Acceptance Criteria:**
- [ ] POST `/auth/register` endpoint working
- [ ] POST `/auth/login` endpoint working
- [ ] Passwords properly hashed with bcrypt
- [ ] JWT tokens generated on successful login
- [ ] User data saved to database

**Testing with Postman:**
```json
// POST http://localhost:4000/auth/register
{
  "email": "test@example.com",
  "password": "password123",
  "fullName": "Test User"
}

// POST http://localhost:4000/auth/login  
{
  "email": "test@example.com",
  "password": "password123"
}
```

---

### **Day 3 (Wednesday) - User Management Endpoints**
**Goal:** Complete user CRUD operations

#### **Task 3: Create User Management Endpoints**
**Owner:** Backend Developer  
**Priority:** 🚨 CRITICAL  
**Time Estimate:** 3-4 hours

**What to do:**

1. **Create Users Module:**
   ```bash
   mkdir src/users
   touch src/users/users.module.ts
   touch src/users/users.controller.ts
   touch src/users/users.service.ts
   ```

2. **Implement Users Service:**
   ```typescript
   // backend/src/users/users.service.ts
   import { Injectable } from '@nestjs/common';
   import { InjectRepository } from '@nestjs/typeorm';
   import { Repository } from 'typeorm';
   import { User } from '../entities/user.entity';
   
   @Injectable()
   export class UsersService {
     constructor(
       @InjectRepository(User)
       private userRepository: Repository<User>,
     ) {}
   
     async findById(id: number): Promise<User> {
       const user = await this.userRepository.findOne({ where: { id } });
       const { password, ...result } = user;
       return result as User;
     }
   
     async updateProfile(id: number, updateData: any): Promise<User> {
       await this.userRepository.update(id, updateData);
       return this.findById(id);
     }
   
     async getAllAlumni(): Promise<User[]> {
       const users = await this.userRepository.find({
         select: ['id', 'fullName', 'email', 'graduationYear', 'currentCompany', 'position'],
       });
       return users;
     }
   }
   ```

3. **Create Users Controller:**
   ```typescript
   // backend/src/users/users.controller.ts
   import { Controller, Get, Put, Body, Param, UseGuards } from '@nestjs/common';
   import { JwtAuthGuard } from '../auth/jwt-auth.guard';
   import { UsersService } from './users.service';
   
   @Controller('users')
   export class UsersController {
     constructor(private usersService: UsersService) {}
   
     @Get('profile/:id')
     @UseGuards(JwtAuthGuard)
     async getProfile(@Param('id') id: number) {
       return this.usersService.findById(id);
     }
   
     @Put('profile/:id')
     @UseGuards(JwtAuthGuard) 
     async updateProfile(@Param('id') id: number, @Body() updateData: any) {
       return this.usersService.updateProfile(id, updateData);
     }
   
     @Get('alumni')
     async getAllAlumni() {
       return this.usersService.getAllAlumni();
     }
   }
   ```

**Acceptance Criteria:**
- [ ] GET `/users/profile/:id` returns user data
- [ ] PUT `/users/profile/:id` updates user profile
- [ ] GET `/users/alumni` returns list of all alumni
- [ ] JWT authentication protects profile endpoints
- [ ] No password data exposed in responses

**Testing:**
```bash
# Get JWT token from login, then:
# GET http://localhost:4000/users/profile/1
# Headers: Authorization: Bearer YOUR_JWT_TOKEN
```

---

### **Day 4 (Thursday) - Frontend Integration**
**Goal:** Connect frontend forms to backend APIs

#### **Task 4: Connect Frontend Auth Forms**
**Owner:** Frontend Developer + Backend Developer  
**Priority:** 🚨 CRITICAL  
**Time Estimate:** 4-6 hours

**What Frontend Developer does:**

1. **Update API client configuration:**
   ```typescript
   // frontend/src/lib/api/client.ts
   const API_BASE_URL = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:4000';
   
   export class ApiClient {
     private baseURL: string;
   
     constructor() {
       this.baseURL = API_BASE_URL;
     }
   
     async post(endpoint: string, data: any) {
       const response = await fetch(`${this.baseURL}${endpoint}`, {
         method: 'POST',
         headers: {
           'Content-Type': 'application/json',
         },
         body: JSON.stringify(data),
       });
       
       if (!response.ok) {
         throw new Error(`API Error: ${response.statusText}`);
       }
       
       return response.json();
     }
   }
   
   export const apiClient = new ApiClient();
   ```

2. **Update authentication service:**
   ```typescript
   // frontend/src/lib/auth/auth-service.ts
   import { apiClient } from '../api/client';
   
   export class AuthService {
     async login(email: string, password: string) {
       try {
         const response = await apiClient.post('/auth/login', {
           email,
           password
         });
         
         // Store JWT token
         localStorage.setItem('auth-token', response.access_token);
         return response;
       } catch (error) {
         throw new Error('Login failed');
       }
     }
   
     async register(userData: any) {
       try {
         const response = await apiClient.post('/auth/register', userData);
         return response;
       } catch (error) {
         throw new Error('Registration failed');
       }
     }
   }
   
   export const authService = new AuthService();
   ```

3. **Update login form:**
   ```typescript
   // frontend/src/app/(auth)/login/page.tsx
   // Find the form submission handler and update it:
   
   const handleSubmit = async (data: LoginFormData) => {
     try {
       setIsLoading(true);
       const response = await authService.login(data.email, data.password);
       
       // Redirect on success
       router.push('/dashboard');
     } catch (error) {
       setError('Invalid email or password');
     } finally {
       setIsLoading(false);
     }
   };
   ```

**What Backend Developer does:**

4. **Configure CORS for frontend:**
   ```bash
   cd backend
   npm install @nestjs/common cors
   ```

   ```typescript
   // backend/src/main.ts
   import { NestFactory } from '@nestjs/core';
   import { AppModule } from './app.module';
   
   async function bootstrap() {
     const app = await NestFactory.create(AppModule);
     
     // Enable CORS for frontend
     app.enableCors({
       origin: 'http://localhost:3000',
       methods: 'GET,HEAD,PUT,PATCH,POST,DELETE',
       credentials: true,
     });
     
     await app.listen(4000);
     console.log('🚀 Backend running on http://localhost:4000');
   }
   bootstrap();
   ```

**Acceptance Criteria:**
- [ ] Frontend login form connects to backend `/auth/login`
- [ ] Frontend register form connects to backend `/auth/register`
- [ ] CORS configured - no browser errors
- [ ] Success messages show on successful auth
- [ ] Error handling works for invalid credentials
- [ ] JWT tokens stored in frontend

**Testing:**
1. Start both frontend (`npm run dev`) and backend (`npm run start:dev`)
2. Go to `http://localhost:3000/login`
3. Try to register a new user
4. Try to login with those credentials
5. Should redirect to dashboard on success

---

### **Day 5 (Friday) - Environment & Polish**
**Goal:** Clean up configuration and prepare for Sprint 2

#### **Task 5: Environment & CORS Setup**
**Owner:** Full Team  
**Priority:** 🟠 HIGH  
**Time Estimate:** 2-3 hours

**What to do:**

1. **Backend Environment Setup:**
   ```bash
   # backend/.env
   DB_HOST=127.0.0.1
   DB_USER=root
   DB_PASSWORD=root
   DB_NAME=TEUAS
   DB_PORT=3306
   JWT_SECRET=your-super-secret-jwt-key-change-this-in-production
   NODE_ENV=development
   ```

2. **Frontend Environment Setup:**
   ```bash
   # frontend/.env.local
   NEXT_PUBLIC_API_URL=http://localhost:4000
   NEXT_PUBLIC_APP_URL=http://localhost:3000
   NEXTAUTH_URL=http://localhost:3000
   NEXTAUTH_SECRET=your-nextauth-secret-change-in-production
   ```

3. **Update package.json scripts:**
   ```json
   // Root package.json
   {
     "scripts": {
       "dev": "turbo dev",
       "build": "turbo build", 
       "start": "turbo start",
       "lint": "turbo lint",
       "type-check": "turbo type-check",
       "backend": "turbo dev --filter=@teuas/backend",
       "frontend": "turbo dev --filter=@teuas/frontend"
     }
   }
   ```

4. **Test full integration:**
   ```bash
   # Start everything
   npm run dev
   
   # In browser:
   # 1. Go to http://localhost:3000
   # 2. Try registration flow
   # 3. Try login flow  
   # 4. Check database has user data
   # 5. Check JWT token in browser storage
   ```

**Acceptance Criteria:**
- [ ] Both frontend and backend start with `npm run dev`
- [ ] Environment variables properly configured
- [ ] No CORS errors in browser console
- [ ] End-to-end auth flow works completely
- [ ] Database persists user data
- [ ] Ready for Sprint 2 work

---

## 🚨 Critical Success Factors

### **Daily Standups**
**Format:**
- **What did you complete yesterday?**
- **What are you working on today?**
- **Any blockers?**
- **Update sprint dashboard** (check off completed tasks)

### **Key Blockers to Watch:**
1. **Database connection issues** → Get DevOps help immediately
2. **CORS problems** → Backend and frontend developers pair
3. **JWT token problems** → Test with Postman first, then frontend
4. **TypeScript errors** → Don't ignore, fix immediately

### **Definition of Done (Each Task):**
- [ ] Code written and tested
- [ ] No TypeScript errors
- [ ] Tested with Postman (backend) or browser (frontend)
- [ ] Committed to git with clear message
- [ ] Sprint dashboard updated

---

## 📊 Sprint 1 Success Metrics

### **Technical Metrics:**
- [ ] Backend API responds to all endpoints
- [ ] Frontend can register and login users
- [ ] Database has at least 3 test users
- [ ] No console errors on successful auth flow

### **User Story Completion:**
- [ ] "As a user, I can register for an account"
- [ ] "As a user, I can login with my credentials" 
- [ ] "As a user, I can see my profile data"
- [ ] "As an admin, I can see the user was created in database"

### **Sprint Review Demo:**
**What to show stakeholders:**
1. **Registration form** → creates user in database
2. **Login form** → generates JWT token
3. **Database view** → shows real user data
4. **API testing** → Postman showing working endpoints

---

## 🆘 Escalation & Support

### **When to Escalate to PM:**
- Any task taking longer than estimated +50%
- Blockers that can't be resolved in 2 hours
- Team disagreement on technical approach
- Scope creep requests

### **Getting Help:**
- **Database issues**: Check MariaDB service, test connection
- **CORS problems**: Use browser dev tools, check Network tab
- **Authentication bugs**: Test API with Postman first
- **Integration issues**: Verify both services running on correct ports

---

**Remember: Sprint 1 success = October launch success. Everything else builds on this foundation!**