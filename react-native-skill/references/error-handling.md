# Error Handling & Logging Reference

## Table of Contents
1. [Error Handling Strategy](#error-handling-strategy)
2. [Error Boundaries](#error-boundaries)
3. [API Error Handling](#api-error-handling)
4. [Form Validation Errors](#form-validation-errors)
5. [Structured Logging](#structured-logging)
6. [Analytics Integration](#analytics-integration)
7. [Crash Reporting](#crash-reporting)

---

## Error Handling Strategy

### Error Types

```typescript
// types/errors.ts

// Base application error
export class AppError extends Error {
  constructor(
    message: string,
    public code: string,
    public context?: Record<string, unknown>
  ) {
    super(message);
    this.name = 'AppError';
  }
}

// Network/API errors
export class ApiError extends AppError {
  constructor(
    message: string,
    public statusCode: number,
    code: string,
    public response?: unknown
  ) {
    super(message, code);
    this.name = 'ApiError';
  }
  
  static fromResponse(response: Response, body?: unknown): ApiError {
    const statusMessages: Record<number, string> = {
      400: 'Invalid request',
      401: 'Please sign in to continue',
      403: 'You don\'t have permission to do this',
      404: 'Resource not found',
      422: 'Validation failed',
      429: 'Too many requests. Please try again later',
      500: 'Something went wrong. Please try again',
      503: 'Service temporarily unavailable',
    };
    
    return new ApiError(
      statusMessages[response.status] || 'An error occurred',
      response.status,
      `HTTP_${response.status}`,
      body
    );
  }
}

// Validation errors
export class ValidationError extends AppError {
  constructor(
    message: string,
    public field: string,
    public rule: string
  ) {
    super(message, 'VALIDATION_ERROR', { field, rule });
    this.name = 'ValidationError';
  }
}

// Network connectivity errors
export class NetworkError extends AppError {
  constructor(message = 'Please check your internet connection') {
    super(message, 'NETWORK_ERROR');
    this.name = 'NetworkError';
  }
}

// Authentication errors
export class AuthError extends AppError {
  constructor(
    message: string,
    code: 'SESSION_EXPIRED' | 'INVALID_CREDENTIALS' | 'UNAUTHORIZED'
  ) {
    super(message, code);
    this.name = 'AuthError';
  }
}
```

### Error Handling Utilities

```typescript
// utils/errorHandler.ts
import { logger } from '@/services/logger';
import { analytics } from '@/services/analytics';

export function handleError(
  error: unknown,
  context?: { screen?: string; action?: string }
): string {
  // Normalize error
  const normalizedError = normalizeError(error);
  
  // Log error
  logger.error('Error occurred', {
    error: normalizedError.message,
    code: normalizedError.code,
    stack: normalizedError.stack,
    ...context,
  });
  
  // Track in analytics
  analytics.trackError(normalizedError, context);
  
  // Return user-friendly message
  return getUserMessage(normalizedError);
}

function normalizeError(error: unknown): AppError {
  if (error instanceof AppError) {
    return error;
  }
  
  if (error instanceof Error) {
    return new AppError(error.message, 'UNKNOWN_ERROR', {
      originalError: error.name,
    });
  }
  
  return new AppError(
    'An unexpected error occurred',
    'UNKNOWN_ERROR',
    { rawError: String(error) }
  );
}

function getUserMessage(error: AppError): string {
  // Map error codes to user-friendly messages
  const messages: Record<string, string> = {
    NETWORK_ERROR: 'Please check your internet connection and try again.',
    SESSION_EXPIRED: 'Your session has expired. Please sign in again.',
    VALIDATION_ERROR: error.message,
    HTTP_429: 'Too many requests. Please wait a moment and try again.',
    HTTP_500: 'Something went wrong on our end. Please try again later.',
  };
  
  return messages[error.code] || 'Something went wrong. Please try again.';
}
```

---

## Error Boundaries

### Global Error Boundary

```typescript
// components/ErrorBoundary.tsx
import React, { Component, ErrorInfo, ReactNode } from 'react';
import { View, StyleSheet } from 'react-native';
import { Text } from '@/components/ui/Text';
import { Button } from '@/components/ui/Button';
import { logger } from '@/services/logger';
import { analytics } from '@/services/analytics';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }
  
  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    // Log the error
    logger.error('React Error Boundary caught error', {
      error: error.message,
      stack: error.stack,
      componentStack: errorInfo.componentStack,
    });
    
    // Track in analytics
    analytics.trackError(error, {
      type: 'react_error_boundary',
      componentStack: errorInfo.componentStack,
    });
    
    // Call optional error handler
    this.props.onError?.(error, errorInfo);
  }
  
  handleRetry = () => {
    this.setState({ hasError: false, error: null });
  };
  
  render() {
    if (this.state.hasError) {
      if (this.props.fallback) {
        return this.props.fallback;
      }
      
      return (
        <View style={styles.container}>
          <Text variant="title2" align="center">
            Something went wrong
          </Text>
          <Text
            variant="body"
            color="secondary"
            align="center"
            style={styles.message}
          >
            We're sorry, but something unexpected happened. Please try again.
          </Text>
          <Button
            label="Try Again"
            onPress={this.handleRetry}
            style={styles.button}
          />
        </View>
      );
    }
    
    return this.props.children;
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    padding: 24,
  },
  message: {
    marginTop: 12,
    marginBottom: 24,
  },
  button: {
    minWidth: 200,
  },
});
```

### Screen-Level Error Boundary

```typescript
// components/ScreenErrorBoundary.tsx
import { useNavigation } from '@react-navigation/native';

interface ScreenErrorFallbackProps {
  error: Error;
  resetError: () => void;
}

export function ScreenErrorFallback({
  error,
  resetError,
}: ScreenErrorFallbackProps) {
  const navigation = useNavigation();
  
  const handleGoBack = () => {
    if (navigation.canGoBack()) {
      navigation.goBack();
    } else {
      resetError();
    }
  };
  
  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.content}>
        <AlertCircle size={64} color={colors.systemRed} />
        <Text variant="title2" style={styles.title}>
          Oops!
        </Text>
        <Text variant="body" color="secondary" align="center">
          We couldn't load this screen. Please try again.
        </Text>
        
        {__DEV__ && (
          <View style={styles.debug}>
            <Text variant="caption1" color="tertiary">
              {error.message}
            </Text>
          </View>
        )}
        
        <View style={styles.actions}>
          <Button label="Try Again" onPress={resetError} />
          <Button
            label="Go Back"
            variant="secondary"
            onPress={handleGoBack}
          />
        </View>
      </View>
    </SafeAreaView>
  );
}
```

### Query Error Boundary (TanStack Query)

```typescript
// components/QueryErrorBoundary.tsx
import { QueryErrorResetBoundary } from '@tanstack/react-query';
import { ErrorBoundary } from 'react-error-boundary';

export function QueryErrorBoundary({ children }: { children: ReactNode }) {
  return (
    <QueryErrorResetBoundary>
      {({ reset }) => (
        <ErrorBoundary
          onReset={reset}
          fallbackRender={({ error, resetErrorBoundary }) => (
            <QueryErrorFallback
              error={error}
              onRetry={resetErrorBoundary}
            />
          )}
        >
          {children}
        </ErrorBoundary>
      )}
    </QueryErrorResetBoundary>
  );
}
```

---

## API Error Handling

### API Client with Error Handling

```typescript
// services/api/client.ts
import { ApiError, NetworkError, AuthError } from '@/types/errors';
import { logger } from '@/services/logger';
import { authStore } from '@/store/authStore';

const BASE_URL = Config.API_URL;

interface RequestConfig extends RequestInit {
  timeout?: number;
}

async function request<T>(
  endpoint: string,
  config: RequestConfig = {}
): Promise<T> {
  const { timeout = 30000, ...fetchConfig } = config;
  
  // Create abort controller for timeout
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeout);
  
  try {
    const response = await fetch(`${BASE_URL}${endpoint}`, {
      ...fetchConfig,
      signal: controller.signal,
      headers: {
        'Content-Type': 'application/json',
        ...getAuthHeaders(),
        ...fetchConfig.headers,
      },
    });
    
    clearTimeout(timeoutId);
    
    // Handle response
    if (!response.ok) {
      const body = await parseResponseBody(response);
      throw ApiError.fromResponse(response, body);
    }
    
    return await response.json();
    
  } catch (error) {
    clearTimeout(timeoutId);
    
    // Handle abort (timeout)
    if (error instanceof Error && error.name === 'AbortError') {
      throw new NetworkError('Request timed out. Please try again.');
    }
    
    // Handle network errors
    if (error instanceof TypeError && error.message === 'Network request failed') {
      throw new NetworkError();
    }
    
    // Handle auth errors
    if (error instanceof ApiError && error.statusCode === 401) {
      await handleUnauthorized();
      throw new AuthError('Your session has expired', 'SESSION_EXPIRED');
    }
    
    throw error;
  }
}

function getAuthHeaders(): Record<string, string> {
  const token = authStore.getState().accessToken;
  return token ? { Authorization: `Bearer ${token}` } : {};
}

async function parseResponseBody(response: Response): Promise<unknown> {
  try {
    return await response.json();
  } catch {
    return null;
  }
}

async function handleUnauthorized(): Promise<void> {
  // Clear auth state
  authStore.getState().signOut();
  // Navigate to login (use navigation service)
}

// Exported methods
export const api = {
  get: <T>(endpoint: string, config?: RequestConfig) =>
    request<T>(endpoint, { ...config, method: 'GET' }),
    
  post: <T>(endpoint: string, body: unknown, config?: RequestConfig) =>
    request<T>(endpoint, {
      ...config,
      method: 'POST',
      body: JSON.stringify(body),
    }),
    
  put: <T>(endpoint: string, body: unknown, config?: RequestConfig) =>
    request<T>(endpoint, {
      ...config,
      method: 'PUT',
      body: JSON.stringify(body),
    }),
    
  delete: <T>(endpoint: string, config?: RequestConfig) =>
    request<T>(endpoint, { ...config, method: 'DELETE' }),
};
```

### React Query Error Handling

```typescript
// hooks/useApiQuery.ts
import { useQuery, UseQueryOptions } from '@tanstack/react-query';
import { handleError } from '@/utils/errorHandler';
import { useToast } from '@/hooks/useToast';

export function useApiQuery<T>(
  queryKey: string[],
  queryFn: () => Promise<T>,
  options?: Omit<UseQueryOptions<T>, 'queryKey' | 'queryFn'>
) {
  const toast = useToast();
  
  return useQuery({
    queryKey,
    queryFn,
    retry: (failureCount, error) => {
      // Don't retry on auth errors
      if (error instanceof AuthError) return false;
      // Don't retry on validation errors
      if (error instanceof ApiError && error.statusCode === 422) return false;
      // Retry up to 3 times for other errors
      return failureCount < 3;
    },
    onError: (error) => {
      const message = handleError(error, { queryKey: queryKey.join('/') });
      toast.show({ type: 'error', message });
    },
    ...options,
  });
}

// Usage
function useUser(userId: string) {
  return useApiQuery(
    ['user', userId],
    () => api.get<User>(`/users/${userId}`),
    { staleTime: 5 * 60 * 1000 }
  );
}
```

---

## Form Validation Errors

### React Hook Form Integration

```typescript
// hooks/useFormWithErrors.ts
import { useForm, UseFormProps, FieldValues } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { ZodSchema } from 'zod';
import { ApiError } from '@/types/errors';

export function useFormWithErrors<T extends FieldValues>(
  schema: ZodSchema<T>,
  options?: UseFormProps<T>
) {
  const form = useForm<T>({
    resolver: zodResolver(schema),
    mode: 'onBlur',
    ...options,
  });
  
  // Handle API validation errors
  const setApiErrors = (error: ApiError) => {
    if (error.statusCode === 422 && error.response) {
      const validationErrors = error.response as Record<string, string[]>;
      
      Object.entries(validationErrors).forEach(([field, messages]) => {
        form.setError(field as any, {
          type: 'server',
          message: messages[0],
        });
      });
    }
  };
  
  return { ...form, setApiErrors };
}

// Usage
const schema = z.object({
  email: z.string().email('Please enter a valid email'),
  password: z.string().min(8, 'Password must be at least 8 characters'),
});

function LoginForm() {
  const { control, handleSubmit, setApiErrors } = useFormWithErrors(schema);
  
  const onSubmit = async (data: z.infer<typeof schema>) => {
    try {
      await api.post('/auth/login', data);
    } catch (error) {
      if (error instanceof ApiError) {
        setApiErrors(error);
      }
    }
  };
  
  return (
    <Form onSubmit={handleSubmit(onSubmit)}>
      <FormInput name="email" control={control} label="Email" />
      <FormInput name="password" control={control} label="Password" secure />
      <Button label="Sign In" onPress={handleSubmit(onSubmit)} />
    </Form>
  );
}
```

### Error Display Component

```typescript
// components/forms/FormError.tsx
import Animated, { FadeIn, FadeOut } from 'react-native-reanimated';
import { AlertCircle } from 'lucide-react-native';

interface FormErrorProps {
  message?: string;
}

export function FormError({ message }: FormErrorProps) {
  if (!message) return null;
  
  return (
    <Animated.View
      entering={FadeIn.duration(200)}
      exiting={FadeOut.duration(200)}
      style={styles.container}
    >
      <AlertCircle size={14} color={colors.systemRed} />
      <Text variant="caption1" style={styles.text}>
        {message}
      </Text>
    </Animated.View>
  );
}
```

---

## Structured Logging

### Logger Service

```typescript
// services/logger.ts
type LogLevel = 'debug' | 'info' | 'warn' | 'error';

interface LogEntry {
  level: LogLevel;
  message: string;
  timestamp: string;
  context?: Record<string, unknown>;
}

interface LogContext {
  screen?: string;
  component?: string;
  action?: string;
  userId?: string;
  sessionId?: string;
  [key: string]: unknown;
}

class Logger {
  private sessionId: string;
  private userId?: string;
  private buffer: LogEntry[] = [];
  private flushInterval: number;
  
  constructor() {
    this.sessionId = generateUUID();
    this.flushInterval = setInterval(() => this.flush(), 30000);
  }
  
  setUserId(userId: string | undefined) {
    this.userId = userId;
  }
  
  debug(message: string, context?: LogContext) {
    this.log('debug', message, context);
  }
  
  info(message: string, context?: LogContext) {
    this.log('info', message, context);
  }
  
  warn(message: string, context?: LogContext) {
    this.log('warn', message, context);
  }
  
  error(message: string, context?: LogContext) {
    this.log('error', message, context);
  }
  
  private log(level: LogLevel, message: string, context?: LogContext) {
    const entry: LogEntry = {
      level,
      message,
      timestamp: new Date().toISOString(),
      context: {
        ...context,
        sessionId: this.sessionId,
        userId: this.userId,
        platform: Platform.OS,
        appVersion: Config.APP_VERSION,
      },
    };
    
    // Console output in dev
    if (__DEV__) {
      const color = {
        debug: '\x1b[36m',
        info: '\x1b[32m',
        warn: '\x1b[33m',
        error: '\x1b[31m',
      }[level];
      console.log(`${color}[${level.toUpperCase()}]\x1b[0m`, message, context);
    }
    
    // Buffer for batch sending
    this.buffer.push(entry);
    
    // Immediately flush errors
    if (level === 'error') {
      this.flush();
    }
  }
  
  private async flush() {
    if (this.buffer.length === 0) return;
    
    const entries = [...this.buffer];
    this.buffer = [];
    
    try {
      // Send to logging service (e.g., Datadog, LogRocket)
      await fetch(Config.LOGGING_ENDPOINT, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ entries }),
      });
    } catch {
      // Re-add to buffer if send fails
      this.buffer.unshift(...entries);
    }
  }
  
  destroy() {
    clearInterval(this.flushInterval);
    this.flush();
  }
}

export const logger = new Logger();
```

### Usage Patterns

```typescript
// Screen-level logging
function ProfileScreen() {
  useEffect(() => {
    logger.info('Screen viewed', { screen: 'Profile' });
  }, []);
  
  const handleEditProfile = () => {
    logger.info('Profile edit started', {
      screen: 'Profile',
      action: 'edit_profile',
    });
  };
}

// API logging
async function fetchUser(userId: string) {
  logger.debug('Fetching user', { userId });
  
  try {
    const user = await api.get(`/users/${userId}`);
    logger.info('User fetched successfully', { userId });
    return user;
  } catch (error) {
    logger.error('Failed to fetch user', {
      userId,
      error: error.message,
    });
    throw error;
  }
}

// Action logging
function useAddToCart() {
  const addToCart = (product: Product, quantity: number) => {
    logger.info('Product added to cart', {
      action: 'add_to_cart',
      productId: product.id,
      productName: product.name,
      quantity,
      price: product.price,
    });
  };
  
  return addToCart;
}
```

---

## Analytics Integration

### Analytics Service

```typescript
// services/analytics.ts
import * as Analytics from 'expo-analytics';

type EventName =
  | 'screen_view'
  | 'button_click'
  | 'sign_up'
  | 'sign_in'
  | 'purchase'
  | 'error';

interface EventParams {
  [key: string]: string | number | boolean | undefined;
}

class AnalyticsService {
  private userId?: string;
  
  identify(userId: string, traits?: Record<string, unknown>) {
    this.userId = userId;
    Analytics.identify(userId, traits);
  }
  
  reset() {
    this.userId = undefined;
    Analytics.reset();
  }
  
  track(event: EventName, params?: EventParams) {
    Analytics.track(event, {
      ...params,
      timestamp: Date.now(),
    });
  }
  
  screen(screenName: string, params?: EventParams) {
    this.track('screen_view', {
      screen_name: screenName,
      ...params,
    });
  }
  
  trackError(error: Error, context?: Record<string, unknown>) {
    this.track('error', {
      error_name: error.name,
      error_message: error.message,
      ...context,
    });
  }
}

export const analytics = new AnalyticsService();
```

---

## Crash Reporting

### Firebase Crashlytics Integration

Crashlytics is the recommended crash/telemetry tool for this stack. It uses
`@react-native-firebase/*`, which has **native code** — it does **not** run in
Expo Go. Use a development build (`npx expo run:ios` / `run:android` or an EAS
dev build) and add the config plugins so `expo prebuild` wires the native deps.

```bash
npx expo install @react-native-firebase/app @react-native-firebase/crashlytics
# Drop GoogleService-Info.plist (iOS) + google-services.json (Android) in the
# project root and reference them from app.json (see plugin config below).
```

```jsonc
// app.json — config plugins so prebuild links the native SDKs
{
  "expo": {
    "ios":     { "googleServicesFile": "./GoogleService-Info.plist" },
    "android": { "googleServicesFile": "./google-services.json" },
    "plugins": [
      "@react-native-firebase/app",
      "@react-native-firebase/crashlytics"
    ]
  }
}
```

```typescript
// services/crashReporting.ts
import crashlytics from '@react-native-firebase/crashlytics';

// Crashlytics auto-captures native + unhandled JS crashes once linked.
// This just gates collection by env and exposes helpers for manual reports.
export async function initCrashReporting() {
  // Disable in dev so local crashes don't pollute the dashboard.
  await crashlytics().setCrashlyticsCollectionEnabled(!__DEV__);
}

export function setUserContext(user: User | null) {
  if (user) {
    crashlytics().setUserId(user.id);
    // NOTE: don't log raw email/PII as an attribute in production —
    // setUserId is the durable, privacy-safe handle. See the security
    // checklist's "Strip tokens from error logs" item.
  } else {
    crashlytics().setUserId('');
  }
}

// Non-fatal: report a handled error with context but keep the app running.
export function captureException(error: Error, context?: Record<string, unknown>) {
  if (context) {
    for (const [key, value] of Object.entries(context)) {
      crashlytics().setAttribute(key, String(value));
    }
  }
  crashlytics().recordError(error);
}

// Breadcrumb-style log attached to the next crash report.
export function logBreadcrumb(message: string) {
  crashlytics().log(message);
}
```

Wire the `ErrorBoundary`'s `componentDidCatch` and the `handleError` utility to
`captureException` so React render crashes and handled errors both reach
Crashlytics as non-fatals.

### App Entry Setup

```typescript
// App.tsx
import { useEffect } from 'react';
import { initCrashReporting } from '@/services/crashReporting';
import { ErrorBoundary } from '@/components/ErrorBoundary';

export default function App() {
  useEffect(() => {
    initCrashReporting();
  }, []);

  return (
    <ErrorBoundary>
      <Providers>
        <Navigation />
      </Providers>
    </ErrorBoundary>
  );
}
```

> Source maps / symbolication: Crashlytics symbolicates native frames from the
> dSYM (iOS) / mapping (Android) the EAS build uploads. For readable JS stack
> traces, upload the Hermes source map to Crashlytics in your build pipeline
> rather than shipping it in the bundle (see the security checklist).
