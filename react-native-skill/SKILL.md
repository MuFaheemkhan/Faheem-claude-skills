---
name: react-native-typescript
description: Production-grade React Native mobile development with TypeScript. Use Apple Human Interface Guidelines for premium minimalist UI and a warm, calming sage-green theme with custom typography and animations UI. Use this skill when building mobile apps, React Native components, cross-platform applications, or any mobile frontend work. Triggers include React Native, mobile app, iOS app, Android app, cross-platform, mobile UI, Expo projects, or mobile deployment. Employs a Genius Panel methodology where multiple expert perspectives debate and converge on optimal solutions.
---

# React Native TypeScript Skill

Build production-ready, premium mobile applications with React Native and TypeScript. This skill follows **Apple Human Interface Guidelines** for refined minimalist design with the CalmMoney sage-green theme and employs the **Genius Panel** methodology for optimal decision-making.

**Dependency rule:** never hand-pin Expo package versions. Use `npx expo install <pkg>` so Expo resolves peer-compatible versions for the SDK in use.

## The Genius Panel Methodology

Before implementation, convene a mental panel of experts who debate the problem:

**The Panel Members:**

1. **The UX Designer** - User flows, Apple HIG compliance, accessibility, interaction patterns
2. **The Performance Engineer** - Render optimization, memory management, bundle size, 60fps
3. **The Architect** - Code structure, state management, navigation patterns, modularity
4. **The Platform Specialist** - iOS/Android specifics, native modules, platform parity
5. **The Pragmatist** - Timeline constraints, maintenance burden, team capabilities

**Panel Process:**

1. **Problem Statement** - Define what we're building clearly
2. **Initial Proposals** - Each expert proposes their approach
3. **Debate** - Experts challenge each other's assumptions
4. **Convergence** - Agree on a unified approach with clear rationale
5. **Implementation Plan** - Break into small, testable tasks

Use this format for complex decisions:

```
🏛️ GENIUS PANEL CONVENED: [Topic]

UX DESIGNER: [User experience and design considerations]
PERFORMANCE: [Optimization and efficiency concerns]
ARCHITECT: [Structure and maintainability perspective]
PLATFORM: [iOS/Android specific considerations]
PRAGMATIST: [Trade-offs and practical constraints]

CONSENSUS: [Agreed approach with rationale]
```

## Core Workflow

1. **Understand** → Clarify requirements, identify user needs
2. **Panel Discussion** → Convene experts for complex decisions
3. **Design** → Document component hierarchy and data flow
4. **Decompose** → Split into small tasks (max 2-4 hours each)
5. **Implement** → Code with tests, one component at a time
6. **Polish** → Apply Apple design principles, micro-interactions
7. **Optimize** → Performance profiling, bundle analysis
8. **Document** → Component docs, README, deployment guide

## Task Decomposition Strategy

**NEVER** tackle large features as monoliths. Always decompose:

```
❌ BAD: "Build onboarding flow"

✅ GOOD:
  Task 1: Create OnboardingScreen container with navigation (1h)
  Task 2: Build OnboardingSlide component with animations (2h)
  Task 3: Implement pagination dots with active state (1h)
  Task 4: Add skip/next/done button bar (1h)
  Task 5: Create slide content components (1h)
  Task 6: Implement gesture-based slide navigation (2h)
  Task 7: Add haptic feedback on interactions (30m)
  Task 8: Persist onboarding completion state (1h)
  Task 9: Write component tests (2h)
```

Each task should be:

- **Independently testable** - Can verify it works in isolation
- **Small** - 1-4 hours of focused work
- **Clear** - Obvious when it's "done"
- **Ordered** - Dependencies flow top to bottom

## Project Structure

