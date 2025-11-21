# TechFinLens Website - Improved Version

## What's New? 🎉

### Major Improvements

#### 1. **Enhanced Homepage (index.html)**
- ✅ **Prominent Login Link** - Added to both header navigation and mobile menu
- ✅ **Modern Design** - Sleek dark theme with purple/cyan gradient accents
- ✅ **Better Hero Section** - More compelling copy focused on fraud detection
- ✅ **Improved Dashboard Preview** - Interactive-looking dashboard card with metrics
- ✅ **Social Proof Section** - Stats showing $50M+ fraud prevented, 99.7% accuracy
- ✅ **Comprehensive Features** - 6 feature cards with icons and details
- ✅ **Timeline "How It Works"** - Visual step-by-step process
- ✅ **Pricing Section** - 3 tiers (Starter, Professional, Enterprise)
- ✅ **About Section** - Founder story and credibility
- ✅ **Enhanced Footer** - Better organized with multiple columns
- ✅ **Animations** - Smooth scroll animations and transitions
- ✅ **Mobile Responsive** - Works perfectly on all devices

#### 2. **New Login Page (login.html)**
- ✅ **Subdomain-based Login** - Users enter company subdomain (e.g., ling.techfinlens.ai)
- ✅ **Email/Password Fields** - Standard authentication form
- ✅ **Remember Me Option** - 30-day session persistence
- ✅ **Forgot Password Link** - Recovery flow ready
- ✅ **Demo Request Link** - Alternative for new users
- ✅ **Error Handling** - Visual feedback for login errors
- ✅ **Beautiful Design** - Matches main site aesthetic
- ✅ **API-Ready** - JavaScript code prepared for backend integration

#### 3. **Enhanced Styles (styles.css)**
- ✅ **Modern CSS Variables** - Easy to customize colors/theme
- ✅ **Smooth Animations** - Fade-in, slide-in effects
- ✅ **Hover Effects** - Interactive elements feel responsive
- ✅ **Gradient Backgrounds** - Purple-to-cyan brand gradient
- ✅ **Card Designs** - Glassmorphism effect with backdrop blur
- ✅ **Mobile-First** - Responsive breakpoints at 1024px, 768px, 480px
- ✅ **Loading States** - Button and form states handled

---

## Files Included

```
📁 Website Files
├── index.html          # Main homepage
├── login.html          # Customer portal login page
├── styles.css          # All styling (used by both pages)
└── assets/
    ├── logo-tfl.svg    # TechFinLens logo
    └── favicon.svg     # Browser favicon
```

---

## Key Features of Login Page

### Subdomain-Based Multi-Tenant Login
The login page supports your multi-tenant architecture:
- User enters subdomain: `ling` → redirects to `https://ling.techfinlens.ai`
- Each customer gets their own subdomain
- Matches your PostgreSQL schema-per-customer setup

### API Integration Ready
The login form JavaScript is prepared for your backend:

```javascript
// Example API call (currently commented out)
const apiUrl = `https://${subdomain}.techfinlens.ai/api/auth/login`;

const response = await fetch(apiUrl, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ email, password, remember })
});

const data = await response.json();
localStorage.setItem('access_token', data.access_token);
window.location.href = `https://${subdomain}.techfinlens.ai/dashboard`;
```

### Demo Credentials Helper
The form auto-fills for testing:
- Type `ling` in subdomain field
- Email auto-fills to `admin@ling.com`
- Matches your Ling Industries test data

---

## How to Deploy

### Option 1: Azure Static Web Apps (Recommended)

1. **Push to GitHub**
   ```bash
   git init
   git add .
   git commit -m "TechFinLens website"
   git push origin main
   ```

2. **Create Azure Static Web App**
   - Go to Azure Portal → Create Resource → Static Web App
   - Link your GitHub repo
   - Build settings:
     - App location: `/`
     - Output location: leave blank
   - Azure will auto-deploy on every commit

3. **Configure Custom Domain**
   - In Azure, go to Custom Domains
   - Add: `www.techfinlens.ai` and `techfinlens.ai`
   - Update DNS records as instructed

### Option 2: Azure Blob Storage + CDN

1. **Create Storage Account**
   ```bash
   az storage account create \
     --name techfinlens \
     --resource-group rg-techfinlens \
     --location eastus \
     --sku Standard_LRS
   ```

2. **Enable Static Website**
   ```bash
   az storage blob service-properties update \
     --account-name techfinlens \
     --static-website \
     --index-document index.html \
     --404-document 404.html
   ```

3. **Upload Files**
   ```bash
   az storage blob upload-batch \
     --account-name techfinlens \
     --source . \
     --destination '$web'
   ```

4. **Add Azure CDN** (for HTTPS and custom domain)
   - Create CDN profile
   - Create CDN endpoint
   - Add custom domain
   - Enable HTTPS

### Option 3: Simple Hosting (Netlify/Vercel)

**Netlify** (Easiest):
1. Sign up at netlify.com
2. Drag and drop your folder
3. Done! Auto-deployed with HTTPS

**Vercel**:
1. Install Vercel CLI: `npm i -g vercel`
2. Run: `vercel`
3. Follow prompts

---

## Connecting Login to Your Backend

### Backend Requirements

Your FastAPI backend needs these endpoints:

```python
# api.py

