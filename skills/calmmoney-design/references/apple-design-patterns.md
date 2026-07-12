# Apple Design Patterns Reference

## Table of Contents
1. [Design Principles](#design-principles)
2. [Typography System](#typography-system)
3. [Color System](#color-system)
4. [Component Library](#component-library)
5. [Animation Patterns](#animation-patterns)
6. [Accessibility](#accessibility)

---

## Design Principles

### Apple Human Interface Guidelines - Core

**Clarity**
- Legible text at every size
- Precise, lucid icons
- Subtle adornments
- Focus on functionality
- Negative space creates focus

**Deference**
- Content is the star
- UI recedes to support content
- Minimal chrome and bezels
- Full-screen layouts
- Translucency hints at more

**Depth**
- Distinct visual layers
- Realistic motion
- Clear hierarchy
- Touch conveys depth

### The Premium Feel

What makes an app feel "Apple-quality":

1. **Restraint** - Every element earns its place
2. **Consistency** - Patterns repeat predictably
3. **Responsiveness** - Instant feedback on every touch
4. **Polish** - Micro-interactions delight
5. **Whitespace** - Generous breathing room

---

## Typography System

### SF Pro Type Scale

```typescript
// theme/typography.ts
import { Platform, TextStyle } from 'react-native';

const fontFamily = Platform.select({
  ios: 'System',
  android: 'Roboto',
});

export const typography = {
  // Display
  largeTitle: {
    fontFamily,
    fontSize: 34,
    lineHeight: 41,
    fontWeight: '700',
    letterSpacing: 0.37,
  } as TextStyle,
  
  // Titles
  title1: {
    fontFamily,
    fontSize: 28,
    lineHeight: 34,
    fontWeight: '700',
    letterSpacing: 0.36,
  } as TextStyle,
  
  title2: {
    fontFamily,
    fontSize: 22,
    lineHeight: 28,
    fontWeight: '700',
    letterSpacing: 0.35,
  } as TextStyle,
  
  title3: {
    fontFamily,
    fontSize: 20,
    lineHeight: 25,
    fontWeight: '600',
    letterSpacing: 0.38,
  } as TextStyle,
  
  // Body
  headline: {
    fontFamily,
    fontSize: 17,
    lineHeight: 22,
    fontWeight: '600',
    letterSpacing: -0.41,
  } as TextStyle,
  
  body: {
    fontFamily,
    fontSize: 17,
    lineHeight: 22,
    fontWeight: '400',
    letterSpacing: -0.41,
  } as TextStyle,
  
  callout: {
    fontFamily,
    fontSize: 16,
    lineHeight: 21,
    fontWeight: '400',
    letterSpacing: -0.32,
  } as TextStyle,
  
  subheadline: {
    fontFamily,
    fontSize: 15,
    lineHeight: 20,
    fontWeight: '400',
    letterSpacing: -0.24,
  } as TextStyle,
  
  // Small
  footnote: {
    fontFamily,
    fontSize: 13,
    lineHeight: 18,
    fontWeight: '400',
    letterSpacing: -0.08,
  } as TextStyle,
  
  caption1: {
    fontFamily,
    fontSize: 12,
    lineHeight: 16,
    fontWeight: '400',
    letterSpacing: 0,
  } as TextStyle,
  
  caption2: {
    fontFamily,
    fontSize: 11,
    lineHeight: 13,
    fontWeight: '400',
    letterSpacing: 0.07,
  } as TextStyle,
} as const;
```

### Text Component

```typescript
// components/ui/Text/Text.tsx
import React from 'react';
import { Text as RNText, TextProps as RNTextProps, StyleSheet } from 'react-native';
import { typography } from '@/theme/typography';
import { colors } from '@/theme/colors';

type TextVariant = keyof typeof typography;
type TextColor = 'primary' | 'secondary' | 'tertiary' | 'accent' | 'error';

interface TextProps extends RNTextProps {
  variant?: TextVariant;
  color?: TextColor;
  align?: 'left' | 'center' | 'right';
  children: React.ReactNode;
}

const colorMap: Record<TextColor, string> = {
  primary: colors.textPrimary,
  secondary: colors.textSecondary,
  tertiary: colors.textTertiary,
  accent: colors.primary,
  error: colors.error,
};

export function Text({
  variant = 'body',
  color = 'primary',
  align = 'left',
  style,
  children,
  ...props
}: TextProps) {
  return (
    <RNText
      style={[
        typography[variant],
        { color: colorMap[color], textAlign: align },
        style,
      ]}
      {...props}
    >
      {children}
    </RNText>
  );
}
```

---

## Color System

### Light & Dark Mode

```typescript
// theme/colors.ts
export const lightColors = {
  // Backgrounds
  background: '#FFFFFF',
  backgroundSecondary: '#F2F2F7',
  backgroundTertiary: '#FFFFFF',
  backgroundElevated: '#FFFFFF',
  
  // Grouped backgrounds (for forms, settings)
  backgroundGrouped: '#F2F2F7',
  backgroundGroupedSecondary: '#FFFFFF',
  backgroundGroupedTertiary: '#F2F2F7',
  
  // Labels (text)
  label: '#000000',
  labelSecondary: 'rgba(60, 60, 67, 0.6)',
  labelTertiary: 'rgba(60, 60, 67, 0.3)',
  labelQuaternary: 'rgba(60, 60, 67, 0.18)',
  
  // Fills
  fill: 'rgba(120, 120, 128, 0.2)',
  fillSecondary: 'rgba(120, 120, 128, 0.16)',
  fillTertiary: 'rgba(118, 118, 128, 0.12)',
  fillQuaternary: 'rgba(116, 116, 128, 0.08)',
  
  // Separators
  separator: 'rgba(60, 60, 67, 0.36)',
  separatorOpaque: '#C6C6C8',
  
  // System colors
  systemBlue: '#007AFF',
  systemGreen: '#34C759',
  systemIndigo: '#5856D6',
  systemOrange: '#FF9500',
  systemPink: '#FF2D55',
  systemPurple: '#AF52DE',
  systemRed: '#FF3B30',
  systemTeal: '#5AC8FA',
  systemYellow: '#FFCC00',
} as const;

export const darkColors = {
  // Backgrounds
  background: '#000000',
  backgroundSecondary: '#1C1C1E',
  backgroundTertiary: '#2C2C2E',
  backgroundElevated: '#1C1C1E',
  
  // Grouped backgrounds
  backgroundGrouped: '#000000',
  backgroundGroupedSecondary: '#1C1C1E',
  backgroundGroupedTertiary: '#2C2C2E',
  
  // Labels
  label: '#FFFFFF',
  labelSecondary: 'rgba(235, 235, 245, 0.6)',
  labelTertiary: 'rgba(235, 235, 245, 0.3)',
  labelQuaternary: 'rgba(235, 235, 245, 0.18)',
  
  // Fills
  fill: 'rgba(120, 120, 128, 0.36)',
  fillSecondary: 'rgba(120, 120, 128, 0.32)',
  fillTertiary: 'rgba(118, 118, 128, 0.24)',
  fillQuaternary: 'rgba(116, 116, 128, 0.18)',
  
  // Separators
  separator: 'rgba(84, 84, 88, 0.65)',
  separatorOpaque: '#38383A',
  
  // System colors (adjusted for dark mode)
  systemBlue: '#0A84FF',
  systemGreen: '#30D158',
  systemIndigo: '#5E5CE6',
  systemOrange: '#FF9F0A',
  systemPink: '#FF375F',
  systemPurple: '#BF5AF2',
  systemRed: '#FF453A',
  systemTeal: '#64D2FF',
  systemYellow: '#FFD60A',
} as const;

// Theme hook
export function useColors() {
  const colorScheme = useColorScheme();
  return colorScheme === 'dark' ? darkColors : lightColors;
}
```

---

## Component Library

### Button

```typescript
// components/ui/Button/Button.tsx
import React from 'react';
import { Pressable, StyleSheet, ActivityIndicator } from 'react-native';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
} from 'react-native-reanimated';
import * as Haptics from 'expo-haptics';
import { Text } from '../Text';
import { useColors } from '@/theme/colors';

interface ButtonProps {
  label: string;
  onPress: () => void;
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'sm' | 'md' | 'lg';
  disabled?: boolean;
  loading?: boolean;
  fullWidth?: boolean;
  testID?: string;
}

const AnimatedPressable = Animated.createAnimatedComponent(Pressable);

export function Button({
  label,
  onPress,
  variant = 'primary',
  size = 'md',
  disabled = false,
  loading = false,
  fullWidth = false,
  testID,
}: ButtonProps) {
  const colors = useColors();
  const scale = useSharedValue(1);
  const opacity = useSharedValue(1);

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
    opacity: opacity.value,
  }));

  const handlePressIn = () => {
    if (disabled || loading) return;
    Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light);
    scale.value = withSpring(0.97, { damping: 15, stiffness: 400 });
    opacity.value = withSpring(0.9);
  };

  const handlePressOut = () => {
    scale.value = withSpring(1, { damping: 15, stiffness: 400 });
    opacity.value = withSpring(1);
  };

  const sizeStyles = {
    sm: { paddingVertical: 8, paddingHorizontal: 16, minHeight: 36 },
    md: { paddingVertical: 12, paddingHorizontal: 24, minHeight: 44 },
    lg: { paddingVertical: 16, paddingHorizontal: 32, minHeight: 52 },
  };

  const variantStyles = {
    primary: {
      backgroundColor: colors.systemBlue,
      borderWidth: 0,
    },
    secondary: {
      backgroundColor: colors.fillSecondary,
      borderWidth: 0,
    },
    ghost: {
      backgroundColor: 'transparent',
      borderWidth: 0,
    },
  };

  const textColor = variant === 'primary' ? '#FFFFFF' : colors.systemBlue;

  return (
    <AnimatedPressable
      onPress={onPress}
      onPressIn={handlePressIn}
      onPressOut={handlePressOut}
      disabled={disabled || loading}
      testID={testID}
      style={[
        styles.button,
        sizeStyles[size],
        variantStyles[variant],
        fullWidth && styles.fullWidth,
        disabled && styles.disabled,
        animatedStyle,
      ]}
    >
      {loading ? (
        <ActivityIndicator color={textColor} size="small" />
      ) : (
        <Text
          variant={size === 'sm' ? 'subheadline' : 'headline'}
          style={{ color: textColor }}
        >
          {label}
        </Text>
      )}
    </AnimatedPressable>
  );
}

const styles = StyleSheet.create({
  button: {
    borderRadius: 12,
    alignItems: 'center',
    justifyContent: 'center',
    flexDirection: 'row',
  },
  fullWidth: {
    width: '100%',
  },
  disabled: {
    opacity: 0.5,
  },
});
```

### Card

```typescript
// components/ui/Card/Card.tsx
import React from 'react';
import { View, StyleSheet, ViewStyle } from 'react-native';
import { BlurView } from 'expo-blur';
import { useColors } from '@/theme/colors';
import { shadows } from '@/theme/shadows';

interface CardProps {
  children: React.ReactNode;
  variant?: 'elevated' | 'filled' | 'glass';
  padding?: 'none' | 'sm' | 'md' | 'lg';
  style?: ViewStyle;
}

export function Card({
  children,
  variant = 'elevated',
  padding = 'md',
  style,
}: CardProps) {
  const colors = useColors();

  const paddingValues = {
    none: 0,
    sm: 12,
    md: 16,
    lg: 24,
  };

  if (variant === 'glass') {
    return (
      <View style={[styles.card, shadows.md, style]}>
        <BlurView intensity={80} tint="light" style={styles.blur}>
          <View style={{ padding: paddingValues[padding] }}>
            {children}
          </View>
        </BlurView>
      </View>
    );
  }

  return (
    <View
      style={[
        styles.card,
        variant === 'elevated' && shadows.md,
        {
          backgroundColor:
            variant === 'elevated'
              ? colors.backgroundElevated
              : colors.fillTertiary,
          padding: paddingValues[padding],
        },
        style,
      ]}
    >
      {children}
    </View>
  );
}

const styles = StyleSheet.create({
  card: {
    borderRadius: 16,
    overflow: 'hidden',
  },
  blur: {
    overflow: 'hidden',
  },
});
```

### Input

```typescript
// components/ui/Input/Input.tsx
import React, { useState } from 'react';
import {
  TextInput,
  View,
  StyleSheet,
  TextInputProps,
  Pressable,
} from 'react-native';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withTiming,
} from 'react-native-reanimated';
import { Text } from '../Text';
import { useColors } from '@/theme/colors';
import { typography } from '@/theme/typography';

interface InputProps extends Omit<TextInputProps, 'style'> {
  label?: string;
  error?: string;
  hint?: string;
  leftIcon?: React.ReactNode;
  rightIcon?: React.ReactNode;
}

export function Input({
  label,
  error,
  hint,
  leftIcon,
  rightIcon,
  onFocus,
  onBlur,
  ...props
}: InputProps) {
  const colors = useColors();
  const [isFocused, setIsFocused] = useState(false);
  const borderColor = useSharedValue(colors.separator);

  const animatedBorderStyle = useAnimatedStyle(() => ({
    borderColor: borderColor.value,
  }));

  const handleFocus = (e: any) => {
    setIsFocused(true);
    borderColor.value = withTiming(colors.systemBlue, { duration: 150 });
    onFocus?.(e);
  };

  const handleBlur = (e: any) => {
    setIsFocused(false);
    borderColor.value = withTiming(
      error ? colors.systemRed : colors.separator,
      { duration: 150 }
    );
    onBlur?.(e);
  };

  return (
    <View style={styles.container}>
      {label && (
        <Text variant="subheadline" color="secondary" style={styles.label}>
          {label}
        </Text>
      )}
      
      <Animated.View
        style={[
          styles.inputContainer,
          { backgroundColor: colors.fillTertiary },
          animatedBorderStyle,
          error && { borderColor: colors.systemRed },
        ]}
      >
        {leftIcon && <View style={styles.icon}>{leftIcon}</View>}
        
        <TextInput
          style={[
            styles.input,
            typography.body,
            { color: colors.label },
          ]}
          placeholderTextColor={colors.labelTertiary}
          onFocus={handleFocus}
          onBlur={handleBlur}
          {...props}
        />
        
        {rightIcon && <View style={styles.icon}>{rightIcon}</View>}
      </Animated.View>
      
      {(error || hint) && (
        <Text
          variant="caption1"
          color={error ? 'error' : 'tertiary'}
          style={styles.helper}
        >
          {error || hint}
        </Text>
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    gap: 6,
  },
  label: {
    marginLeft: 4,
  },
  inputContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    borderRadius: 12,
    borderWidth: 1,
    minHeight: 48,
    paddingHorizontal: 16,
  },
  input: {
    flex: 1,
    paddingVertical: 12,
  },
  icon: {
    marginRight: 12,
  },
  helper: {
    marginLeft: 4,
  },
});
```

### List Item

```typescript
// components/ui/ListItem/ListItem.tsx
import React from 'react';
import { Pressable, View, StyleSheet } from 'react-native';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withTiming,
} from 'react-native-reanimated';
import * as Haptics from 'expo-haptics';
import { ChevronRight } from 'lucide-react-native';
import { Text } from '../Text';
import { useColors } from '@/theme/colors';

interface ListItemProps {
  title: string;
  subtitle?: string;
  leftIcon?: React.ReactNode;
  rightElement?: React.ReactNode;
  showChevron?: boolean;
  onPress?: () => void;
  destructive?: boolean;
}

const AnimatedPressable = Animated.createAnimatedComponent(Pressable);

export function ListItem({
  title,
  subtitle,
  leftIcon,
  rightElement,
  showChevron = false,
  onPress,
  destructive = false,
}: ListItemProps) {
  const colors = useColors();
  const backgroundColor = useSharedValue('transparent');

  const animatedStyle = useAnimatedStyle(() => ({
    backgroundColor: backgroundColor.value,
  }));

  const handlePressIn = () => {
    if (!onPress) return;
    Haptics.selectionAsync();
    backgroundColor.value = withTiming(colors.fillSecondary, { duration: 100 });
  };

  const handlePressOut = () => {
    backgroundColor.value = withTiming('transparent', { duration: 200 });
  };

  return (
    <AnimatedPressable
      onPress={onPress}
      onPressIn={handlePressIn}
      onPressOut={handlePressOut}
      disabled={!onPress}
      style={[styles.container, animatedStyle]}
    >
      {leftIcon && <View style={styles.leftIcon}>{leftIcon}</View>}
      
      <View style={styles.content}>
        <Text
          variant="body"
          style={destructive && { color: colors.systemRed }}
        >
          {title}
        </Text>
        {subtitle && (
          <Text variant="subheadline" color="secondary">
            {subtitle}
          </Text>
        )}
      </View>
      
      {rightElement}
      
      {showChevron && onPress && (
        <ChevronRight
          size={20}
          color={colors.labelTertiary}
          style={styles.chevron}
        />
      )}
    </AnimatedPressable>
  );
}

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    alignItems: 'center',
    paddingVertical: 12,
    paddingHorizontal: 16,
    minHeight: 44,
  },
  leftIcon: {
    marginRight: 16,
  },
  content: {
    flex: 1,
    gap: 2,
  },
  chevron: {
    marginLeft: 8,
  },
});
```

---

## Animation Patterns

### Spring Configurations

```typescript
// theme/animations.ts
export const springConfigs = {
  // Gentle - for subtle movements
  gentle: {
    damping: 20,
    stiffness: 150,
  },
  // Default - balanced feel
  default: {
    damping: 15,
    stiffness: 300,
  },
  // Snappy - for quick responses
  snappy: {
    damping: 20,
    stiffness: 400,
  },
  // Bouncy - playful
  bouncy: {
    damping: 10,
    stiffness: 200,
  },
} as const;

// Timing configurations
export const timingConfigs = {
  fast: { duration: 150 },
  normal: { duration: 250 },
  slow: { duration: 400 },
} as const;
```

### Common Animation Patterns

```typescript
// Fade In on Mount
export function FadeIn({ children, delay = 0 }: FadeInProps) {
  const opacity = useSharedValue(0);
  const translateY = useSharedValue(10);

  useEffect(() => {
    opacity.value = withDelay(delay, withTiming(1, { duration: 300 }));
    translateY.value = withDelay(delay, withSpring(0, springConfigs.gentle));
  }, []);

  const animatedStyle = useAnimatedStyle(() => ({
    opacity: opacity.value,
    transform: [{ translateY: translateY.value }],
  }));

  return <Animated.View style={animatedStyle}>{children}</Animated.View>;
}

// Scale on Press
export function useScalePress() {
  const scale = useSharedValue(1);

  const onPressIn = () => {
    scale.value = withSpring(0.95, springConfigs.snappy);
  };

  const onPressOut = () => {
    scale.value = withSpring(1, springConfigs.snappy);
  };

  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));

  return { onPressIn, onPressOut, animatedStyle };
}

// Staggered List Animation
export function StaggeredList({ items, renderItem }: StaggeredListProps) {
  return (
    <>
      {items.map((item, index) => (
        <FadeIn key={item.id} delay={index * 50}>
          {renderItem(item)}
        </FadeIn>
      ))}
    </>
  );
}
```

---

## Accessibility

### Minimum Standards

```typescript
// Every interactive element needs:
<Pressable
  accessible={true}
  accessibilityRole="button"
  accessibilityLabel="Add item to cart"
  accessibilityHint="Double tap to add this item to your shopping cart"
  accessibilityState={{ disabled: isDisabled }}
  style={{ minHeight: 44, minWidth: 44 }} // Minimum touch target
>
  {/* content */}
</Pressable>

// Images need descriptions
<Image
  source={source}
  accessible={true}
  accessibilityLabel="Product photo showing blue sneakers"
/>

// Group related elements
<View
  accessible={true}
  accessibilityRole="summary"
  accessibilityLabel={`Order total: $${total}. ${itemCount} items.`}
>
  <Text>Total: ${total}</Text>
  <Text>{itemCount} items</Text>
</View>
```

### Dynamic Type Support

```typescript
// Use relative font sizes that scale with system settings
import { useWindowDimensions } from 'react-native';

export function useScaledFontSize(baseSize: number) {
  const { fontScale } = useWindowDimensions();
  return baseSize * Math.min(fontScale, 1.3); // Cap at 130%
}
```