```
src/
├── app/                      # App entry, providers, navigation
│   ├── App.tsx
│   ├── navigation/
│   │   ├── index.tsx
│   │   ├── RootNavigator.tsx
│   │   ├── MainTabNavigator.tsx
│   │   └── types.ts
│   └── providers/
│       ├── ThemeProvider.tsx
│       └── QueryProvider.tsx
├── components/               # Shared UI components
│   ├── ui/                   # Primitive components
│   │   ├── Button/
│   │   │   ├── Button.tsx
│   │   │   ├── Button.styles.ts
│   │   │   ├── Button.test.tsx
│   │   │   └── index.ts
│   │   ├── Text/
│   │   ├── Input/
│   │   └── Card/
│   ├── forms/                # Form components
│   ├── feedback/             # Toasts, alerts, loaders
│   └── layout/               # Screen wrappers, containers
├── features/                 # Feature modules
│   ├── auth/
│   │   ├── screens/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── services/
│   │   └── types.ts
│   ├── home/
│   └── settings/
├── hooks/                    # Shared custom hooks
│   ├── useDebounce.ts
│   ├── useKeyboard.ts
│   └── useAppState.ts
├── services/                 # API, storage, external services
│   ├── api/
│   │   ├── client.ts
│   │   ├── interceptors.ts
│   │   └── endpoints/
│   ├── storage/
│   └── analytics/
├── store/                    # Global state (Zustand/Redux)
│   ├── useAuthStore.ts
│   └── useSettingsStore.ts
├── theme/                    # Design system
│   ├── index.ts
│   ├── colors.ts
│   ├── typography.ts
│   ├── spacing.ts
│   └── shadows.ts
├── utils/                    # Pure utility functions
│   ├── format.ts
│   ├── validation.ts
│   └── platform.ts
├── constants/                # App-wide constants
│   └── config.ts
└── types/                    # Global TypeScript types
    ├── navigation.ts
    └── api.ts
```

## CalmMoney Design System

### Core Principles

1. **Clarity** - Text is legible, icons precise, adornments subtle (Apple HIG)
2. **Deference** - UI helps understand content, never competes with it
3. **Depth** - Visual layers and motion convey hierarchy
4. **Calm & Premium** - Sage-green palette with warm neutrals for a "Serene Wealth" aesthetic

### Design Tokens

> Note: the CalmMoney project already ships these tokens at `mobile/src/theme/`.
> Prefer importing from that module instead of re-declaring — it's the source of truth.
> The block below is illustrative and should stay in sync with the theme module.