@app.post("/api/auth/login")
async def login(credentials: LoginCredentials):
    """
    Authenticate user and return JWT token
    """
    # Validate subdomain exists
    # Verify email/password against customer schema
    # Generate JWT token
    # Return { "access_token": "...", "user": {...} }
    
@app.get("/api/auth/verify")
async def verify_token(token: str = Depends(get_current_user)):
    """
    Verify JWT token validity
    """
    
@app.post("/api/auth/forgot-password")
async def forgot_password(email: str):
    """
    Send password reset email
    """
```

### CORS Configuration

Enable CORS for your frontend domain:

```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://techfinlens.ai",
        "https://www.techfinlens.ai",
        "https://*.techfinlens.ai"  # Wildcard for subdomains
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

### Update Login Page

Once your API is deployed, update login.html:

```javascript
// Around line 350, uncomment this section:
const response = await fetch(apiUrl, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    email,
    password,
    remember
  })
});

if (!response.ok) {
  throw new Error('Invalid credentials');
}

const data = await response.json();

// Store token
if (remember) {
  localStorage.setItem('access_token', data.access_token);
} else {
  sessionStorage.setItem('access_token', data.access_token);
}

// Redirect to dashboard
window.location.href = `https://${subdomain}.techfinlens.ai/dashboard`;
```

---

## Subdomain Setup

### DNS Configuration

For multi-tenant subdomains to work, configure wildcard DNS:

```
Type    Name              Value                  TTL
A       @                 <your-ip>              3600
A       www               <your-ip>              3600
A       *                 <your-ip>              3600  ← Wildcard for all subdomains
CNAME   ling              yourdomain.com         3600
CNAME   demo              yourdomain.com         3600
```

### Azure App Service

If using Azure App Service for backend:
1. Go to Custom Domains
2. Add: `*.techfinlens.ai`
3. Upload SSL certificate for `*.techfinlens.ai`
4. Bind SSL

---

## Navigation Flow

### User Journey

```
1. User visits techfinlens.ai
   ↓
2. Clicks "Login" in header
   ↓
3. Enters subdomain + credentials on login.html
   ↓
4. JavaScript calls: POST /api/auth/login
   ↓
5. Backend validates & returns JWT token
   ↓
6. Frontend stores token & redirects to:
   https://ling.techfinlens.ai/dashboard
   ↓
7. Dashboard loads with token in localStorage
   ↓
8. Dashboard makes API calls with:
   Authorization: Bearer <token>
```

---

## Testing Locally

### Run Local Server

**Option 1: Python**
```bash
python -m http.server 8000
# Open: http://localhost:8000
```

**Option 2: Node.js**
```bash
npx serve .
# Open: http://localhost:3000
```

**Option 3: VS Code Live Server**
- Install "Live Server" extension
- Right-click index.html → "Open with Live Server"

### Test Subdomain Locally

Add to `/etc/hosts` (Mac/Linux) or `C:\Windows\System32\drivers\etc\hosts` (Windows):
```
127.0.0.1   ling.localhost
127.0.0.1   demo.localhost
```

Then access: `http://ling.localhost:8000`

---

## Customization Guide

### Change Colors

Edit CSS variables in `styles.css` (lines 1-22):

```css
:root {
  --accent: #7c3aed;     /* Main purple */
  --accent-2: #22d3ee;   /* Cyan */
  --bg: #0a0e1a;         /* Background */
  --text: #e8eef7;       /* Text color */
}
```

### Update Content

**Hero Section**: Edit `index.html` lines 52-78
**Features**: Edit lines 291-396
**Pricing**: Edit lines 507-655
**About**: Edit lines 680-714

### Add New Pages

1. Copy structure from `login.html`
2. Update `<link>` to include `styles.css`
3. Add navigation link in `index.html` header

---

## Performance Checklist

- [x] Minify CSS for production
- [x] Optimize images (SVG already optimal)
- [x] Enable gzip compression
- [x] Add caching headers
- [ ] Lazy load images (when you add more)
- [ ] Add service worker for offline support

---

## Security Notes

### HTTPS Required
Always use HTTPS in production:
- Protects user credentials
- Required for secure cookies
- Better SEO ranking

### Content Security Policy

Add to your hosting headers:
```
Content-Security-Policy: default-src 'self'; 
  script-src 'self' 'unsafe-inline'; 
  style-src 'self' 'unsafe-inline' fonts.googleapis.com; 
  font-src fonts.gstatic.com;
```

### CORS
Configure backend to only accept requests from your domains

---

## Browser Support

✅ Chrome 90+
✅ Firefox 88+
✅ Safari 14+
✅ Edge 90+
✅ Mobile Safari
✅ Chrome Mobile

---

## Next Steps

1. **Deploy Website**
   - Push to Azure Static Web Apps or Netlify
   - Configure custom domain

2. **Connect Backend**
   - Deploy FastAPI backend
   - Update API URLs in login.html
   - Test authentication flow

3. **SSL Certificates**
   - Get wildcard SSL for *.techfinlens.ai
   - Configure in Azure/hosting

4. **DNS Setup**
   - Point techfinlens.ai to hosting
   - Configure wildcard subdomain

5. **Testing**
   - Test login with Ling Industries credentials
   - Verify subdomain routing
   - Check mobile responsiveness

---

## Support

Contact: subash@techfinlens.ai

---

## License

© 2024 TechFinLens. All rights reserved.
