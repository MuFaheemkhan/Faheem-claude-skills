# Performance Patterns Reference

## Table of Contents
1. [Rendering Optimization](#rendering-optimization)
2. [List Performance](#list-performance)
3. [Image Optimization](#image-optimization)
4. [Animation Performance](#animation-performance)
5. [Memory Management](#memory-management)
6. [Bundle Optimization](#bundle-optimization)
7. [Profiling Tools](#profiling-tools)

---

## Rendering Optimization

### React.memo for Expensive Components

```typescript
// ❌ BAD: Re-renders on every parent render
function UserCard({ user }: { user: User }) {
  return (
    <View>
      <Avatar uri={user.avatar} />
      <Text>{user.name}</Text>
    </View>
  );
}

// ✅ GOOD: Only re-renders when user changes
const UserCard = React.memo(function UserCard({ user }: { user: User }) {
  return (
    <View>
      <Avatar uri={user.avatar} />
      <Text>{user.name}</Text>
    </View>
  );
});

// With custom comparison for complex objects
const UserCard = React.memo(
  function UserCard({ user }: { user: User }) {
    // ...
  },
  (prevProps, nextProps) => {
    return prevProps.user.id === nextProps.user.id &&
           prevProps.user.updatedAt === nextProps.user.updatedAt;
  }
);
```

### useMemo for Expensive Calculations

```typescript
// ❌ BAD: Recalculates on every render
function TransactionList({ transactions }: Props) {
  const sortedTransactions = transactions
    .filter(t => t.amount > 0)
    .sort((a, b) => b.date - a.date);
  
  const totalAmount = sortedTransactions.reduce((sum, t) => sum + t.amount, 0);
  
  return <List data={sortedTransactions} />;
}

// ✅ GOOD: Only recalculates when transactions change
function TransactionList({ transactions }: Props) {
  const sortedTransactions = useMemo(
    () => transactions
      .filter(t => t.amount > 0)
      .sort((a, b) => b.date - a.date),
    [transactions]
  );
  
  const totalAmount = useMemo(
    () => sortedTransactions.reduce((sum, t) => sum + t.amount, 0),
    [sortedTransactions]
  );
  
  return <List data={sortedTransactions} />;
}
```

### useCallback for Event Handlers

```typescript
// ❌ BAD: New function reference on every render
function TodoList({ todos, onToggle }: Props) {
  return (
    <FlatList
      data={todos}
      renderItem={({ item }) => (
        <TodoItem
          todo={item}
          onToggle={() => onToggle(item.id)} // New function every render!
        />
      )}
    />
  );
}

// ✅ GOOD: Stable function reference
function TodoList({ todos, onToggle }: Props) {
  const handleToggle = useCallback(
    (id: string) => onToggle(id),
    [onToggle]
  );
  
  const renderItem = useCallback(
    ({ item }: { item: Todo }) => (
      <TodoItem todo={item} onToggle={handleToggle} />
    ),
    [handleToggle]
  );
  
  return <FlatList data={todos} renderItem={renderItem} />;
}
```

### Avoid Inline Styles and Functions

```typescript
// ❌ BAD: Creates new objects every render
function Card({ title }: Props) {
  return (
    <View style={{ padding: 16, margin: 8 }}>
      <Text style={{ fontSize: 18, fontWeight: 'bold' }}>{title}</Text>
    </View>
  );
}

// ✅ GOOD: Static StyleSheet
const styles = StyleSheet.create({
  container: { padding: 16, margin: 8 },
  title: { fontSize: 18, fontWeight: 'bold' },
});

function Card({ title }: Props) {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>{title}</Text>
    </View>
  );
}
```

---

## List Performance

### FlatList Best Practices

```typescript
function OptimizedList({ data }: { data: Item[] }) {
  // Stable key extractor
  const keyExtractor = useCallback((item: Item) => item.id, []);
  
  // Memoized render function
  const renderItem = useCallback(
    ({ item }: { item: Item }) => <ListItem item={item} />,
    []
  );
  
  // Fixed item height for getItemLayout
  const getItemLayout = useCallback(
    (_: any, index: number) => ({
      length: ITEM_HEIGHT,
      offset: ITEM_HEIGHT * index,
      index,
    }),
    []
  );
  
  return (
    <FlatList
      data={data}
      keyExtractor={keyExtractor}
      renderItem={renderItem}
      getItemLayout={getItemLayout}
      
      // Performance props
      removeClippedSubviews={true}
      maxToRenderPerBatch={10}
      windowSize={5}
      initialNumToRender={10}
      updateCellsBatchingPeriod={50}
      
      // Prevent re-renders
      extraData={undefined} // Only pass if needed
    />
  );
}

const ITEM_HEIGHT = 72;
```

### FlashList for Large Lists

```typescript
import { FlashList } from '@shopify/flash-list';

function LargeList({ data }: { data: Item[] }) {
  const renderItem = useCallback(
    ({ item }: { item: Item }) => <ListItem item={item} />,
    []
  );
  
  return (
    <FlashList
      data={data}
      renderItem={renderItem}
      estimatedItemSize={72}
      
      // FlashList-specific optimizations
      drawDistance={250}
      overrideItemLayout={(layout, item) => {
        layout.size = item.type === 'header' ? 48 : 72;
      }}
    />
  );
}
```

> **Always set `estimatedItemSize`.** Without it FlashList falls back to a slow
> path and the recycling win evaporates — it's the one prop that's easy to omit
> and silently kills the perf gain you switched libraries for.
>
> **Scope caveat (phone-first):** known FlashList rendering gaps on **react-native-web**
> are out of scope for this project — the app is phone-first and the web target
> is only a render smoke harness. Don't burn effort chasing web-only FlashList
> layout bugs; verify on iOS/Android.

### Virtualized Section List

```typescript
function SectionedList({ sections }: { sections: Section[] }) {
  return (
    <SectionList
      sections={sections}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => <ListItem item={item} />}
      renderSectionHeader={({ section }) => (
        <SectionHeader title={section.title} />
      )}
      
      // Performance
      stickySectionHeadersEnabled={true}
      removeClippedSubviews={true}
      maxToRenderPerBatch={10}
      windowSize={5}
      
      // Prevent header re-renders
      renderSectionHeader={useMemo(
        () => ({ section }: { section: Section }) => (
          <SectionHeader title={section.title} />
        ),
        []
      )}
    />
  );
}
```

---

## Image Optimization

### Expo Image (Recommended)

```typescript
import { Image } from 'expo-image';

// Blurhash placeholder for instant loading feel
function OptimizedImage({ uri, blurhash }: Props) {
  return (
    <Image
      source={{ uri }}
      placeholder={blurhash}
      contentFit="cover"
      transition={200}
      style={styles.image}
      
      // Caching
      cachePolicy="memory-disk"
      
      // Recycling for lists
      recyclingKey={uri}
    />
  );
}
```

### Progressive Loading

```typescript
function ProgressiveImage({ thumbnail, fullSize }: Props) {
  const [isLoaded, setIsLoaded] = useState(false);
  
  return (
    <View style={styles.container}>
      {/* Blurred thumbnail */}
      <Image
        source={{ uri: thumbnail }}
        style={[styles.image, styles.thumbnail]}
        blurRadius={10}
      />
      
      {/* Full resolution */}
      <Animated.Image
        source={{ uri: fullSize }}
        style={[styles.image, { opacity: isLoaded ? 1 : 0 }]}
        onLoad={() => setIsLoaded(true)}
      />
    </View>
  );
}
```

### Image Sizing

```typescript
// Always specify dimensions to prevent layout shifts
<Image
  source={{ uri, width: 300, height: 200 }}
  style={{ width: 300, height: 200 }}
/>

// Use aspect ratio for responsive images
<Image
  source={{ uri }}
  style={{ width: '100%', aspectRatio: 16 / 9 }}
  contentFit="cover"
/>
```

---

## Animation Performance

### Reanimated UI Thread Animations

```typescript
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
  runOnUI,
} from 'react-native-reanimated';

// ✅ Runs on UI thread - 60fps guaranteed
function AnimatedCard({ isExpanded }: Props) {
  const height = useSharedValue(100);
  
  useEffect(() => {
    height.value = withSpring(isExpanded ? 300 : 100);
  }, [isExpanded]);
  
  const animatedStyle = useAnimatedStyle(() => ({
    height: height.value,
  }));
  
  return <Animated.View style={animatedStyle} />;
}

// ❌ Animated API runs on JS thread - can drop frames
function SlowAnimatedCard({ isExpanded }: Props) {
  const height = useRef(new Animated.Value(100)).current;
  
  useEffect(() => {
    Animated.spring(height, {
      toValue: isExpanded ? 300 : 100,
      useNativeDriver: false, // Height can't use native driver!
    }).start();
  }, [isExpanded]);
  
  return <Animated.View style={{ height }} />;
}
```

### Gesture-Driven Animations

```typescript
import { Gesture, GestureDetector } from 'react-native-gesture-handler';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
  runOnJS,
} from 'react-native-reanimated';

function SwipeableCard({ onDismiss }: Props) {
  const translateX = useSharedValue(0);
  
  const gesture = Gesture.Pan()
    .onUpdate((event) => {
      translateX.value = event.translationX;
    })
    .onEnd((event) => {
      if (Math.abs(event.translationX) > SWIPE_THRESHOLD) {
        translateX.value = withSpring(
          event.translationX > 0 ? SCREEN_WIDTH : -SCREEN_WIDTH,
          {},
          () => runOnJS(onDismiss)()
        );
      } else {
        translateX.value = withSpring(0);
      }
    });
  
  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ translateX: translateX.value }],
  }));
  
  return (
    <GestureDetector gesture={gesture}>
      <Animated.View style={animatedStyle}>
        {/* Card content */}
      </Animated.View>
    </GestureDetector>
  );
}
```

### Layout Animation Optimization

```typescript
// ❌ Avoid animating layout properties when possible
const badStyle = useAnimatedStyle(() => ({
  width: width.value,      // Triggers layout
  height: height.value,    // Triggers layout
  marginTop: margin.value, // Triggers layout
}));

// ✅ Prefer transform and opacity
const goodStyle = useAnimatedStyle(() => ({
  transform: [
    { translateX: x.value },
    { translateY: y.value },
    { scale: scale.value },
  ],
  opacity: opacity.value,
}));
```

---

## Memory Management

### Cleanup Subscriptions

```typescript
function UserProfile({ userId }: Props) {
  const [user, setUser] = useState<User | null>(null);
  
  useEffect(() => {
    let isMounted = true;
    const controller = new AbortController();
    
    async function fetchUser() {
      try {
        const data = await api.getUser(userId, {
          signal: controller.signal,
        });
        if (isMounted) {
          setUser(data);
        }
      } catch (error) {
        if (!controller.signal.aborted) {
          console.error(error);
        }
      }
    }
    
    fetchUser();
    
    return () => {
      isMounted = false;
      controller.abort();
    };
  }, [userId]);
  
  return <UserCard user={user} />;
}
```

### useFocusEffect for Screen Logic

```typescript
import { useFocusEffect } from '@react-navigation/native';

function HomeScreen() {
  useFocusEffect(
    useCallback(() => {
      // Runs when screen gains focus
      const subscription = eventEmitter.subscribe(handleEvent);
      fetchData();
      
      return () => {
        // Cleanup when screen loses focus
        subscription.unsubscribe();
      };
    }, [])
  );
  
  return <View />;
}
```

### Avoiding Memory Leaks

```typescript
// ❌ Memory leak: Timer not cleared
function BadTimer() {
  useEffect(() => {
    setInterval(() => {
      console.log('tick');
    }, 1000);
  }, []);
}

// ✅ Proper cleanup
function GoodTimer() {
  useEffect(() => {
    const timer = setInterval(() => {
      console.log('tick');
    }, 1000);
    
    return () => clearInterval(timer);
  }, []);
}

// ❌ Memory leak: Event listener not removed
function BadListener() {
  useEffect(() => {
    Keyboard.addListener('keyboardDidShow', handleKeyboard);
  }, []);
}

// ✅ Proper cleanup
function GoodListener() {
  useEffect(() => {
    const subscription = Keyboard.addListener('keyboardDidShow', handleKeyboard);
    return () => subscription.remove();
  }, []);
}
```

---

## Bundle Optimization

### Code Splitting with React.lazy

```typescript
// Lazy load heavy screens
const SettingsScreen = React.lazy(() => import('./screens/SettingsScreen'));
const ProfileScreen = React.lazy(() => import('./screens/ProfileScreen'));

function Navigator() {
  return (
    <Suspense fallback={<LoadingScreen />}>
      <Stack.Navigator>
        <Stack.Screen name="Settings" component={SettingsScreen} />
        <Stack.Screen name="Profile" component={ProfileScreen} />
      </Stack.Navigator>
    </Suspense>
  );
}
```

### Tree Shaking Imports

```typescript
// ❌ BAD: Imports entire library
import _ from 'lodash';
const result = _.debounce(fn, 300);

// ✅ GOOD: Import only what you need
import debounce from 'lodash/debounce';
const result = debounce(fn, 300);

// ❌ BAD: Imports all icons
import * as Icons from 'lucide-react-native';

// ✅ GOOD: Import specific icons
import { Home, Settings, User } from 'lucide-react-native';
```

### Analyzing Bundle Size

```bash
# Generate bundle stats
npx react-native-bundle-visualizer

# Or with Expo
npx expo export --dump-sourcemap
npx source-map-explorer dist/bundles/ios-*.js
```

---

## Server State & Caching

Replace `useState`/`useEffect` fetch chains with TanStack Query (already in the
stack). It gives you request deduplication, stale-while-revalidate caching,
background refetch, and retry-with-backoff for free — boilerplate you'd
otherwise hand-roll and get subtly wrong (race conditions, stale renders,
missing cleanup).

### QueryClient Configuration

```typescript
// app/providers/QueryProvider.tsx
import { QueryClient, QueryClientProvider, focusManager } from '@tanstack/react-query';
import { AppState, Platform } from 'react-native';
import NetInfo from '@react-native-community/netinfo';
import { onlineManager } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60_000,            // 1 min: serve cache, revalidate in background
      gcTime: 5 * 60_000,           // keep unused cache 5 min before GC
      retry: (failureCount, error) => {
        // Don't retry auth/validation; back off on everything else
        if (error instanceof AuthError) return false;
        if (error instanceof ApiError && error.statusCode === 422) return false;
        return failureCount < 3;
      },
      retryDelay: (attempt) => Math.min(1000 * 2 ** attempt, 30_000), // exp backoff
      refetchOnReconnect: true,
    },
  },
});

// Refetch when the app returns to foreground (RN has no window "focus" event)
AppState.addEventListener('change', (status) => {
  if (Platform.OS !== 'web') focusManager.setFocused(status === 'active');
});

// Pause queries when offline; they resume + refetch on reconnect
onlineManager.setEventListener((setOnline) =>
  NetInfo.addEventListener((state) => setOnline(!!state.isConnected))
);

export function QueryProvider({ children }: { children: React.ReactNode }) {
  return <QueryClientProvider client={queryClient}>{children}</QueryClientProvider>;
}
```

See `error-handling.md` → "React Query Error Handling" for the `useApiQuery`
wrapper that layers per-query error toasts on top of this config.

### Offline Persistence (optional)

For read-heavy screens that should survive a cold start offline, persist the
query cache to AsyncStorage with `@tanstack/query-async-storage-persister` +
`PersistQueryClientProvider`. Don't persist auth-sensitive responses — scope
the persister with a `dehydrateOptions.shouldDehydrateQuery` filter.

---

## Perceived Performance (Skeletons)

A skeleton that mirrors the final layout reads as "loading fast"; a centered
spinner reads as "stuck". Prefer skeletons for first-load of content screens.
Reserve spinners for indeterminate inline actions (button submit, pull-to-refresh).

```typescript
// components/feedback/Skeleton.tsx — shimmer placeholder on the UI thread
import { useEffect } from 'react';
import { View, StyleSheet, useColorScheme } from 'react-native';
import Animated, {
  useSharedValue, useAnimatedStyle, withRepeat, withTiming, Easing,
} from 'react-native-reanimated';
import { colors } from '@/theme/colors';

export function Skeleton({ width, height, radius = 8 }: {
  width: number | string; height: number; radius?: number;
}) {
  const scheme = useColorScheme() ?? 'light';
  const opacity = useSharedValue(0.5);

  useEffect(() => {
    opacity.value = withRepeat(
      withTiming(1, { duration: 800, easing: Easing.inOut(Easing.ease) }),
      -1, true,
    );
  }, []);

  const style = useAnimatedStyle(() => ({ opacity: opacity.value }));

  return (
    <Animated.View
      style={[
        { width, height, borderRadius: radius, backgroundColor: colors[scheme].fill },
        style,
      ]}
    />
  );
}

// Usage: compose skeletons into the SAME layout as the loaded state so there's
// no layout shift when real data swaps in.
function TransactionRowSkeleton() {
  return (
    <View style={styles.row}>
      <Skeleton width={40} height={40} radius={20} />
      <View style={styles.col}>
        <Skeleton width="60%" height={16} />
        <Skeleton width="40%" height={12} />
      </View>
      <Skeleton width={64} height={16} />
    </View>
  );
}
```

> Note the `useColorScheme()` here: the shimmer colour resolves per-scheme. If a
> parent pins a brand background, make sure the skeleton fill still has contrast
> in dark mode — same trap the `mobile-theme-parity` skill guards against.

---

## Profiling Tools

### React DevTools Profiler

```typescript
// Wrap components to profile
import { Profiler } from 'react';

function onRenderCallback(
  id: string,
  phase: 'mount' | 'update',
  actualDuration: number,
  baseDuration: number,
  startTime: number,
  commitTime: number
) {
  console.log(`${id} ${phase}: ${actualDuration}ms`);
}

<Profiler id="ExpensiveComponent" onRender={onRenderCallback}>
  <ExpensiveComponent />
</Profiler>
```

### Performance Monitoring Hook

```typescript
function useRenderCount(componentName: string) {
  const renderCount = useRef(0);
  
  useEffect(() => {
    renderCount.current += 1;
    if (__DEV__) {
      console.log(`${componentName} rendered ${renderCount.current} times`);
    }
  });
}

function MyComponent() {
  useRenderCount('MyComponent');
  // ...
}
```

### Flipper Performance Plugin

```typescript
// Enable Flipper in your app
if (__DEV__) {
  import('react-native-flipper').then(({ addPlugin }) => {
    // React DevTools
    // Network Inspector
    // Layout Inspector
    // Performance Monitor
  });
}
```

### Frame Rate Monitoring

```typescript
import { PerformanceObserver } from 'perf_hooks';

// Monitor JS thread frame drops
function useFrameRateMonitor() {
  useEffect(() => {
    if (!__DEV__) return;
    
    let lastTime = performance.now();
    let frameCount = 0;
    
    const measure = () => {
      const now = performance.now();
      frameCount++;
      
      if (now - lastTime >= 1000) {
        console.log(`FPS: ${frameCount}`);
        frameCount = 0;
        lastTime = now;
      }
      
      requestAnimationFrame(measure);
    };
    
    const handle = requestAnimationFrame(measure);
    return () => cancelAnimationFrame(handle);
  }, []);
}
```