```typescript
// theme/colors.ts
export const colors = {
  // Brand colors - CalmMoney Sage Palette
  brand: {
    primary: '#8BA888',      // Calm sage
    primaryLight: '#A8C4A5', // Calm sage light
    primaryDark: '#6B8B69',  // Calm sage dark
    primaryDeep: '#4A6B48',  // Deep forest sage
    mint: '#E8F0E8',         // Calm mint (light green bg)
    cream: '#FAF9F7',        // Calm cream
    warm: '#F5F0EB',         // Calm warm
    // Accent colors
    gold: '#D4A574',         // Warm gold accent
    goldLight: '#E8D4B8',    // Light gold
    coral: '#E8A598',        // Soft coral
    lavender: '#B8A8C8',     // Soft lavender
  },

  // Transaction colors
  transaction: {
    income: '#4CAF7A',       // Custom income green
    incomeBg: '#E8F5EE',     // Income background
    expense: '#E07A5F',      // Coral expense
    expenseBg: '#FCEEE9',    // Expense background
  },

  // Budget status colors
  budget: {
    safe: '#8BA888',         // Sage (under 50%)
    warning: '#F2A93B',      // Amber (50-80%)
    danger: '#E07A5F',       // Coral (over 80%)
    exceeded: '#C44536',     // Deep coral (over 100%)
  },

  // Light mode
  light: {
    background: '#FFFFFF',
    backgroundSecondary: '#F8FAF8',   // Subtle green tint
    backgroundTertiary: '#F0F4F0',
    surface: '#FFFFFF',
    surfaceElevated: '#FFFFFF',
    text: '#1A1D1A',
    textSecondary: '#5A5F5A',
    textTertiary: '#8A8F8A',
    textInverse: '#FFFFFF',
    separator: '#E5E8E5',
    separatorOpaque: '#D0D5D0',
    fill: 'rgba(139, 168, 136, 0.2)',        // Sage-tinted
    fillSecondary: 'rgba(139, 168, 136, 0.16)',
    fillTertiary: 'rgba(139, 168, 136, 0.12)',
  },

  // Dark mode
  dark: {
    background: '#0D1210',   // Dark with green tint
    backgroundSecondary: '#1A201A',
    backgroundTertiary: '#242A24',
    surface: '#1A201A',
    surfaceElevated: '#242A24',
    text: '#FFFFFF',
    textSecondary: 'rgba(255, 255, 255, 0.7)',
    textTertiary: 'rgba(255, 255, 255, 0.4)',
    textInverse: '#1A1D1A',
    separator: 'rgba(139, 168, 136, 0.3)',
    separatorOpaque: '#3A403A',
    fill: 'rgba(139, 168, 136, 0.36)',
    fillSecondary: 'rgba(139, 168, 136, 0.28)',
    fillTertiary: 'rgba(139, 168, 136, 0.20)',
  },

  // Gradient presets
  gradients: {
    primary: ['#8BA888', '#6B8B69'],
    header: ['#6B8B69', '#4A6B48', '#3A5A38'],
    headerOrb1: ['#A8C4A5', '#8BA888'],    // Floating orb gradient
    headerOrb2: ['#D4A574', '#B8956A'],    // Gold orb gradient
    card: ['rgba(255,255,255,0.18)', 'rgba(255,255,255,0.08)'], // Glass effect
    cardBorder: ['rgba(255,255,255,0.3)', 'rgba(255,255,255,0.1)'],
    income: ['#4CAF7A', '#3A9A6A'],
    expense: ['#E07A5F', '#C86A4F'],
  },

  // Semantic colors
  semantic: {
    success: '#4CAF7A',
    warning: '#F2A93B',
    error: '#E07A5F',
    info: '#8BA888',
  },
} as const;

// theme/typography.ts
// Dual-font system: Plus Jakarta Sans (body) + Fraunces (display)
export const fontFamilies = {
  // Plus Jakarta Sans - Modern, clean body text
  body: 'PlusJakartaSans-Regular',
  bodyMedium: 'PlusJakartaSans-Medium',
  bodySemiBold: 'PlusJakartaSans-SemiBold',
  bodyBold: 'PlusJakartaSans-Bold',
  // Fraunces - Elegant serif for headlines
  display: 'Fraunces-Regular',
  displayMedium: 'Fraunces-Medium',
  displaySemiBold: 'Fraunces-SemiBold',
  displayBold: 'Fraunces-Bold',
} as const;

export const typography = {
  // Large titles - Fraunces serif for premium feel
  largeTitle: {
    fontFamily: fontFamilies.displayBold,
    fontSize: 34, lineHeight: 42, fontWeight: '700', letterSpacing: 0.25,
  },
  title1: {
    fontFamily: fontFamilies.displayBold,
    fontSize: 28, lineHeight: 36, fontWeight: '700', letterSpacing: 0,
  },
  title2: {
    fontFamily: fontFamilies.displaySemiBold,
    fontSize: 22, lineHeight: 30, fontWeight: '600', letterSpacing: 0,
  },
  title3: {
    fontFamily: fontFamilies.displaySemiBold,
    fontSize: 20, lineHeight: 28, fontWeight: '600', letterSpacing: 0,
  },
  // Body text - Plus Jakarta Sans
  headline: {
    fontFamily: fontFamilies.bodySemiBold,
    fontSize: 17, lineHeight: 24, fontWeight: '600', letterSpacing: -0.2,
  },
  body: {
    fontFamily: fontFamilies.body,
    fontSize: 16, lineHeight: 24, fontWeight: '400', letterSpacing: -0.1,
  },
  bodyMedium: {
    fontFamily: fontFamilies.bodyMedium,
    fontSize: 16, lineHeight: 24, fontWeight: '500', letterSpacing: -0.1,
  },
  callout: {
    fontFamily: fontFamilies.body,
    fontSize: 16, lineHeight: 22, fontWeight: '400', letterSpacing: -0.2,
  },
  subheadline: {
    fontFamily: fontFamilies.bodyMedium,
    fontSize: 15, lineHeight: 22, fontWeight: '500', letterSpacing: -0.1,
  },
  footnote: {
    fontFamily: fontFamilies.body,
    fontSize: 13, lineHeight: 18, fontWeight: '400', letterSpacing: 0,
  },
  caption1: {
    fontFamily: fontFamilies.body,
    fontSize: 12, lineHeight: 16, fontWeight: '400', letterSpacing: 0.1,
  },
  caption2: {
    fontFamily: fontFamilies.body,
    fontSize: 11, lineHeight: 14, fontWeight: '400', letterSpacing: 0.2,
  },
  // Special variants
  currency: {
    fontFamily: fontFamilies.displayBold,
    fontSize: 42, lineHeight: 50, fontWeight: '700', letterSpacing: -0.5,
  },
  currencySmall: {
    fontFamily: fontFamilies.displaySemiBold,
    fontSize: 28, lineHeight: 36, fontWeight: '600', letterSpacing: -0.3,
  },
  button: {
    fontFamily: fontFamilies.bodySemiBold,
    fontSize: 16, lineHeight: 22, fontWeight: '600', letterSpacing: 0.2,
  },
  buttonSmall: {
    fontFamily: fontFamilies.bodyMedium,
    fontSize: 14, lineHeight: 20, fontWeight: '500', letterSpacing: 0.1,
  },
  label: {
    fontFamily: fontFamilies.bodyMedium,
    fontSize: 14, lineHeight: 20, fontWeight: '500', letterSpacing: 0.1,
  },
  input: {
    fontFamily: fontFamilies.body,
    fontSize: 16, lineHeight: 22, fontWeight: '400', letterSpacing: 0,
  },
} as const;

// theme/spacing.ts
export const spacing = {
  // Base scale (4px grid)
  xxs: 2,
  xs: 4,
  sm: 8,
  md: 16,
  lg: 24,
  xl: 32,
  xxl: 48,
  xxxl: 64,

  // Screen spacing
  screenHorizontal: 20,
  screenVertical: 24,

  // Cards & Containers
  cardPadding: 16,
  cardPaddingLarge: 20,
  containerPadding: 16,
  containerPaddingLarge: 24,

  // Lists
  listItemPadding: 16,
  listItemVertical: 14,
  listItemGap: 12,

  // Sections
  sectionGap: 32,
  sectionGapLarge: 48,
  componentGap: 16,
  componentGapSmall: 12,

  // Forms
  formGap: 20,
  inputGap: 8,
  labelGap: 6,

  // Touch targets (Apple HIG minimum)
  inputHeight: 52,
  buttonHeight: 52,
  buttonHeightSmall: 44,
  touchTarget: 44,

  // Icons
  iconSize: 24,
  iconSizeSmall: 20,
  iconSizeLarge: 32,
  iconSizeXLarge: 48,

  // Border radius scale
  radiusXs: 4,
  radiusSm: 8,
  radiusMd: 12,
  radiusLg: 16,
  radiusXl: 20,
  radiusXxl: 24,
  radiusFull: 9999,
} as const;

// theme/shadows.ts
import { Platform } from 'react-native';

export const shadows = {
  none: {
    shadowColor: 'transparent',
    shadowOffset: { width: 0, height: 0 },
    shadowOpacity: 0, shadowRadius: 0, elevation: 0,
  },
  xs: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.04, shadowRadius: 1, elevation: 1,
  },
  sm: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.05, shadowRadius: 2, elevation: 2,
  },
  md: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.08, shadowRadius: 8, elevation: 3,
  },
  lg: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.12, shadowRadius: 16, elevation: 6,
  },
  xl: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 8 },
    shadowOpacity: 0.16, shadowRadius: 24, elevation: 12,
  },
  // Card-specific (platform-aware)
  card: Platform.select({
    ios: {
      shadowColor: '#000',
      shadowOffset: { width: 0, height: 2 },
      shadowOpacity: 0.06, shadowRadius: 8,
    },
    android: { elevation: 2 },
  }),
  buttonPressed: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.02, shadowRadius: 1, elevation: 1,
  },
} as const;
```

