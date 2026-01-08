# Frontend Development Standards

**The Cozm - Frontend Engineering Guide**

This document outlines the development standards, patterns, and best practices for frontend development at The Cozm, based on our production codebases: **Cozm Employer Dashboard** and **Travel Visa The Cozm**.

---

## Table of Contents

1. [Tech Stack Overview](#tech-stack-overview)
2. [React & TypeScript Patterns](#react--typescript-patterns)
3. [State Management with Zustand](#state-management-with-zustand)
4. [API Client & Data Fetching](#api-client--data-fetching)
5. [Routing & Protected Routes](#routing--protected-routes)
6. [Component Library (Material-UI)](#component-library-material-ui)
7. [Tailwind CSS Styling Guidelines](#tailwind-css-styling-guidelines)
8. [Form Handling & Validation](#form-handling--validation)
9. [Internationalization (i18n)](#internationalization-i18n)
10. [Feature Flags & Multi-Tenancy](#feature-flags--multi-tenancy)
11. [Authorization & RBAC](#authorization--rbac)
12. [Project Structure](#project-structure)
13. [Common Pitfalls & Best Practices](#common-pitfalls--best-practices)

---

## Tech Stack Overview

### Core Frontend Stack
- **React 18.3+** - Modern functional components with hooks
- **TypeScript 5.6+** - Type-safe development
- **Vite 7.0+** - Fast build tool and dev server
- **React Router 6/7** - Client-side routing

### State & Data Management
- **Zustand 5.0+** - Lightweight state management with modular slices
- **TanStack Query (React Query) 5.61+** - Server state management with caching
- **Axios 1.12+** - HTTP client with interceptors

### UI & Styling
- **Material-UI (MUI) 6.4+** - Component library (DatePicker, DataGrid, etc.)
- **Tailwind CSS 4.1+** - Utility-first styling
- **Lucide React** - Icon library
- **React Select 5.7+** - Enhanced dropdowns

### Forms & Validation
- **Formik 2.4** - Complex form state management
- **React Hook Form** - Lightweight form handling (newer components)
- **Yup** - Schema validation

### Internationalization
- **i18next 25.x** - Translation framework
- **i18n-iso-countries** - Country name translations (18+ languages)

### Testing
- **Playwright** - End-to-end testing (Employer Dashboard)
- **Vitest** - Unit testing (Travel Visa)

---

## React & TypeScript Patterns

### Component Structure

We use functional components with TypeScript interfaces for props:

```typescript
// components/button.tsx
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  mode?: "fill" | "outline";
  children: React.ReactNode;
  icon?: LucideIcon;
  iconPosition?: "left" | "right";
  requiredPermission?: Permission;        // Employer Dashboard only
  permissionBehavior?: "hide" | "disable"; // Employer Dashboard only
}

const Button: React.FC<ButtonProps> = ({
  mode = "fill",
  icon: Icon,
  iconPosition = "left",
  children,
  requiredPermission,
  permissionBehavior = "hide",
  ...restProps
}) => {
  // Permission check (Employer Dashboard)
  const hasPermission = usePermission(requiredPermission || "view");

  if (requiredPermission && !hasPermission) {
    if (permissionBehavior === "hide") return null;
    if (permissionBehavior === "disable") {
      restProps.disabled = true;
    }
  }

  return (
    <button
      className={mode === "fill" ? "fill-btn" : "outline-btn"}
      {...restProps}
    >
      {Icon && iconPosition === "left" && <Icon size={20} />}
      {children}
      {Icon && iconPosition === "right" && <Icon size={20} />}
    </button>
  );
};

export default Button;
```

### Directory Structure

```
components/
├── button.tsx              # Reusable UI components
├── inputs/                 # Form input components
│   ├── text-input.tsx
│   ├── phone-input.tsx
│   ├── select-input.tsx
│   ├── date-input.tsx
│   └── styles/
├── hoc/                    # Higher-Order Components
│   ├── private-route.tsx
│   └── public-route.tsx
├── modal/                  # Modal components
├── sidebar/                # Navigation
├── loaders/                # Loading skeletons
└── shared/                 # Shared UI elements
```

### forwardRef Pattern for Input Components

Complex input components use `forwardRef` to expose the underlying input element:

```typescript
// components/inputs/text-input.tsx
interface TextInputProps {
  label?: string;
  value: string;
  onChange: (value: string) => void;
  required?: boolean;
  error?: string;
  fieldType?: FieldType;
  disabled?: boolean;
  autoPopulatedValues?: Record<string, FieldValue>;
  shouldShowError?: boolean;
}

const TextInput = forwardRef<HTMLInputElement, TextInputProps>((props, ref) => {
  const { user } = useBoundStore();
  const { t } = useTranslation();
  const [showPassword, setShowPassword] = useState(false);

  const handleChange = (value: string) => {
    props.onChange(value);
  };

  const isAutoPopulated = !!(
    props.fieldName && props.autoPopulatedValues?.[props.fieldName] === props.value
  );

  return (
    <div className="input-container">
      {props.label && (
        <label className="input-label">
          {props.label}
          {props.required && <span className="text-red-500">*</span>}
        </label>
      )}
      <input
        ref={ref}
        type={props.fieldType === "password" && showPassword ? "text" : props.fieldType}
        value={props.value}
        onChange={(e) => handleChange(e.target.value)}
        disabled={props.disabled}
        className={`input-field ${props.error ? "border-red-500" : ""}`}
      />
      {props.error && props.shouldShowError && (
        <span className="text-red-500 text-sm">{props.error}</span>
      )}
    </div>
  );
});

export default TextInput;
```

### TypeScript Path Aliases

Configure path aliases in `tsconfig.app.json`:

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@utils/*": ["src/utils/*"],
      "@pages/*": ["src/pages/*"],
      "@contexts/*": ["src/utils/context/*"]
    }
  }
}
```

**Usage:**
```typescript
import { useBoundStore } from "@/utils/zustand-slices/store";
import Button from "@components/button";
import { Login } from "@pages/login";
```

### Hooks Best Practices

- Use custom hooks for reusable logic
- Extract complex component logic into hooks
- Prefix hook names with `use`

```typescript
// utils/rbac/hooks.ts
export const usePermission = (permission: Permission): boolean => {
  const { user } = useUser();
  return hasPermission(user?.access_level, permission);
};

export const useRequirePermission = (
  permission: Permission,
  redirectTo: string = "/",
): void => {
  const { user } = useUser();
  const navigate = useNavigate();
  const hasRequiredPermission = hasPermission(user?.access_level, permission);

  useEffect(() => {
    if (!hasRequiredPermission) {
      navigate(redirectTo, { replace: true });
    }
  }, [hasRequiredPermission, navigate, redirectTo]);
};
```

---

## State Management with Zustand

We use **Zustand with modular slices** for predictable, type-safe state management.

### Store Architecture

```typescript
// utils/zustand-slices/store.ts
import { create } from "zustand";
import { userSlice } from "./user-slice";
import { locationConstantsSlice } from "./location-constants-slice";
import { questionnaireSlice } from "./questionnaire-slice";
import { assessmentSlice } from "./assessment-slice";
import { sidebarSlice } from "./sidebar-slice"; // Employer Dashboard
import { filedApplicationsSlice } from "./filed-applications-slice"; // Employer Dashboard

interface BoundStore
  extends ReturnType<typeof userSlice>,
    ReturnType<typeof locationConstantsSlice>,
    ReturnType<typeof questionnaireSlice>,
    ReturnType<typeof assessmentSlice>,
    ReturnType<typeof sidebarSlice>,
    ReturnType<typeof filedApplicationsSlice> {}

export const useBoundStore = create<BoundStore>(
  (set, get, api): BoundStore => ({
    ...userSlice(set, get, api),
    ...locationConstantsSlice(set, get, api),
    ...questionnaireSlice(set, get, api),
    ...assessmentSlice(set, get, api),
    ...sidebarSlice(set, get, api),
    ...filedApplicationsSlice(set, get, api),
  }),
);
```

### Slice Pattern

Each slice is a separate module with its own state and actions:

```typescript
// utils/zustand-slices/user-slice.ts
import { StateCreator } from "zustand";

export interface UserState {
  user: User | null;
  selectedLang: Language;
  termsAccepted: boolean;
}

export interface UserActions {
  setUser: (user: User | null) => void;
  setSelectedLang: (lang: Language) => void;
  setTermsAccepted: (accepted: boolean) => void;
  logout: () => Promise<void>;
}

export interface UserSlice extends UserState, UserActions {}

type UserSliceCreator = StateCreator<
  BoundStore,
  [],
  [],
  UserSlice
>;

export const userSlice: UserSliceCreator = (set, get) => {
  return {
    // State
    user: null,
    selectedLang: { countryCode: "GB", value: "en", label: "English" },
    termsAccepted: getTermsAcceptedFromStorage(),

    // Actions
    setUser: (user: User | null) => set({ user }),

    setSelectedLang: (lang) => {
      set({ selectedLang: lang });
      i18n.changeLanguage(lang.value);
    },

    setTermsAccepted: (accepted: boolean) => {
      setTermsAcceptedInStorage(accepted);
      set({ termsAccepted: accepted });
    },

    logout: async () => {
      try {
        await httpClient.post("auth0/logout");
        localStorage.clear();
        set({ user: null, termsAccepted: false });
        window.location.href = "/email-check";
      } catch (error) {
        console.error("Logout failed:", error);
      }
    },
  };
};
```

### Using the Store

```typescript
// In components
const Component = () => {
  // Select only what you need
  const user = useBoundStore((state) => state.user);
  const setUser = useBoundStore((state) => state.setUser);

  // Or destructure multiple values
  const { user, logout, selectedLang } = useBoundStore();

  return <div>{user?.name}</div>;
};
```

### Slice Organization

Common slices across projects:
- **user-slice** - User authentication state
- **questionnaire-slice** - Form state and field data
- **location-constants-slice** - Countries, currencies, etc.
- **assessment-slice** - Assessment flow state (Travel Visa)
- **sidebar-slice** - Sidebar state (Employer Dashboard)
- **filed-applications-slice** - Applications data (Employer Dashboard)

---

## API Client & Data Fetching

### Axios Configuration with Interceptors

We use Axios with request/response interceptors for authentication and error handling:

```typescript
// utils/config/axios.ts
import axios, { AxiosInstance, AxiosResponse, InternalAxiosRequestConfig } from "axios";

export const API_BASE_URL = `${import.meta.env.VITE_BACKEND_URL}/api/`;

const httpClient: AxiosInstance = axios.create({
  baseURL: API_BASE_URL,
});

// Request Interceptor - Add auth tokens and headers
httpClient.interceptors.request.use(
  (config: InternalAxiosRequestConfig) => {
    const isNotExternalApiRequest =
      config.url && !config.url.startsWith("http") && !config.url.startsWith("https");

    if (isNotExternalApiRequest) {
      const authTokensStr = localStorage.getItem("AUTH0_TOKEN_RESPONSE");
      const tenantStr = localStorage.getItem("tenantsInfo");

      // Portal identification
      config.headers["Request-Portal"] = "EE"; // "EE" for Travel Visa, "ER" for Employer Dashboard

      // Add authorization token
      if (authTokensStr) {
        const authTokens = JSON.parse(authTokensStr) as AuthTokens;
        config.headers["Authorization"] = `Bearer ${authTokens.id_token}`;
      }

      // Add tenant version
      if (tenantStr) {
        const tenant = JSON.parse(tenantStr) as TenantInfo;
        config.headers["Version"] = tenant.tenant;
      }
    }

    return config;
  },
  (error) => Promise.reject(error)
);

// Response Interceptor - Handle token refresh on 401
httpClient.interceptors.response.use(
  (response: AxiosResponse) => response,
  async (error: CustomAxiosError) => {
    const originalRequest = error.config;
    const authTokensStr = localStorage.getItem("AUTH0_TOKEN_RESPONSE");

    // Attempt token refresh on 401 (only once per request)
    if (authTokensStr && error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;

      // Travel Visa uses TokenRefreshManager singleton
      const tokenRefreshManager = TokenRefreshManager.getInstance();
      const id_token = await tokenRefreshManager.handleTokenRefresh();

      // Employer Dashboard uses refreshToken() from sso-functions.ts
      // const id_token = await refreshToken();

      if (id_token) {
        originalRequest.headers["Authorization"] = `Bearer ${id_token}`;
        return httpClient(originalRequest);
      }
    }

    return Promise.reject(error);
  }
);

export default httpClient;
```

### React Query (TanStack Query) Usage

Use React Query for server state management with caching:

```typescript
// Example: Fetching user data
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query";
import httpClient from "@utils/config/axios";

// Query
const useUserQuery = () => {
  return useQuery<User>({
    queryKey: ["user"],
    queryFn: () => httpClient.get<User>("users/me").then((res) => res.data),
    staleTime: 1000 * 60 * 60 * 24, // 24 hours
    gcTime: 1000 * 60 * 60 * 24,    // 24 hours (formerly cacheTime)
  });
};

// Mutation
const useSubmitApplication = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: (payload: ApplicationPayload) =>
      httpClient.post("applications", payload),
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ["applications"] });
    },
    onError: (error) => {
      errorToast("Failed to submit application");
    },
  });
};

// In component
const Component = () => {
  const { data: user, isLoading } = useUserQuery();
  const { mutate: submitApplication, isPending } = useSubmitApplication();

  const handleSubmit = () => {
    submitApplication(formData);
  };

  if (isLoading) return <Loader />;

  return <div>{user?.name}</div>;
};
```

### API Error Handling

Centralized error toast notifications:

```typescript
// utils/config/toast.ts
import { toast } from "react-toastify";

export const errorToast = (message: string) => {
  toast.error(message, {
    position: "top-right",
    autoClose: 5000,
  });
};

export const successToast = (message: string) => {
  toast.success(message, {
    position: "top-right",
    autoClose: 3000,
  });
};
```

---

## Routing & Protected Routes

### React Router Setup

```typescript
// main.tsx
import { createBrowserRouter, RouterProvider } from "react-router-dom";
import PrivateRoute from "@components/hoc/private-route";
import PublicRoute from "@components/hoc/public-route";
import { featureFlags } from "@utils/env/feature-flags";

const router = createBrowserRouter([
  {
    path: "/email-check",
    element: <PublicRoute><Login /></PublicRoute>,
    errorElement: <ErrorPage />,
  },
  {
    path: "/",
    element: <PrivateRoute><FileApplication /></PrivateRoute>,
    errorElement: <ErrorPage />,
  },
  // Feature flag-based conditional routing
  ...(featureFlags.ENABLE_ASSESSMENT ? [
    {
      path: "/assessment",
      element: <PrivateRoute><Assessment /></PrivateRoute>,
    },
  ] : []),
]);

const root = ReactDOM.createRoot(document.getElementById("root")!);
root.render(<RouterProvider router={router} />);
```

### Protected Route HOC

```typescript
// components/hoc/private-route.tsx
import { useEffect, useCallback } from "react";
import { useNavigate, useLocation, useSearchParams } from "react-router-dom";
import { useBoundStore } from "@/utils/zustand-slices/store";
import { getDataFromToken } from "@utils/config/sso-functions";
import { redirectUtils } from "@utils/config/redirect-utils";

interface PrivateRouteProps {
  children: React.ReactNode;
}

const PrivateRoute: React.FC<PrivateRouteProps> = ({ children }) => {
  const storageAccessToken = localStorage.getItem("AUTH0_TOKEN_RESPONSE") || "";
  const location = useLocation();
  const navigate = useNavigate();
  const [searchParams, setSearchParams] = useSearchParams();
  const { setUser } = useBoundStore();

  // Handle auth code from OAuth callback
  useEffect(() => {
    const code = searchParams.get("code");

    if (!storageAccessToken && !code) {
      redirectToLogin();
    } else if (code) {
      handleAuthCode(code);
    }
  }, [searchParams, storageAccessToken]);

  const redirectToLogin = () => {
    // Store current location for redirect after login
    redirectUtils.storeRedirectUrl(
      location.pathname,
      location.search,
      location.hash
    );
    window.location.href = "/email-check";
  };

  const handleAuthCode = useCallback(
    async (code: string): Promise<void> => {
      try {
        const res = await getDataFromToken(code);
        if (res) {
          setUser(res.user);

          // Restore pre-login URL
          const redirectUrl = redirectUtils.getRedirectUrl();
          if (redirectUrl && redirectUrl !== "/" && redirectUrl !== location.pathname) {
            navigate(redirectUrl, { replace: true });
            redirectUtils.clearRedirectUrl();
          } else {
            // Remove code from URL
            setSearchParams({});
          }
        }
      } catch (error) {
        console.error("Auth code handling failed:", error);
        redirectToLogin();
      }
    },
    [setUser, navigate, location.pathname]
  );

  return <>{children}</>;
};

export default PrivateRoute;
```

### Public Route HOC

```typescript
// components/hoc/public-route.tsx
const PublicRoute: React.FC<PublicRouteProps> = ({ children }) => {
  const storageAccessToken = localStorage.getItem("AUTH0_TOKEN_RESPONSE");
  const navigate = useNavigate();

  useEffect(() => {
    if (storageAccessToken) {
      navigate("/", { replace: true });
    }
  }, [storageAccessToken, navigate]);

  return <>{children}</>;
};

export default PublicRoute;
```

---

## Component Library (Material-UI)

We use **Material-UI (MUI) v6** for pre-built components, particularly for date pickers and data grids.

### Material-UI Components Used

**Travel Visa:**
- `Tooltip` - Contextual help text
- `DatePicker` (MUI X) - Date selection
- `CircularProgress` - Loading indicators

**Employer Dashboard (additional):**
- `DataGrid` (MUI X) - Advanced tables
- `TextField`, `Select`, `Checkbox` - Form controls
- `Drawer`, `Dialog` - Overlays

### DatePicker Setup

```typescript
import { AdapterDayjs } from "@mui/x-date-pickers/AdapterDayjs";
import { DatePicker } from "@mui/x-date-pickers/DatePicker";
import { LocalizationProvider } from "@mui/x-date-pickers/LocalizationProvider";
import dayjs, { Dayjs } from "dayjs";

interface DateInputProps {
  value: string;
  onChange: (date: string) => void;
  minDate?: string;
  maxDate?: string;
  label?: string;
  disabled?: boolean;
}

const DateInput: React.FC<DateInputProps> = ({
  value,
  onChange,
  minDate,
  maxDate,
  label,
  disabled,
}) => {
  const handleDateChange = (date: Dayjs | null) => {
    if (date) {
      onChange(date.format("YYYY-MM-DD"));
    }
  };

  return (
    <LocalizationProvider dateAdapter={AdapterDayjs}>
      <DatePicker
        label={label}
        value={value ? dayjs(value) : null}
        onChange={handleDateChange}
        minDate={minDate ? dayjs(minDate) : undefined}
        maxDate={maxDate ? dayjs(maxDate) : undefined}
        disabled={disabled}
        slotProps={{
          textField: {
            fullWidth: true,
            error: false,
          },
        }}
      />
    </LocalizationProvider>
  );
};

export default DateInput;
```

### MUI Global Style Overrides

Add z-index fixes for popovers in global CSS:

```css
/* index.css */
.MuiPickersPopper-root {
  z-index: 9999 !important;
}

.MuiTooltip-popper {
  z-index: 9999 !important;
}
```

### Tooltip Usage

```typescript
import { Tooltip } from "@mui/material";

<Tooltip title="This field is required for compliance" placement="top">
  <div>
    <TextInput {...props} />
  </div>
</Tooltip>
```

**Note:** Material-UI is used selectively. We primarily build custom components styled with Tailwind CSS.

---

## Tailwind CSS Styling Guidelines

### The Cozm Brand Colors

We use a consistent color palette across all Cozm applications. These colors are defined in the Tailwind config and can be used throughout the application.

**Primary Brand Colors:**

| Color Name | Hex Code | Usage | Tailwind Class |
|------------|----------|-------|----------------|
| **Cozm Teal** | `#44919c` | Primary brand color, CTAs, links | `bg-cozmTeal` `text-cozmTeal` |
| **Cozm Teal Light** | `#c7e5e9` | Backgrounds, hover states, subtle accents | `bg-cozmTealLight` |
| **Cozm Red** | `#bd4040` | Errors, warnings, important actions | `bg-cozmRed` `text-cozmRed` |
| **Cozm Red Light** | `#f1cfcf` | Error backgrounds, soft alerts | `bg-cozmRedLight` |
| **Cozm Purple** | `#ac40bd` | Secondary accents, highlights | `bg-cozmPurple` `text-cozmPurple` |
| **Cozm Gold** | `#bd8941` | Premium features, awards, special badges | `bg-cozmGold` `text-cozmGold` |
| **Cozm Gold Light** | `#f7e6ce` | Gold backgrounds, soft highlights | `bg-cozmGoldLight` |
| **Cozm Black** | `#121212` | Primary text, dark mode | `bg-cozmBlack` `text-cozmBlack` |
| **Cozm White** | `#FFFFFF` | White backgrounds, light text | `bg-cozmWhite` `text-cozmWhite` |
| **Cozm Grey** | `#f3f3f3` | Borders, dividers, subtle backgrounds | `bg-cozmGrey` `text-cozmGrey` |

**Semantic Colors:**

| Color Name | Hex Code | Usage | Tailwind Class |
|------------|----------|-------|----------------|
| **Primary** | `rgb(var(--primaryColor))` | Dynamic tenant color (changes per tenant) | `bg-primary` `text-primary` |
| **Secondary** | `#6c757d` | Secondary text, muted elements | `bg-secondary` `text-secondary` |
| **Light Error** | `#EF4444` | Error states, validation messages | `bg-lightError` `text-lightError` |
| **Light Success** | `#198754` | Success states, confirmations | `bg-lightSuccess` `text-lightSuccess` |
| **Light Warning** | `#F0C97D` | Warning states, caution messages | `bg-lightWarning` `text-lightWarning` |

**Color Usage Examples:**

```typescript
// Tailwind utility classes
<div className="bg-cozmTeal text-white p-4 rounded-lg">
  Primary Button Style
</div>

<div className="border border-cozmGrey bg-cozmTealLight p-6">
  Light background with subtle border
</div>

<span className="text-cozmRed font-semibold">
  Error Message
</span>

<button className="bg-cozmGold hover:bg-cozmGoldLight transition-colors">
  Premium Feature
</button>

// Dynamic tenant color (changes per client)
<button className="bg-primary text-white">
  Sign Up
</button>
```

**CSS Variables (for programmatic use):**

```css
:root {
  /* Dynamic tenant color (RGB format for opacity support) */
  --primaryColor: 68, 145, 156;  /* Default: Cozm Teal RGB */

  /* The Cozm Brand Colors */
  --cozm-teal: #44919c;
  --cozm-teal-light: #c7e5e9;
  --cozm-red: #bd4040;
  --cozm-red-light: #f1cfcf;
  --cozm-purple: #ac40bd;
  --cozm-gold: #bd8941;
  --cozm-gold-light: #f7e6ce;
  --cozm-black: #121212;
  --cozm-white: #FFFFFF;
  --cozm-grey: #f3f3f3;
}

/* Usage with opacity */
.semi-transparent-bg {
  background-color: rgba(var(--primaryColor), 0.5);
}
```

**Color Accessibility Guidelines:**

- **Text on Cozm Teal**: Use white text for optimal contrast
- **Text on Cozm Teal Light**: Use dark text (cozmBlack or secondary)
- **Text on Cozm Red**: Use white text
- **Text on Cozm Gold**: Use white or cozmBlack depending on weight
- **Minimum contrast ratio**: 4.5:1 for normal text, 3:1 for large text

### Tailwind Configuration

```javascript
// tailwind.config.js
module.exports = {
  content: ["./index.html", "./src/**/*.{js,jsx,ts,tsx}"],
  theme: {
    extend: {
      colors: {
        // Dynamic tenant color (changes per client)
        primary: "rgb(var(--primaryColor))",

        // Semantic colors
        secondary: "#6c757d",
        lightError: "#EF4444",
        lightSuccess: "#198754",
        lightWarning: "#F0C97D",

        // The Cozm brand colors
        cozmTeal: "#44919c",
        cozmTealLight: "#c7e5e9",
        cozmRed: "#bd4040",
        cozmRedLight: "#f1cfcf",
        cozmPurple: "#ac40bd",
        cozmGold: "#bd8941",
        cozmGoldLight: "#f7e6ce",
        cozmBlack: "#121212",
        cozmWhite: "#FFFFFF",
        cozmGrey: "#f3f3f3",
      },
      boxShadow: {
        mintGreen: "0 0px 0px -20px rgba(90, 238, 149, 0.16), 0 25px 50px -12px rgba(90, 238, 149, 0.25)",
        custom: "0 20px 25px -5px rgb(0 0 0 / 0.1), 0 8px 10px -6px rgb(0 0 0 / 0.1)",
      },
      fontFamily: {
        inter: ["Inter", "sans-serif"],
      },
    },
  },
  plugins: [
    function ({ addComponents }) {
      addComponents({
        ".outline-btn": {
          backgroundColor: "#FFFFFF",
          borderWidth: "1px",
          borderColor: "rgb(var(--primaryColor))",
          borderRadius: "9999px",
          padding: "0.5rem 1.5rem",
          color: "rgb(var(--primaryColor))",
          cursor: "pointer",
          transition: "all 0.2s ease-in-out",
          "&:hover": {
            backgroundColor: "rgb(var(--primaryColor))",
            color: "#FFFFFF",
          },
          "&:disabled": {
            opacity: "0.5",
            cursor: "not-allowed",
          },
        },
        ".fill-btn": {
          backgroundColor: "rgb(var(--primaryColor))",
          borderWidth: "1px",
          borderColor: "rgb(var(--primaryColor))",
          borderRadius: "9999px",
          padding: "0.5rem 1.5rem",
          color: "#FFFFFF",
          cursor: "pointer",
          transition: "all 0.2s ease-in-out",
          "&:hover": {
            opacity: "0.9",
          },
          "&:disabled": {
            opacity: "0.5",
            cursor: "not-allowed",
          },
        },
      });
    },
  ],
};
```

### Global Styles

```css
/* index.css */
@import url("https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap");
@import "tailwindcss";
@config "../tailwind.config.js";

body {
  color: #1e3542;
  font-family: "Inter", sans-serif !important;
  overflow-y: auto;
  margin: 0;
  padding: 0;
}

/* Remove number input spinners */
input[type="number"]::-webkit-outer-spin-button,
input[type="number"]::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

input[type="number"] {
  -moz-appearance: textfield;
}

/* Scrollbar styling */
::-webkit-scrollbar {
  width: 8px;
  height: 8px;
}

::-webkit-scrollbar-track {
  background: #f1f1f1;
}

::-webkit-scrollbar-thumb {
  background: #888;
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: #555;
}
```

### CSS Modules for Component-Specific Styles

Use CSS Modules for complex component styling:

```css
/* components/modal/styles.module.css */
.modalOverlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modalContent {
  background: white;
  border-radius: 8px;
  padding: 2rem;
  max-width: 600px;
  width: 90%;
  max-height: 90vh;
  overflow-y: auto;
}
```

```typescript
// components/modal/index.tsx
import styles from "./styles.module.css";

const Modal: React.FC<ModalProps> = ({ children, onClose }) => {
  return (
    <div className={styles.modalOverlay} onClick={onClose}>
      <div className={styles.modalContent} onClick={(e) => e.stopPropagation()}>
        {children}
      </div>
    </div>
  );
};
```

### Styling Best Practices

1. **Use Tailwind utility classes first** for simple styling
2. **Use CSS Modules** for complex, component-scoped styles
3. **Define custom colors in tailwind.config.js** instead of hardcoding hex values
4. **Use CSS custom properties** (CSS variables) for theme colors:
   ```css
   :root {
     --primaryColor: 68, 145, 156; /* #44919c in RGB */
   }
   ```
5. **Avoid inline styles** unless absolutely necessary (dynamic values)
6. **Use semantic class names** in CSS Modules (`.modalContent` not `.mc`)

---

## Form Handling & Validation

### Formik for Complex Forms

Use **Formik** for multi-step forms with complex validation:

```typescript
// pages/questionnaire/index.tsx
import { useFormik } from "formik";
import * as Yup from "yup";

const Questionnaire: React.FC = () => {
  const { t } = useTranslation();
  const queryClient = useQueryClient();

  // Generate validation schema dynamically
  const validationSchema = generateA1sValidationSchema(fields);

  const formik = useFormik({
    initialValues: initialFormValues,
    validationSchema: validationSchema,
    validateOnChange: true,
    validateOnBlur: true,
    onSubmit: async (values) => {
      try {
        await submitApplication(values);
        successToast(t("questionnaire.success"));
        queryClient.invalidateQueries({ queryKey: ["applications"] });
      } catch (error) {
        errorToast(t("questionnaire.error"));
      }
    },
  });

  return (
    <form onSubmit={formik.handleSubmit}>
      <TextInput
        name="firstName"
        value={formik.values.firstName}
        onChange={(value) => formik.setFieldValue("firstName", value)}
        error={formik.errors.firstName}
        shouldShowError={formik.touched.firstName}
      />
      <Button type="submit" disabled={formik.isSubmitting}>
        {t("common.submit")}
      </Button>
    </form>
  );
};
```

### Yup Validation Schemas

```typescript
// utils/validation/schemas.ts
import * as Yup from "yup";

export const generateA1sValidationSchema = (fields: Field[]) => {
  const schema: Record<string, any> = {};

  fields.forEach((field) => {
    if (field.type === "text") {
      schema[field.name] = Yup.string();

      if (field.required) {
        schema[field.name] = schema[field.name].required("This field is required");
      }

      if (field.minLength) {
        schema[field.name] = schema[field.name].min(
          field.minLength,
          `Minimum ${field.minLength} characters required`
        );
      }
    }

    if (field.type === "email") {
      schema[field.name] = Yup.string()
        .email("Invalid email format")
        .required("Email is required");
    }

    if (field.type === "number") {
      schema[field.name] = Yup.number()
        .typeError("Must be a number")
        .required("This field is required");
    }

    if (field.type === "date") {
      schema[field.name] = Yup.date()
        .typeError("Invalid date")
        .required("Date is required");
    }
  });

  return Yup.object().shape(schema);
};
```

### React Hook Form for Simple Forms

For simpler forms, use **React Hook Form** (lighter weight):

```typescript
import { useForm } from "react-hook-form";

interface LoginForm {
  email: string;
  password: string;
}

const Login: React.FC = () => {
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<LoginForm>();

  const onSubmit = async (data: LoginForm) => {
    try {
      await login(data);
    } catch (error) {
      errorToast("Login failed");
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input
        {...register("email", {
          required: "Email is required",
          pattern: {
            value: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
            message: "Invalid email address",
          },
        })}
        type="email"
      />
      {errors.email && <span>{errors.email.message}</span>}

      <input
        {...register("password", { required: "Password is required" })}
        type="password"
      />
      {errors.password && <span>{errors.password.message}</span>}

      <button type="submit" disabled={isSubmitting}>
        Login
      </button>
    </form>
  );
};
```

---

## Internationalization (i18n)

We support **18+ languages** using **i18next**.

### i18next Setup

```typescript
// main.tsx
import i18n from "i18next";
import { initReactI18next } from "react-i18next";
import LanguageDetector from "i18next-browser-languagedetector";
import countries from "i18n-iso-countries";

// Register all country locales
import en from "i18n-iso-countries/langs/en.json";
import de from "i18n-iso-countries/langs/de.json";
import fr from "i18n-iso-countries/langs/fr.json";
// ... register 18+ languages

countries.registerLocale(en);
countries.registerLocale(de);
countries.registerLocale(fr);
// ... etc

// Import translations
import enTranslations from "./locales/en/translation.json";
import deTranslations from "./locales/de/translation.json";
import frTranslations from "./locales/fr/translation.json";

i18n
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    resources: {
      en: { translation: enTranslations },
      de: { translation: deTranslations },
      fr: { translation: frTranslations },
      // ... 18+ languages
    },
    fallbackLng: "en",
    interpolation: {
      escapeValue: false,
    },
  });

export default i18n;
```

### Translation File Structure

```
locales/
├── en/
│   └── translation.json
├── de/
│   └── translation.json
├── fr/
│   └── translation.json
└── ... (18+ languages)
```

**Example translation.json:**
```json
{
  "common": {
    "submit": "Submit",
    "cancel": "Cancel",
    "save": "Save",
    "delete": "Delete",
    "edit": "Edit",
    "loading": "Loading..."
  },
  "questionnaire": {
    "title": "Application Form",
    "inputs": {
      "textInput": {
        "placeholder": "Enter text here"
      }
    },
    "success": "Application submitted successfully",
    "error": "Failed to submit application"
  },
  "validation": {
    "required": "This field is required",
    "invalidEmail": "Invalid email address",
    "minLength": "Minimum {{count}} characters required"
  }
}
```

### Using Translations in Components

```typescript
import { useTranslation } from "react-i18next";

const Component: React.FC = () => {
  const { t, i18n } = useTranslation();

  const changeLanguage = (lang: string) => {
    i18n.changeLanguage(lang);
  };

  return (
    <div>
      <h1>{t("questionnaire.title")}</h1>
      <p>{t("validation.minLength", { count: 5 })}</p>

      <select onChange={(e) => changeLanguage(e.target.value)}>
        <option value="en">English</option>
        <option value="de">Deutsch</option>
        <option value="fr">Français</option>
      </select>
    </div>
  );
};
```

### Language State Management

Store selected language in Zustand:

```typescript
// user-slice.ts
export interface UserSlice {
  selectedLang: Language;
  setSelectedLang: (lang: Language) => void;
}

export const userSlice: UserSliceCreator = (set) => ({
  selectedLang: { countryCode: "GB", value: "en", label: "English" },

  setSelectedLang: (lang) => {
    set({ selectedLang: lang });
    i18n.changeLanguage(lang.value);
    localStorage.setItem("selectedLang", JSON.stringify(lang));
  },
});
```

### Automated Translations

Use **json-autotranslate** for generating translations:

```json
// package.json
{
  "scripts": {
    "translate": "json-autotranslate --input ./src/locales/en --output ./src/locales --service google-translate --matcher=i18next --config ./translate.config.json"
  }
}
```

---

## Feature Flags & Multi-Tenancy

We use a **tenant-based feature flag system** to customize application behavior for different clients. Each tenant can have unique branding, enabled features, and compliance types.

### Tenant Configuration System

Feature flags are managed through tenant-specific environment files:

```
utils/env/
├── interface.ts          # Tenant resolver
├── helpers.ts            # Feature flag helpers
├── index.tsx            # Exported featureFlags object
├── cozm-env.ts          # The Cozm (default)
├── cibt-env.ts          # CIBT tenant
├── kion-env.ts          # Kion tenant
├── monzo-env.ts         # Monzo tenant
├── andersen-env.ts      # Andersen tenant
├── fragomen-env.ts      # Fragomen tenant
└── ... (other tenants)
```

### Tenant Resolution

The system automatically selects the correct configuration based on tenant information:

```typescript
// utils/env/interface.ts
const getTenantsInfo = () => {
  try {
    const tenantsInfo = localStorage.getItem("tenantsInfo");
    return tenantsInfo ? JSON.parse(tenantsInfo) : null;
  } catch (error) {
    return null;
  }
};

const resolveConfig = () => {
  const tenantsInfo = getTenantsInfo();
  let tenant = tenantsInfo?.tenant?.toUpperCase();

  // Override with VITE_INTERFACE if set
  const INTERFACE = import.meta.env.VITE_INTERFACE;
  if (INTERFACE?.trim()) tenant = INTERFACE;

  switch (tenant) {
    case "CIBT":
      return cibt_env;
    case "KION":
      return kion_env;
    case "MONZO":
      return monzo_env;
    // ... other tenants
    default:
      return cozm_env;  // Default to Cozm
  }
};

export const envInterface = resolveConfig();
```

### Tenant Environment Configuration

Each tenant environment file contains:

```typescript
// utils/env/cozm-env.ts
const base_env = {
  // Branding
  PRIMARY_COLOR: "64, 174, 189",  // RGB format for CSS variables
  logo: "https://public-cozm.s3.eu-central-1.amazonaws.com/assets/logos/the-cozm-logo.png",
  FAVICON: "https://public-cozm.s3.eu-central-1.amazonaws.com/assets/logos/cozm-favicon.ico",
  BrandName: "The Cozm Travel",
  interfaceName: "COZM",

  // Contact & Social Media
  OFFICIAL_MAIL: "support@thecozm.com",
  FACEBOOK_URL: "",
  LINKEDIN_URL: "https://www.linkedin.com/company/the-cozm",
  XTWITTER_URL: "",
  INSTAGRAM_URL: "",
  MEETING_LINK: "https://outlook.office365.com/owa/calendar/...",

  // Feature Flags
  ALLOWED_TENANTS: ["COZM", "CIBT", "KION", "MONZO", ...],
  ENABLE_ASSESSMENT: true,
  EXTEND_APPLICATION: false,
  ENABLE_SAVE_DRAFT: false,
  ENABLE_WORKDAY: true,
  ENABLE_LANGUAGE: true,
  ENABLE_UNSTABLE_FEATURES: true,
  CONCUR_TRAVEL_DETAILS: true,
  ASSISTED_FILINGS: true,

  // Compliance Types (hierarchical)
  COMPLIANCE_TYPES: generateHierarchicalComplianceTypes({
    category: "social-security",  // or "immigration", "tax-returns", "contractors"
    complianceTypes: {
      A1: true,
      "MSW-A1": true,
      COC: true,
      PWN: true,
      BV: true,
      ETA: true,
      // ... other types
    },
    personaComplianceTypes: {
      A1: true,
      COC: true,
      // ... persona-specific overrides
    },
  }),

  // Supported Languages
  SUPPORTED_LANGUAGES: {
    en: true,
    de: true,
    fr: false,
    es: false,
    // ... 30+ languages
  },
};
```

### Using Feature Flags

Import and use the `featureFlags` object throughout your application:

```typescript
// utils/env/index.tsx exports processed flags
import { featureFlags, primaryColor } from "@utils/env";

// Example: Conditional routing
const router = createBrowserRouter([
  {
    path: "/",
    element: <PrivateRoute><FileApplication /></PrivateRoute>,
  },
  // Only include assessment route if enabled
  ...(featureFlags.ENABLE_ASSESSMENT ? [
    {
      path: "/assessment",
      element: <PrivateRoute><Assessment /></PrivateRoute>,
    },
  ] : []),
]);

// Example: Conditional UI rendering
const Component = () => {
  return (
    <div>
      <h1>Applications</h1>

      {featureFlags.ENABLE_SAVE_DRAFT && (
        <Button onClick={handleSaveDraft}>Save Draft</Button>
      )}

      {featureFlags.ENABLE_WORKDAY && (
        <WorkdayIntegration />
      )}

      {featureFlags.ASSISTED_FILINGS && (
        <AssistedFilingBanner />
      )}
    </div>
  );
};

// Example: Compliance type filtering
const availableComplianceTypes = featureFlags.COMPLIANCE_TYPES;
// ['A1', 'MSW-A1', 'COC', 'PWN', 'BV', 'ETA']

const showForm = featureFlags.COMPLIANCE_TYPES.includes('A1');
```

### Available Feature Flags

**Core Features:**
- `ENABLE_ASSESSMENT` - Enable/disable assessment flow
- `ENABLE_SAVE_DRAFT` - Allow saving drafts
- `ENABLE_WORKDAY` - Workday integration
- `ENABLE_LANGUAGE` - Multi-language support
- `ENABLE_EXTEND_APPLICATION` - Application extension feature
- `ENABLE_UNSTABLE_FEATURES` - Beta features toggle
- `ASSISTED_FILINGS` - Assisted filing support
- `CONCUR_TRAVEL_DETAILS` - Concur travel integration

**Compliance Categories:**
- `COMPLIANCE_CATEGORY` - Active category: "social-security", "immigration", "tax-returns", or "contractors"
- `SOCIAL_SECURITY_ENABLED` - Social security compliance types active
- `IMMIGRATION_ENABLED` - Immigration compliance types active
- `TAX_RETURNS_ENABLED` - Tax return compliance types active
- `CONTRACTORS_ENABLED` - Contractor compliance types active

**Compliance Types:**
- `COMPLIANCE_TYPES` - Array of enabled compliance types (e.g., `['A1', 'COC', 'BV']`)
- `PERSONA_COMPLIANCE_TYPES` - Persona-specific overrides
- `COMBINED_COMPLIANCE_TYPES` - Union of all enabled types

**Legacy Flags (for backward compatibility):**
- `ENABLE_PWN_A1_ONLY` - Only PWN and A1 enabled
- `PWN_ENABLED` - PWN compliance type enabled
- `PWN_ONLY_ENABLED` - ONLY PWN enabled
- `COC_ENABLED` - COC compliance type enabled

**Language Support:**
- `SUPPORTED_LANGUAGES` - Array of enabled language codes (e.g., `['en', 'de', 'fr']`)

### Dynamic Branding with Feature Flags

Use tenant branding from `envInterface`:

```typescript
import { envInterface, primaryColor } from "@utils/env";

// Apply dynamic favicon
const favicon = document.querySelector('link[rel="icon"]') as HTMLLinkElement;
if (favicon) favicon.href = envInterface.FAVICON;

// Use tenant colors
document.documentElement.style.setProperty('--primaryColor', envInterface.PRIMARY_COLOR);

// Display tenant branding
const Header = () => (
  <div>
    <img src={envInterface.logo} alt={envInterface.BrandName} />
    <span>{envInterface.BrandName}</span>
  </div>
);

// Use primary color in styled components
const StyledButton = styled.button`
  background-color: ${primaryColor};
`;
```

### Environment-Specific Overrides

Each tenant can override settings per environment:

```typescript
const base_env = {
  ENABLE_ASSESSMENT: true,
  // ... base config
};

const dev_env = {
  ...base_env,
  ENABLE_UNSTABLE_FEATURES: true,  // Enable in dev
};

const prod_env = {
  ...base_env,
  ENABLE_UNSTABLE_FEATURES: false,  // Disable in prod
};

const getEnvironmentConfig = () => {
  const env = import.meta.env.VITE_ENV || "dev";

  switch (env.toLowerCase()) {
    case "production":
      return prod_env;
    case "staging":
      return staging_env;
    default:
      return dev_env;
  }
};
```

### Best Practices for Feature Flags

1. **Always check feature flags before rendering conditional features**
   ```typescript
   // Good
   {featureFlags.ENABLE_SAVE_DRAFT && <SaveDraftButton />}

   // Bad - assumes feature is always enabled
   <SaveDraftButton />
   ```

2. **Use conditional routing for major features**
   ```typescript
   ...(featureFlags.ENABLE_ASSESSMENT ? [assessmentRoute] : [])
   ```

3. **Document tenant-specific behavior in environment files**
   ```typescript
   // kion-env.ts
   ENABLE_WORKDAY: false,  // Kion doesn't use Workday integration
   ```

4. **Test with multiple tenant configurations**
   - Use `VITE_INTERFACE=CIBT npm run dev` to test as different tenants

5. **Gracefully handle missing features**
   ```typescript
   if (!featureFlags.ENABLE_SAVE_DRAFT) {
     return <InfoMessage>Draft saving not available</InfoMessage>;
   }
   ```

---

## Authorization & RBAC

**Employer Dashboard** implements role-based access control (RBAC) for multi-tenant applications.

### Permission System

```typescript
// utils/rbac/permissions.ts

// Define available permissions
export const PERMISSIONS = {
  VIEW: "view",
  CREATE: "create",
  EDIT: "edit",
  DELETE: "delete",
} as const;

export type Permission = (typeof PERMISSIONS)[keyof typeof PERMISSIONS];

// Define access levels (roles)
export const ACCESS_LEVEL = {
  VIEW_ONLY: "View Only",
  COMPREHENSIVE: "Comprehensive",
  SUPER_USER: "Superuser",
} as const;

export type AccessLevel = (typeof ACCESS_LEVEL)[keyof typeof ACCESS_LEVEL];

// Map roles to permissions
export const ROLE_PERMISSIONS: Record<AccessLevel, Permission[]> = {
  [ACCESS_LEVEL.VIEW_ONLY]: [PERMISSIONS.VIEW],
  [ACCESS_LEVEL.COMPREHENSIVE]: [
    PERMISSIONS.VIEW,
    PERMISSIONS.CREATE,
    PERMISSIONS.EDIT,
  ],
  [ACCESS_LEVEL.SUPER_USER]: [
    PERMISSIONS.VIEW,
    PERMISSIONS.CREATE,
    PERMISSIONS.EDIT,
    PERMISSIONS.DELETE,
  ],
};

// Check if user has permission
export const hasPermission = (
  accessLevel: AccessLevel | undefined,
  permission: Permission,
): boolean => {
  if (!accessLevel) return false;
  return ROLE_PERMISSIONS[accessLevel]?.includes(permission) || false;
};
```

### Permission Hooks

```typescript
// utils/rbac/hooks.ts
import { useUser } from "@contexts/user-provider";
import { hasPermission } from "./permissions";

export const usePermission = (permission: Permission): boolean => {
  const { user } = useUser();
  return hasPermission(user?.access_level, permission);
};

export const useRequirePermission = (
  permission: Permission,
  redirectTo: string = "/",
): void => {
  const { user } = useUser();
  const navigate = useNavigate();
  const hasRequiredPermission = hasPermission(user?.access_level, permission);

  useEffect(() => {
    if (!hasRequiredPermission) {
      navigate(redirectTo, { replace: true });
    }
  }, [hasRequiredPermission, navigate, redirectTo]);
};
```

### Protected Components

```typescript
// components/button.tsx
interface ButtonProps {
  requiredPermission?: Permission;
  permissionBehavior?: "hide" | "disable";
  // ... other props
}

const Button: React.FC<ButtonProps> = ({
  requiredPermission,
  permissionBehavior = "hide",
  ...props
}) => {
  const hasPermission = usePermission(requiredPermission || "view");

  if (requiredPermission && !hasPermission) {
    if (permissionBehavior === "hide") return null;
    if (permissionBehavior === "disable") {
      props.disabled = true;
    }
  }

  return <button {...props}>{props.children}</button>;
};
```

**Usage:**
```typescript
<Button
  requiredPermission="edit"
  permissionBehavior="hide"
  onClick={handleEdit}
>
  Edit Application
</Button>

<Button
  requiredPermission="delete"
  permissionBehavior="disable"
  onClick={handleDelete}
>
  Delete
</Button>
```

### Route Protection with Permissions

```typescript
// pages/settings/index.tsx
const Settings: React.FC = () => {
  // Redirect if user doesn't have edit permission
  useRequirePermission("edit", "/");

  return <div>Settings Page</div>;
};
```

---

## Project Structure

### Standard Directory Layout

```
src/
├── components/          # Reusable UI components
│   ├── button.tsx
│   ├── inputs/         # Form inputs
│   │   ├── text-input.tsx
│   │   ├── date-input.tsx
│   │   ├── select-input.tsx
│   │   └── phone-input.tsx
│   ├── hoc/            # Higher-Order Components
│   │   ├── private-route.tsx
│   │   └── public-route.tsx
│   ├── modal/          # Modal components
│   ├── sidebar/        # Navigation sidebar
│   ├── loaders/        # Loading states
│   └── shared/         # Shared utilities
│
├── pages/              # Route-level components
│   ├── questionnaire/
│   ├── applications/
│   ├── settings/
│   └── login.tsx
│
├── utils/              # Utilities and helpers
│   ├── config/         # Configuration files
│   │   ├── axios.ts    # HTTP client setup
│   │   ├── toast.ts    # Toast notifications
│   │   └── redirect-utils.ts
│   ├── context/        # React Context providers
│   │   └── user-provider.tsx
│   ├── rbac/           # Authorization (Employer Dashboard)
│   │   ├── permissions.ts
│   │   └── hooks.ts
│   ├── zustand-slices/ # State management slices
│   │   ├── store.ts
│   │   ├── user-slice.ts
│   │   ├── questionnaire-slice.ts
│   │   └── ...
│   ├── functions/      # Helper functions
│   ├── locales/        # Translation files
│   └── env/            # Environment configs
│
├── hooks/              # Custom React hooks
├── constants/          # App constants
├── assets/             # Static assets (images, fonts)
├── styles/             # Global CSS
├── main.tsx            # App entry point
├── index.css           # Global styles
└── vite-env.d.ts       # Vite type definitions
```

### File Naming Conventions

- **Components**: `kebab-case.tsx` (e.g., `text-input.tsx`)
- **Pages**: `kebab-case/index.tsx` (e.g., `questionnaire/index.tsx`)
- **Utils**: `kebab-case.ts` (e.g., `redirect-utils.ts`)
- **Types**: `PascalCase` interfaces/types (e.g., `UserSlice`, `ButtonProps`)
- **CSS Modules**: `*.module.css` (e.g., `styles.module.css`)

---

## Common Pitfalls & Best Practices

### Performance Optimization

**1. Memoization for Expensive Computations**
```typescript
import { useMemo } from "react";

const Component = ({ data }) => {
  const processedData = useMemo(() => {
    return data.map(item => heavyComputation(item));
  }, [data]);

  return <div>{processedData}</div>;
};
```

**2. Lazy Loading Routes**
```typescript
import { lazy, Suspense } from "react";

const Assessment = lazy(() => import("@pages/assessment"));

const router = createBrowserRouter([
  {
    path: "/assessment",
    element: (
      <Suspense fallback={<Loader />}>
        <PrivateRoute><Assessment /></PrivateRoute>
      </Suspense>
    ),
  },
]);
```

**3. Debounce Search Inputs**
```typescript
import { useMemo } from "react";
import debounce from "lodash/debounce";

const SearchComponent = () => {
  const debouncedSearch = useMemo(
    () => debounce((value: string) => {
      // Perform search
    }, 300),
    []
  );

  return <input onChange={(e) => debouncedSearch(e.target.value)} />;
};
```

### Code Organization

**1. Colocate Related Files**
```
components/
└── modal/
    ├── index.tsx
    ├── styles.module.css
    └── types.ts
```

**2. Use Barrel Exports**
```typescript
// components/inputs/index.ts
export { default as TextInput } from "./text-input";
export { default as DateInput } from "./date-input";
export { default as SelectInput } from "./select-input";

// Usage
import { TextInput, DateInput } from "@components/inputs";
```

**3. Separate Business Logic from UI**
```typescript
// hooks/use-application-form.ts
export const useApplicationForm = () => {
  const formik = useFormik({ /* ... */ });

  const handleSubmit = async () => { /* ... */ };
  const handleFieldChange = (field, value) => { /* ... */ };

  return { formik, handleSubmit, handleFieldChange };
};

// pages/questionnaire/index.tsx
const Questionnaire = () => {
  const { formik, handleSubmit } = useApplicationForm();

  return <form onSubmit={handleSubmit}>...</form>;
};
```

### Accessibility

**1. Semantic HTML**
```typescript
// Good
<button onClick={handleClick}>Submit</button>

// Bad
<div onClick={handleClick}>Submit</div>
```

**2. ARIA Labels**
```typescript
<button aria-label="Close modal" onClick={onClose}>
  <X size={20} />
</button>
```

**3. Keyboard Navigation**
```typescript
const handleKeyDown = (e: React.KeyboardEvent) => {
  if (e.key === "Enter" || e.key === " ") {
    handleClick();
  }
};

<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={handleKeyDown}
>
  Click me
</div>
```

### Testing

**Playwright E2E Tests (Employer Dashboard):**
```typescript
// tests/login.spec.ts
import { test, expect } from "@playwright/test";

test("user can login", async ({ page }) => {
  await page.goto("/email-check");

  await page.fill('input[name="email"]', "test@example.com");
  await page.click('button[type="submit"]');

  await expect(page).toHaveURL("/");
});
```

**Vitest Unit Tests (Travel Visa):**
```typescript
// components/button.test.tsx
import { render, screen } from "@testing-library/react";
import { describe, it, expect } from "vitest";
import Button from "./button";

describe("Button", () => {
  it("renders children correctly", () => {
    render(<Button>Click me</Button>);
    expect(screen.getByText("Click me")).toBeInTheDocument();
  });

  it("applies correct mode class", () => {
    const { rerender } = render(<Button mode="fill">Submit</Button>);
    expect(screen.getByRole("button")).toHaveClass("fill-btn");

    rerender(<Button mode="outline">Submit</Button>);
    expect(screen.getByRole("button")).toHaveClass("outline-btn");
  });
});
```

### Common Mistakes to Avoid

**1. Don't Store Derived State**
```typescript
// Bad
const [users, setUsers] = useState([]);
const [activeUsers, setActiveUsers] = useState([]);

useEffect(() => {
  setActiveUsers(users.filter(u => u.isActive));
}, [users]);

// Good
const [users, setUsers] = useState([]);
const activeUsers = useMemo(() => users.filter(u => u.isActive), [users]);
```

**2. Don't Forget Cleanup in useEffect**
```typescript
// Bad
useEffect(() => {
  const interval = setInterval(() => fetchData(), 5000);
}, []);

// Good
useEffect(() => {
  const interval = setInterval(() => fetchData(), 5000);
  return () => clearInterval(interval);
}, []);
```

**3. Don't Use Index as Key**
```typescript
// Bad
{items.map((item, index) => <div key={index}>{item}</div>)}

// Good
{items.map(item => <div key={item.id}>{item}</div>)}
```

**4. Don't Mutate State Directly**
```typescript
// Bad
const handleAdd = () => {
  users.push(newUser);
  setUsers(users);
};

// Good
const handleAdd = () => {
  setUsers([...users, newUser]);
};
```

**5. Don't Forget to Handle Loading/Error States**
```typescript
// Good
const { data, isLoading, error } = useQuery({ /* ... */ });

if (isLoading) return <Loader />;
if (error) return <ErrorMessage error={error} />;
return <div>{data}</div>;
```

**6. Don't Over-Fetch Data**
```typescript
// Use React Query's staleTime and gcTime
const { data } = useQuery({
  queryKey: ["user"],
  queryFn: fetchUser,
  staleTime: 1000 * 60 * 5, // 5 minutes
  gcTime: 1000 * 60 * 10,   // 10 minutes
});
```

**7. Don't Hardcode API URLs**
```typescript
// Bad
axios.get("https://api.example.com/users");

// Good
import httpClient from "@utils/config/axios";
httpClient.get("users"); // Uses configured base URL
```

**8. Don't Skip TypeScript Types**
```typescript
// Bad
const handleSubmit = (data: any) => { /* ... */ };

// Good
interface FormData {
  firstName: string;
  lastName: string;
  email: string;
}

const handleSubmit = (data: FormData) => { /* ... */ };
```

---

## Additional Resources

### Development Commands

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Run linting
npm run lint

# Format code
npm run format

# Run tests
npm test            # Playwright or Vitest depending on project
npm run test:ui     # UI mode for tests
npm run coverage    # Coverage report (Vitest)

# Generate translations
npm run translate
```

### Environment Variables

Create `.env` file:
```
VITE_BACKEND_URL=https://api.development.example.com
VITE_MIXPANEL_TOKEN=your_mixpanel_token
```

### Useful VS Code Extensions

- ESLint
- Prettier
- Tailwind CSS IntelliSense
- TypeScript Vue Plugin (Volar)
- i18n Ally (for translations)

### Reference Documentation

- [React Documentation](https://react.dev)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [Zustand Documentation](https://docs.pmnd.rs/zustand)
- [TanStack Query](https://tanstack.com/query/latest)
- [Tailwind CSS](https://tailwindcss.com/docs)
- [Material-UI](https://mui.com/material-ui/)
- [React Router](https://reactrouter.com/)
- [i18next](https://www.i18next.com/)

---

**Last Updated:** January 2026

**Maintained by:** The Cozm Engineering Team
