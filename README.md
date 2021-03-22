# Personal Finance Vue Frontend

A modern Vue.js 3 frontend application for the Personal Finance API, providing a complete user interface for managing personal finances.

## Features

- 🔐 **Authentication**: User registration and login with JWT tokens
- 📊 **Dashboard**: Overview of financial health with key metrics
- 💰 **Transaction Management**: Add, view, and delete income/expense transactions
- 🎯 **Budget Tracking**: Create and monitor budgets with spending alerts
- 📱 **Responsive Design**: Works on desktop, tablet, and mobile devices
- ⚡ **Modern Tech Stack**: Built with Vue 3, Pinia, Vue Router, and Vite

## Tech Stack

- **Vue 3** - Progressive JavaScript framework with Composition API
- **Pinia** - Modern state management for Vue
- **Vue Router** - Official router for Vue.js
- **Axios** - HTTP client for API communication
- **Vite** - Fast build tool and dev server
- **VueUse** - Collection of Vue composition utilities

## Prerequisites

- Node.js 16+ and npm
- Personal Finance API running on `http://localhost:3000`

## Installation

1. **Install dependencies**:
   ```bash
   npm install
   ```

2. **Start development server**:
   ```bash
   npm run dev
   ```

3. **Open in browser**: Navigate to `http://localhost:3001`

## Available Scripts

- `npm run dev` - Start development server with hot reload
- `npm run build` - Build for production
- `npm run preview` - Preview production build locally

## Project Structure

```
src/
├── components/         # Reusable Vue components
├── views/             # Page components
│   ├── Login.vue      # User authentication
│   ├── Register.vue   # User registration
│   ├── Dashboard.vue  # Financial overview
│   ├── Transactions.vue # Transaction management
│   └── Budgets.vue    # Budget tracking
├── stores/            # Pinia stores
│   └── auth.js        # Authentication state management
├── services/          # API services
│   └── api.js         # Axios configuration and interceptors
├── router/            # Vue Router configuration
│   └── index.js       # Route definitions and guards
├── App.vue            # Root component
├── main.js            # Application entry point
└── style.css          # Global styles
```

## Features Overview

### Authentication
- Secure JWT-based authentication
- Registration and login forms
- Protected routes with authentication guards
- Automatic token refresh handling

### Dashboard
- Total balance calculation
- Monthly income/expense tracking
- Recent transactions preview
- Active budget count
- Quick navigation to other sections

### Transaction Management
- Add new income/expense transactions
- View all transactions with filtering
- Delete transactions
- Categorize transactions
- Date-based organization

### Budget Tracking
- Create monthly/yearly budgets
- Track spending against budget limits
- Visual progress indicators
- Alert thresholds with color coding
- Activate/deactivate budgets
- Delete budget management

## API Integration

The frontend communicates with the Personal Finance API through:

- **Base URL**: `http://localhost:3000/api/v1`
- **Authentication**: JWT token in Authorization header
- **Endpoints**:
  - `POST /auth/register` - User registration
  - `POST /auth/login` - User login
  - `GET /auth/profile` - Get user profile
  - `GET /transactions` - List transactions
  - `POST /transactions` - Create transaction
  - `DELETE /transactions/:id` - Delete transaction
  - `GET /budgets` - List budgets
  - `POST /budgets` - Create budget
  - `PUT /budgets/:id` - Update budget
  - `DELETE /budgets/:id` - Delete budget

## Development

### Authentication Flow
1. User logs in/registers through forms
2. JWT token stored in localStorage
3. Token automatically included in API requests
4. Automatic redirect to login if token expires

### State Management
- Pinia stores handle authentication state
- Reactive data updates across components
- Centralized API error handling

### Styling
- Modern CSS with flexbox/grid layouts
- Responsive design for all screen sizes
- Consistent color scheme and typography
- Smooth transitions and hover effects

## Browser Support

- Chrome 87+
- Firefox 78+
- Safari 14+
- Edge 88+

## License

MIT License