### Animation System

```typescript
// theme/animations.ts
// Spring configurations for natural motion
export const springConfigs = {
  gentle: { damping: 20, stiffness: 200, mass: 1 },    // Cards, modals
  snappy: { damping: 15, stiffness: 400, mass: 0.8 },  // Buttons, quick interactions
  bouncy: { damping: 12, stiffness: 180, mass: 1 },    // Playful elements
  smooth: { damping: 25, stiffness: 150, mass: 1 },    // Subtle movements
} as const;

// Timing configurations
export const timingConfigs = {
  fast: { duration: 200, easing: Easing.bezier(0.25, 0.1, 0.25, 1) },
  normal: { duration: 300, easing: Easing.bezier(0.25, 0.1, 0.25, 1) },
  slow: { duration: 500, easing: Easing.bezier(0.25, 0.1, 0.25, 1) },
  easeOut: { duration: 400, easing: Easing.bezier(0.0, 0.0, 0.2, 1) },  // Entrances
  easeIn: { duration: 300, easing: Easing.bezier(0.4, 0.0, 1, 1) },     // Exits
} as const;

// Stagger delays for list animations
export const staggerDelays = {
  fast: 50,
  normal: 80,
  slow: 120,
} as const;

// Animation helpers
export const animations = {
  fadeInUp: (delay = 0) => ({
    entering: () => {
      'worklet';
      return {
        initialValues: { opacity: 0, transform: [{ translateY: 20 }] },
        animations: {
          opacity: withDelay(delay, withTiming(1, timingConfigs.easeOut)),
          transform: [{ translateY: withDelay(delay, withSpring(0, springConfigs.gentle)) }],
        },
      };
    },
  }),
  scaleIn: (delay = 0) => ({ /* scale from 0.9 to 1 with snappy spring */ }),
  slideInRight: (delay = 0) => ({ /* slide from translateX: 30 */ }),
  pressScale: (scale, pressed) => {
    'worklet';
    scale.value = pressed
      ? withSpring(0.97, springConfigs.snappy)
      : withSpring(1, springConfigs.snappy);
  },
  shimmer: (progress) => { /* 1000ms shimmer for loading states */ },
  float: (translateY, amplitude = 8) => { /* 2s sine wave for decorative elements */ },
  pulse: (scale) => { /* 1.05 scale pulse for attention */ },
} as const;

// Scroll-based interpolations
export const interpolations = {
  scrollOpacity: (scrollY, start, end) => { /* fade based on scroll position */ },
  parallax: (scrollY, factor = 0.5) => { /* parallax movement */ },
} as const;
```

