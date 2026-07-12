# React Native TypeScript Skill - Installation & Usage

## Installation for Claude Code

```bash
# Navigate to your project
cd /path/to/your/project

# Create skills directory
mkdir -p .claude/skills

# Extract the skill
unzip react-native-skill.skill -d .claude/skills/
```

**Structure after extraction:**
```
.claude/skills/react-native-skill/
├── SKILL.md                              # Main skill instructions
└── references/
    ├── apple-design-patterns.md          # Apple HIG components & design system
    ├── performance-patterns.md           # Optimization & profiling
    └── error-handling.md                 # Error boundaries, logging, crash reporting
```

---

## What's Included

### Core Features
- **Genius Panel Methodology** - 5 experts debate before implementation (UX Designer, Performance Engineer, Architect, Platform Specialist, Pragmatist)
- **Task Decomposition** - Breaks large features into 1-4 hour testable tasks
- **Apple Human Interface Guidelines** - Premium minimalist design system

### Reference Files

| File | Contents |
|------|----------|
| `apple-design-patterns.md` | Typography scale, color system, Button/Card/Input/ListItem components, animations, accessibility |
| `performance-patterns.md` | React.memo, useMemo, useCallback, FlatList/FlashList optimization, image caching, Reanimated patterns, memory management, TanStack Query caching config, skeleton loaders |
| `error-handling.md` | Error types, ErrorBoundary components, API error handling, React Query integration, structured logging, Firebase Crashlytics setup |

---

## Usage Examples

### Start a New App
```
Create a React Native app for tracking personal finances with 
Apple-style design
```

### Add a Feature
```
Add an onboarding flow with smooth animations following Apple HIG
```

### Request Panel Discussion
```
Convene the genius panel to discuss navigation architecture for 
my e-commerce app
```

### Performance Review
```
Review my FlatList implementation and optimize for 60fps scrolling
```

---

## Technology Stack (Defaults)

- **Framework:** React Native 0.73+ / Expo SDK 50+
- **Language:** TypeScript 5.0+ (strict mode)
- **Navigation:** React Navigation 6+
- **State:** Zustand + TanStack Query
- **Animation:** Reanimated 3 + Gesture Handler
- **Forms:** React Hook Form + Zod
- **Testing:** Jest + Testing Library

---

## Design System

The skill includes the CalmMoney "Serene Wealth" design system:

**Typography:** Dual-font system
- **Plus Jakarta Sans** - Modern sans-serif for body text
- **Fraunces** - Elegant serif for headlines and currency displays
- Special variants: currency, button, label, input

**Colors:** Calming sage-green palette
- Brand: Sage primary (#8BA888), gold accent (#D4A574)
- Transaction: Income green (#4CAF7A), Expense coral (#E07A5F)
- Budget status: Safe/Warning/Danger/Exceeded
- Full light/dark mode support with sage-tinted backgrounds

**Spacing:** Extended 4px grid (xxs:2 to xxxl:64)
- Semantic spacing for screens, cards, lists, forms
- Border radius scale (radiusXs:4 to radiusFull:9999)

**Animations:** Reanimated-based system
- Spring configs: gentle, snappy, bouncy, smooth
- Timing configs: fast (200ms), normal (300ms), slow (500ms)
- Helpers: fadeInUp, scaleIn, pressScale, shimmer, float, pulse

**Shadows:** 5-level system (xs, sm, md, lg, xl) with platform-aware handling

**Components:** Button, Card, FloatingTabBar with haptic feedback and press animations

---

## The Genius Panel

For complex decisions, Claude simulates expert discussion:

```
🏛️ GENIUS PANEL CONVENED: Authentication Flow

UX DESIGNER: "Biometric first, fallback to PIN. Minimal friction..."
PERFORMANCE: "Cache auth state, lazy load profile data..."
ARCHITECT: "Separate auth module, token refresh middleware..."
PLATFORM: "Face ID on iOS, Fingerprint on Android, handle permissions..."
PRAGMATIST: "Start with email/password, add biometrics in v1.1..."

CONSENSUS: [Unified approach with clear rationale]
```

---

## Quick Reference

### Project Structure
```
src/
├── app/                 # Entry, navigation, providers
├── components/ui/       # Design system primitives
├── features/            # Feature modules (auth, home, etc.)
├── hooks/               # Shared custom hooks
├── services/            # API, storage, analytics
├── store/               # Global state (Zustand)
├── theme/               # Design tokens
├── utils/               # Pure utilities
└── types/               # Global TypeScript types
```

### Performance Checklist
- [ ] React.memo for expensive components
- [ ] useMemo/useCallback appropriately
- [ ] FlatList with keyExtractor + getItemLayout
- [ ] Reanimated for 60fps animations
- [ ] expo-image for caching
- [ ] Cleanup in useEffect

### CalmMoney Design Checklist
- [ ] 44-52pt touch targets (Apple HIG)
- [ ] Plus Jakarta Sans (body) + Fraunces (display) fonts
- [ ] Sage-green brand palette with light/dark mode
- [ ] Platform-aware shadows (iOS shadowColor, Android elevation)
- [ ] Haptic feedback on interactions (expo-haptics)
- [ ] Reanimated spring animations (snappy: damping 15, stiffness 400)
- [ ] Gradient headers with floating orbs
- [ ] FloatingTabBar with center FAB