### UI Component Patterns

**Premium Button with Haptic Feedback:**

```typescript
// components/ui/Button/Button.tsx
import { ActivityIndicator, Pressable, useColorScheme } from 'react-native';
import Animated, {
  useSharedValue, useAnimatedStyle, withSpring,
} from 'react-native-reanimated';
import * as Haptics from 'expo-haptics';
import { colors } from '@/theme/colors';
import { Text } from '@/components/ui/Text';

const AnimatedPressable = Animated.createAnimatedComponent(Pressable);

export function Button({ label, onPress, variant = 'primary', size = 'md', disabled, loading }) {
  const colorScheme = useColorScheme() ?? 'light';
  const scale = useSharedValue(1);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));

  const handlePressIn = () => {
    if (!disabled && !loading) {
      Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light);
      scale.value = withSpring(0.97, { damping: 15, stiffness: 400 });
    }
  };

  const handlePressOut = () => {
    scale.value = withSpring(1, { damping: 15, stiffness: 400 });
  };

  return (
    <AnimatedPressable
      onPress={onPress}
      onPressIn={handlePressIn}
      onPressOut={handlePressOut}
      disabled={disabled || loading}
      style={[styles.button, animatedStyle]}
    >
      {loading ? (
        <ActivityIndicator color={variant === 'primary' ? '#FFFFFF' : colors.brand.primary} />
      ) : (
        <Text variant="headline" style={styles.label}>{label}</Text>
      )}
    </AnimatedPressable>
  );
}

// Variant styles: primary (sage bg), secondary (fill bg), ghost (transparent), danger (coral bg)
// Size styles: sm (44px), md (44px touch target), lg (52px)
```

**Card with Press Animation:**

```typescript
// components/ui/Card/Card.tsx
import { Pressable, useColorScheme } from 'react-native';
import Animated, { useSharedValue, useAnimatedStyle, withSpring } from 'react-native-reanimated';
import * as Haptics from 'expo-haptics';
import { spacing } from '@/theme/spacing';
import { shadows } from '@/theme/shadows';

const AnimatedPressable = Animated.createAnimatedComponent(Pressable);

export function Card({ children, onPress, variant = 'default', padding = 'md' }) {
  const colorScheme = useColorScheme() ?? 'light';
  const scale = useSharedValue(1);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));

  const handlePressIn = () => {
    if (onPress) {
      Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light);
      scale.value = withSpring(0.98, { damping: 15, stiffness: 400 });
    }
  };

  // Variants: default (surface + card shadow), elevated (md shadow), outlined (border only)
  // Padding: none, sm (8), md (16), lg (24)
  return (
    <AnimatedPressable
      onPress={onPress}
      onPressIn={handlePressIn}
      onPressOut={() => scale.value = withSpring(1)}
      style={[styles.card, { borderRadius: spacing.radiusLg }, shadows.card, animatedStyle]}
    >
      {children}
    </AnimatedPressable>
  );
}
```

**FloatingTabBar with FAB:**

```typescript
// components/ui/FloatingTabBar/FloatingTabBar.tsx
// Custom bottom navigation with:
// - 4 tabs: Home, Budgets, History, More
// - Center FAB for quick "Add Transaction"
// - "More" menu as bottom sheet with Goals, Deals, Settings
// - Press animations with haptic feedback
// - Platform-aware shadows

<View style={styles.container}>
  {/* Left tabs */}
  <View style={styles.tabGroup}>{leftTabs}</View>

  {/* Center FAB */}
  <Pressable style={styles.fab} onPress={handleAddPress}>
    <PlusIcon />  {/* Sage green, elevated shadow */}
  </Pressable>

  {/* Right tabs */}
  <View style={styles.tabGroup}>{rightTabs}</View>
</View>

// FAB: 52x52, borderRadius: 26, sage green background, elevated shadow
// Tab items: minWidth 64, icon + caption2 label
```

**Glass Effect Cards (for Dashboard Header):**

```typescript
// Gradient background with translucent card overlay
<LinearGradient colors={colors.gradients.header} style={styles.header}>
  {/* Floating decorative orbs */}
  <Animated.View style={[styles.orb, floatingStyle]}>
    <LinearGradient colors={colors.gradients.headerOrb1} />
  </Animated.View>

  {/* Glass card */}
  <View style={[styles.glassCard, {
    backgroundColor: 'rgba(255,255,255,0.15)',
    borderWidth: 1,
    borderColor: 'rgba(255,255,255,0.2)',
    borderRadius: spacing.radiusXl,
  }]}>
    <Text variant="currency" color="#FFFFFF">{balance}</Text>
  </View>
</LinearGradient>
```

See `references/apple-design-patterns.md` for complete component library.

## Code Quality Standards

### TypeScript Strictness

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "exactOptionalPropertyTypes": true
  }
}
```

### Component Typing

```typescript
// Always type props explicitly
interface ButtonProps {
  label: string;
  onPress: () => void;
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  loading?: boolean;
  icon?: React.ReactNode;
  testID?: string;
}

// Use discriminated unions for complex states
type AsyncState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: Error };
```

### Error Handling

```typescript
// services/api/client.ts
class ApiError extends Error {
  constructor(
    message: string,
    public statusCode: number,
    public code?: string,
    public details?: unknown
  ) {
    super(message);
    this.name = 'ApiError';
  }
}

// Global error boundary
class ErrorBoundary extends Component<Props, State> {
  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    logger.error('Uncaught error', { error, errorInfo });
    analytics.trackError(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <ErrorFallback error={this.state.error} />;
    }
    return this.props.children;
  }
}
```

See `references/error-handling.md` for complete patterns.

### Logging

```typescript
// services/logger.ts
type LogLevel = 'debug' | 'info' | 'warn' | 'error';

interface LogContext {
  screen?: string;
  component?: string;
  action?: string;
  userId?: string;
  [key: string]: unknown;
}

const logger = {
  debug: (message: string, context?: LogContext) => log('debug', message, context),
  info: (message: string, context?: LogContext) => log('info', message, context),
  warn: (message: string, context?: LogContext) => log('warn', message, context),
  error: (message: string, context?: LogContext) => log('error', message, context),
};

// Usage
logger.info('User signed in', { userId: user.id, method: 'email' });
logger.error('Payment failed', { orderId, error: error.message });
```

See `references/logging-patterns.md` for complete setup.

## Mobile Security Checklist

Mobile apps are not browsers — the binary lives on a device the user (or attacker) controls.
Apply to every feature that touches auth, PII, or money.

**Secret / token storage:**

- [ ] **Never store JWTs or refresh tokens in AsyncStorage** — it's plaintext on disk
- [ ] Use `expo-secure-store` (iOS Keychain / Android Keystore) for tokens, session IDs, encryption keys
- [ ] Rotate refresh tokens on use and invalidate the old one server-side
- [ ] Wipe secrets on logout — a stale token outlives the user's intent

```typescript
// services/storage/secureStorage.ts
import * as SecureStore from 'expo-secure-store';

export const secureStorage = {
  async setToken(key: 'access' | 'refresh', value: string) {
    await SecureStore.setItemAsync(key, value, {
      keychainAccessible: SecureStore.WHEN_UNLOCKED_THIS_DEVICE_ONLY,
    });
  },
  getToken: (key: 'access' | 'refresh') => SecureStore.getItemAsync(key),
  clearAll: async () => {
    await Promise.all([
      SecureStore.deleteItemAsync('access'),
      SecureStore.deleteItemAsync('refresh'),
    ]);
  },
};
```

**Network:**

- [ ] HTTPS only — reject `http://` in production builds
- [ ] Consider certificate pinning for high-value APIs (`react-native-ssl-pinning` or native config)
- [ ] Strip tokens/PII from error logs (don't pass them as Crashlytics attributes) and analytics events

**Deep links & URL schemes:**

- [ ] Validate every deep-link parameter — treat them as untrusted user input
- [ ] Use Universal Links / App Links (not just custom schemes) to prevent scheme hijacking
- [ ] Don't navigate to untrusted URLs from deep-link params

**Device integrity (for high-value actions):**

- [ ] App Attest (iOS) / Play Integrity (Android) before sensitive operations
- [ ] Detect jailbreak/root if the threat model requires it (`jail-monkey`)
- [ ] Require biometric re-auth (`expo-local-authentication`) for sensitive operations

**Build hardening:**

- [ ] Hermes engine (default in RN 0.70+) — provides bytecode, harder to reverse
- [ ] Strip source maps from production bundles; upload the Hermes source map to Crashlytics separately for readable JS stack traces
- [ ] No dev/test API keys in `app.json` extra — use EAS Secrets
- [ ] `android:allowBackup="false"` to prevent ADB-based data extraction

**PII handling:**

- [ ] Mark sensitive text inputs `secureTextEntry` and `textContentType="password"`
- [ ] Exclude sensitive screens from iOS app switcher preview with blur overlay
- [ ] Clear clipboard after copying account numbers/OTP

## Performance Checklist

Apply to EVERY screen and component:

**Rendering:**

- [ ] Use `React.memo()` for expensive components
- [ ] Implement `useMemo` for expensive calculations
- [ ] Use `useCallback` for event handlers passed as props
- [ ] Avoid inline styles and functions in render
- [ ] Use `keyExtractor` properly in FlatList

**Lists:**

- [ ] Use `FlatList` or `FlashList`, never `ScrollView` for lists
- [ ] Always set `estimatedItemSize` on FlashList — omitting it silently drops it to the slow path
- [ ] Implement `getItemLayout` for fixed-height items
- [ ] Set `removeClippedSubviews={true}` for long lists
- [ ] Use `windowSize` and `maxToRenderPerBatch` appropriately
- [ ] Phone-first: don't chase FlashList **web-only** layout gaps; verify on iOS/Android

**Server state & loading:**

- [ ] Fetch via TanStack Query, not `useState`/`useEffect` chains (dedup, SWR cache, retry/backoff for free) — see `references/performance-patterns.md` → "Server State & Caching"
- [ ] Show layout-matched skeletons on first load; reserve spinners for inline actions

**Images:**

- [ ] Use `expo-image` or `FastImage` for caching
- [ ] Specify dimensions to prevent layout shifts
- [ ] Use appropriate resolution for device
- [ ] Implement progressive loading for large images

**Animations:**

- [ ] Use Reanimated for 60fps animations
- [ ] Run animations on UI thread with `useAnimatedStyle`
- [ ] Use `withSpring` / `withTiming` for natural motion
- [ ] Avoid animating layout properties when possible

**Memory:**

- [ ] Clean up subscriptions in `useEffect` cleanup
- [ ] Cancel pending requests on unmount
- [ ] Use `useFocusEffect` for screen-specific logic
- [ ] Profile with Flipper for memory leaks

See `references/performance-patterns.md` for detailed implementations.

## Technology Stack

**Core:**

- React Native 0.73+ / Expo SDK 50+
- TypeScript 5.0+
- React Navigation 6+

**State Management:**

- Zustand (simple global state)
- TanStack Query (server state)
- React Hook Form (forms)

**UI & Animation:**

- Reanimated 3 (animations)
- Gesture Handler (gestures)
- expo-haptics (haptic feedback)
- expo-blur (blur effects)

**Development:**

- ESLint + Prettier
- Jest + Testing Library
- **React Native DevTools** (React Native 0.76+) for network, React tree, perf profiling. Flipper was removed from RN core in 0.73; don't recommend it for new projects. Reactotron is a solid third-party alternative.

See `references/stack-configs.md` for recommended configurations.

## Deployment Readiness

### Environment Configuration

```typescript
// config/env.ts
const ENV = {
  development: {
    apiUrl: 'http://localhost:8000',
    enableLogs: true,
  },
  staging: {
    apiUrl: 'https://staging-api.example.com',
    enableLogs: true,
  },
  production: {
    apiUrl: 'https://api.example.com',
    enableLogs: false,
  },
};
```

### App Store Checklist

- [ ] App icons (all sizes)
- [ ] Splash screen
- [ ] Privacy policy URL
- [ ] App Store screenshots
- [ ] Accessibility labels
- [ ] Deep linking configured
- [ ] Push notifications setup
- [ ] Analytics integration
- [ ] Crash reporting (Firebase Crashlytics — needs a dev/EAS build, not Expo Go)
- [ ] OTA updates (EAS Update)

See `references/deployment-guide.md` for complete checklist.

## Example: Panel Discussion for Navigation

```
🏛️ GENIUS PANEL CONVENED: App Navigation Structure

UX DESIGNER: "Users expect tab-based navigation for primary 
features. Home, Search, Profile as core tabs. Settings should 
be accessible from Profile, not a separate tab."

PERFORMANCE: "Lazy load tab screens. Use native stack navigator 
for 60fps transitions. Preload adjacent screens for instant feels."

ARCHITECT: "Type-safe navigation with discriminated unions. 
Centralize route params. Use navigation service for deep links 
and analytics tracking."

PLATFORM: "Use native tab bar on iOS, material bottom tabs match 
Android conventions. Handle safe areas and keyboard consistently."

PRAGMATIST: "Start with 3 tabs maximum. Add more only when user 
research demands it. Drawer nav adds complexity - avoid unless 
necessary."

CONSENSUS:
- 3-tab bottom navigation (Home, Search, Profile)
- Native stack within each tab for drill-down
- Settings as modal from Profile header
- Type-safe route definitions
- Lazy loading with pre-fetching
```

## References

- `references/apple-design-patterns.md` - Complete Apple HIG component library
- `references/performance-patterns.md` - Optimization techniques and profiling
- `references/error-handling.md` - Error boundaries, API errors, logging

_Planned (not yet written): `logging-patterns.md`, `stack-configs.md`, `deployment-guide.md`. Until they exist, see Expo docs and the project's `mobile/src/services/` and `eas.json` for current patterns._

## Quick Commands

```bash
# Create new Expo project
npx create-expo-app@latest myapp -t expo-template-blank-typescript

# Install core dependencies
npx expo install react-native-reanimated react-native-gesture-handler
npx expo install @react-navigation/native @react-navigation/native-stack
npx expo install expo-haptics expo-blur expo-image

# Development
npx expo start

# Run tests
npm test

# Build for production
eas build --platform ios
eas build --platform android

# Submit to stores
eas submit --platform ios
eas submit --platform android
```
