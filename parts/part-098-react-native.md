# Part 98: Mobile Apps ด้วย React Native (Steps 1931-1950)

## บทนำ

React Native คือ framework ที่ช่วยให้เราสร้าง mobile application สำหรับทั้ง iOS และ Android ด้วย JavaScript และ React โดยที่ UI ที่ได้จะเป็น native components จริงๆ ไม่ใช่ WebView ทำให้ประสิทธิภาพใกล้เคียงกับ native app

---

## Step 1931: React Native Overview

### React Native vs Expo

```
┌─────────────────────────────────────────────────┐
│              React Native Ecosystem             │
├─────────────────────────┬───────────────────────┤
│     Bare React Native   │         Expo           │
│                         │                        │
│ - เข้าถึง native code  │ - Setup ง่ายกว่า       │
│ - ควบคุมได้เต็มที่     │ - Managed workflow     │
│ - ต้องการ Xcode/AS     │ - Expo Go app          │
│ - Build เอง            │ - OTA updates          │
│                         │ - SDK ครอบคลุม         │
└─────────────────────────┴───────────────────────┘
```

### การทำงานของ React Native

```
JavaScript Thread         Native Thread
      │                        │
      │ ── JSBridge ──────────▶│ Native Modules
      │                        │ (Camera, Location, etc.)
      │                        │
      │ ◀── Shadow Thread ─────│ Layout Calculation
      │                        │ (Yoga)
      │                        │
                               │ Native UI Components
                               │ (View → UIView/android.view.View)
```

---

## Step 1932: การติดตั้ง Expo

### Setup Expo Project

```bash
# ติดตั้ง Expo CLI
npm install -g expo-cli

# สร้างโปรเจคใหม่
npx create-expo-app my-app
cd my-app

# หรือ template ที่เฉพาะเจาะจง
npx create-expo-app my-app --template blank
npx create-expo-app my-app --template blank-typescript
npx create-expo-app my-app --template tabs
npx create-expo-app my-app --template bare-minimum

# เริ่มรัน
npx expo start

# รันบน simulator
npx expo start --ios      # iOS Simulator
npx expo start --android  # Android Emulator
npx expo start --web      # Web browser
```

### โครงสร้างโปรเจค Expo

```
my-app/
├── app/                  # Expo Router (file-based routing)
│   ├── _layout.tsx
│   ├── index.tsx
│   └── (tabs)/
│       ├── _layout.tsx
│       ├── index.tsx
│       └── explore.tsx
├── components/
│   ├── ui/
│   └── themed/
├── constants/
│   └── Colors.ts
├── hooks/
├── assets/
│   ├── fonts/
│   └── images/
├── app.json              # Expo configuration
└── package.json
```

### app.json

```json
{
  "expo": {
    "name": "My App",
    "slug": "my-app",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "light",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#ffffff"
    },
    "updates": {
      "fallbackToCacheTimeout": 0
    },
    "assetBundlePatterns": ["**/*"],
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.company.myapp",
      "buildNumber": "1"
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#FFFFFF"
      },
      "package": "com.company.myapp",
      "versionCode": 1,
      "permissions": [
        "CAMERA",
        "ACCESS_FINE_LOCATION",
        "READ_EXTERNAL_STORAGE",
        "WRITE_EXTERNAL_STORAGE"
      ]
    },
    "web": {
      "favicon": "./assets/favicon.png"
    },
    "plugins": [
      "expo-camera",
      "expo-location",
      [
        "expo-notifications",
        {
          "icon": "./assets/notification-icon.png",
          "color": "#ffffff"
        }
      ]
    ]
  }
}
```

---

## Step 1933: Core Components

### View, Text, Image

```typescript
// App.tsx - Core Components
import React from 'react';
import {
  View,
  Text,
  Image,
  ImageBackground,
  ScrollView,
  SafeAreaView,
  StatusBar,
  StyleSheet,
  Platform
} from 'react-native';

export default function App() {
  return (
    <SafeAreaView style={styles.safeArea}>
      <StatusBar
        barStyle="dark-content"
        backgroundColor="#ffffff"
        translucent={false}
      />
      
      <ScrollView
        style={styles.container}
        contentContainerStyle={styles.contentContainer}
        showsVerticalScrollIndicator={false}
        bounces={true}
      >
        {/* Header */}
        <View style={styles.header}>
          <Image
            source={require('./assets/logo.png')}
            style={styles.logo}
            resizeMode="contain"
          />
          <Text style={styles.headerTitle}>React Native App</Text>
        </View>
        
        {/* Card */}
        <View style={styles.card}>
          <ImageBackground
            source={{ uri: 'https://picsum.photos/400/200' }}
            style={styles.cardImage}
            imageStyle={styles.cardImageStyle}
          >
            <View style={styles.overlay}>
              <Text style={styles.cardTitle}>Welcome</Text>
            </View>
          </ImageBackground>
          
          <View style={styles.cardContent}>
            <Text style={styles.cardText}>
              ยินดีต้อนรับสู่ React Native
            </Text>
            <Text style={styles.cardSubtext} numberOfLines={2}>
              สร้าง mobile app ด้วย JavaScript สำหรับทั้ง iOS และ Android
            </Text>
          </View>
        </View>
        
        {/* Platform-specific */}
        <View style={styles.platformInfo}>
          <Text style={styles.infoText}>
            Platform: {Platform.OS}
          </Text>
          <Text style={styles.infoText}>
            Version: {Platform.Version}
          </Text>
          {Platform.OS === 'ios' && (
            <Text style={styles.infoText}>
              Model: {Platform.constants.systemName}
            </Text>
          )}
        </View>
      </ScrollView>
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  safeArea: {
    flex: 1,
    backgroundColor: '#f5f5f5'
  },
  container: {
    flex: 1
  },
  contentContainer: {
    padding: 16
  },
  header: {
    flexDirection: 'row',
    alignItems: 'center',
    marginBottom: 20,
    padding: 16,
    backgroundColor: '#fff',
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3 // Android
  },
  logo: {
    width: 40,
    height: 40,
    marginRight: 12
  },
  headerTitle: {
    fontSize: 20,
    fontWeight: '700',
    color: '#1a1a1a'
  },
  card: {
    backgroundColor: '#fff',
    borderRadius: 16,
    overflow: 'hidden',
    marginBottom: 16,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.1,
    shadowRadius: 8,
    elevation: 5
  },
  cardImage: {
    height: 180,
    justifyContent: 'flex-end'
  },
  cardImageStyle: {
    borderTopLeftRadius: 16,
    borderTopRightRadius: 16
  },
  overlay: {
    padding: 16,
    backgroundColor: 'rgba(0,0,0,0.4)'
  },
  cardTitle: {
    color: '#fff',
    fontSize: 24,
    fontWeight: '700'
  },
  cardContent: {
    padding: 16
  },
  cardText: {
    fontSize: 16,
    fontWeight: '600',
    color: '#1a1a1a',
    marginBottom: 8
  },
  cardSubtext: {
    fontSize: 14,
    color: '#666',
    lineHeight: 20
  },
  platformInfo: {
    backgroundColor: '#fff',
    borderRadius: 12,
    padding: 16
  },
  infoText: {
    fontSize: 14,
    color: '#333',
    marginBottom: 4
  }
});
```

---

## Step 1934: FlatList และ ScrollView

### Lists ใน React Native

```typescript
// FlatList สำหรับ large lists (efficient)
import React, { useState, useCallback } from 'react';
import {
  FlatList,
  SectionList,
  VirtualizedList,
  View,
  Text,
  TouchableOpacity,
  RefreshControl,
  ActivityIndicator,
  StyleSheet
} from 'react-native';

interface Item {
  id: string;
  title: string;
  subtitle: string;
  image: string;
}

function ItemCard({ item, onPress }: { item: Item; onPress: () => void }) {
  return (
    <TouchableOpacity style={styles.itemCard} onPress={onPress} activeOpacity={0.7}>
      <View style={styles.itemContent}>
        <Text style={styles.itemTitle}>{item.title}</Text>
        <Text style={styles.itemSubtitle}>{item.subtitle}</Text>
      </View>
    </TouchableOpacity>
  );
}

function ProductList() {
  const [data, setData] = useState<Item[]>([]);
  const [loading, setLoading] = useState(false);
  const [refreshing, setRefreshing] = useState(false);
  const [page, setPage] = useState(1);
  const [hasMore, setHasMore] = useState(true);

  const fetchData = useCallback(async (pageNum: number, isRefresh = false) => {
    if (loading && !isRefresh) return;

    setLoading(true);
    try {
      const response = await fetch(
        `https://api.example.com/items?page=${pageNum}&limit=20`
      );
      const result = await response.json();

      if (isRefresh) {
        setData(result.items);
      } else {
        setData(prev => [...prev, ...result.items]);
      }

      setHasMore(result.items.length === 20);
      setPage(pageNum);
    } catch (error) {
      console.error(error);
    } finally {
      setLoading(false);
      setRefreshing(false);
    }
  }, [loading]);

  const handleRefresh = useCallback(() => {
    setRefreshing(true);
    fetchData(1, true);
  }, [fetchData]);

  const handleLoadMore = useCallback(() => {
    if (!loading && hasMore) {
      fetchData(page + 1);
    }
  }, [loading, hasMore, page, fetchData]);

  const renderItem = useCallback(({ item, index }: { item: Item; index: number }) => (
    <ItemCard
      item={item}
      onPress={() => console.log('Pressed:', item.id)}
    />
  ), []);

  const keyExtractor = useCallback((item: Item) => item.id, []);

  const renderFooter = () => {
    if (!loading) return null;
    return (
      <View style={styles.footer}>
        <ActivityIndicator size="small" color="#007AFF" />
      </View>
    );
  };

  const renderEmpty = () => (
    <View style={styles.empty}>
      <Text style={styles.emptyText}>ไม่พบข้อมูล</Text>
    </View>
  );

  const renderSeparator = () => (
    <View style={styles.separator} />
  );

  return (
    <FlatList
      data={data}
      renderItem={renderItem}
      keyExtractor={keyExtractor}
      
      // Performance
      removeClippedSubviews={true}
      maxToRenderPerBatch={10}
      windowSize={10}
      initialNumToRender={10}
      updateCellsBatchingPeriod={50}
      
      // Pull to refresh
      refreshControl={
        <RefreshControl
          refreshing={refreshing}
          onRefresh={handleRefresh}
          tintColor="#007AFF"
          colors={['#007AFF']} // Android
        />
      }
      
      // Infinite scroll
      onEndReached={handleLoadMore}
      onEndReachedThreshold={0.5}
      
      // Decorators
      ListFooterComponent={renderFooter}
      ListEmptyComponent={renderEmpty}
      ItemSeparatorComponent={renderSeparator}
      
      // Layout
      contentContainerStyle={{ padding: 16, flexGrow: 1 }}
      showsVerticalScrollIndicator={false}
    />
  );
}

// SectionList สำหรับ grouped data
function ContactsList() {
  const sections = [
    {
      title: 'A',
      data: [
        { id: '1', name: 'Alice' },
        { id: '2', name: 'Andy' }
      ]
    },
    {
      title: 'B',
      data: [
        { id: '3', name: 'Bob' },
        { id: '4', name: 'Betty' }
      ]
    }
  ];

  return (
    <SectionList
      sections={sections}
      keyExtractor={(item) => item.id}
      renderItem={({ item }) => (
        <View style={styles.contactItem}>
          <Text>{item.name}</Text>
        </View>
      )}
      renderSectionHeader={({ section: { title } }) => (
        <View style={styles.sectionHeader}>
          <Text style={styles.sectionTitle}>{title}</Text>
        </View>
      )}
      stickySectionHeadersEnabled={true}
    />
  );
}

const styles = StyleSheet.create({
  itemCard: {
    backgroundColor: '#fff',
    borderRadius: 12,
    padding: 16,
    marginBottom: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 1 },
    shadowOpacity: 0.05,
    shadowRadius: 4,
    elevation: 2
  },
  itemContent: {},
  itemTitle: {
    fontSize: 16,
    fontWeight: '600',
    color: '#1a1a1a',
    marginBottom: 4
  },
  itemSubtitle: {
    fontSize: 14,
    color: '#666'
  },
  footer: {
    padding: 20,
    alignItems: 'center'
  },
  empty: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    padding: 40
  },
  emptyText: {
    fontSize: 16,
    color: '#999'
  },
  separator: {
    height: 1,
    backgroundColor: '#f0f0f0',
    marginHorizontal: 16
  },
  contactItem: {
    padding: 16,
    backgroundColor: '#fff'
  },
  sectionHeader: {
    backgroundColor: '#f5f5f5',
    padding: 8,
    paddingHorizontal: 16
  },
  sectionTitle: {
    fontWeight: '700',
    color: '#666'
  }
});
```

---

## Step 1935: TextInput และ Forms

### การจัดการ Forms

```typescript
import React, { useState, useRef } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  KeyboardAvoidingView,
  Platform,
  ScrollView,
  Keyboard,
  StyleSheet,
  Alert
} from 'react-native';

interface FormData {
  name: string;
  email: string;
  password: string;
  phone: string;
  bio: string;
}

function SignUpForm() {
  const [form, setForm] = useState<FormData>({
    name: '',
    email: '',
    password: '',
    phone: '',
    bio: ''
  });
  const [errors, setErrors] = useState<Partial<FormData>>({});
  const [showPassword, setShowPassword] = useState(false);
  const [loading, setLoading] = useState(false);

  // Refs สำหรับ focus ถัดไป
  const emailRef = useRef<TextInput>(null);
  const passwordRef = useRef<TextInput>(null);
  const phoneRef = useRef<TextInput>(null);
  const bioRef = useRef<TextInput>(null);

  const validate = () => {
    const newErrors: Partial<FormData> = {};

    if (!form.name.trim()) {
      newErrors.name = 'กรุณากรอกชื่อ';
    } else if (form.name.length < 2) {
      newErrors.name = 'ชื่อต้องมีอย่างน้อย 2 ตัวอักษร';
    }

    if (!form.email.trim()) {
      newErrors.email = 'กรุณากรอกอีเมล';
    } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.email)) {
      newErrors.email = 'รูปแบบอีเมลไม่ถูกต้อง';
    }

    if (!form.password) {
      newErrors.password = 'กรุณากรอกรหัสผ่าน';
    } else if (form.password.length < 8) {
      newErrors.password = 'รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร';
    }

    if (form.phone && !/^0[0-9]{9}$/.test(form.phone)) {
      newErrors.phone = 'รูปแบบเบอร์โทรไม่ถูกต้อง';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = async () => {
    Keyboard.dismiss();

    if (!validate()) return;

    setLoading(true);
    try {
      const response = await fetch('https://api.example.com/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(form)
      });

      const result = await response.json();

      if (response.ok) {
        Alert.alert('สำเร็จ', 'สมัครสมาชิกเรียบร้อยแล้ว');
      } else {
        Alert.alert('ผิดพลาด', result.message);
      }
    } catch (error) {
      Alert.alert('ผิดพลาด', 'ไม่สามารถเชื่อมต่อเซิร์ฟเวอร์ได้');
    } finally {
      setLoading(false);
    }
  };

  return (
    <KeyboardAvoidingView
      style={{ flex: 1 }}
      behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
    >
      <ScrollView
        contentContainerStyle={styles.container}
        keyboardShouldPersistTaps="handled"
      >
        <Text style={styles.title}>สมัครสมาชิก</Text>

        <View style={styles.field}>
          <Text style={styles.label}>ชื่อ *</Text>
          <TextInput
            style={[styles.input, errors.name && styles.inputError]}
            placeholder="กรอกชื่อ-นามสกุล"
            value={form.name}
            onChangeText={(text) => {
              setForm(prev => ({ ...prev, name: text }));
              if (errors.name) setErrors(prev => ({ ...prev, name: undefined }));
            }}
            returnKeyType="next"
            onSubmitEditing={() => emailRef.current?.focus()}
            autoCapitalize="words"
            autoComplete="name"
          />
          {errors.name && <Text style={styles.error}>{errors.name}</Text>}
        </View>

        <View style={styles.field}>
          <Text style={styles.label}>อีเมล *</Text>
          <TextInput
            ref={emailRef}
            style={[styles.input, errors.email && styles.inputError]}
            placeholder="example@email.com"
            value={form.email}
            onChangeText={(text) => {
              setForm(prev => ({ ...prev, email: text }));
              if (errors.email) setErrors(prev => ({ ...prev, email: undefined }));
            }}
            returnKeyType="next"
            onSubmitEditing={() => passwordRef.current?.focus()}
            keyboardType="email-address"
            autoCapitalize="none"
            autoComplete="email"
          />
          {errors.email && <Text style={styles.error}>{errors.email}</Text>}
        </View>

        <View style={styles.field}>
          <Text style={styles.label}>รหัสผ่าน *</Text>
          <View style={styles.passwordContainer}>
            <TextInput
              ref={passwordRef}
              style={[styles.passwordInput, errors.password && styles.inputError]}
              placeholder="รหัสผ่าน (อย่างน้อย 8 ตัวอักษร)"
              value={form.password}
              onChangeText={(text) => {
                setForm(prev => ({ ...prev, password: text }));
                if (errors.password) setErrors(prev => ({ ...prev, password: undefined }));
              }}
              returnKeyType="next"
              onSubmitEditing={() => phoneRef.current?.focus()}
              secureTextEntry={!showPassword}
              autoCapitalize="none"
              autoComplete="new-password"
            />
            <TouchableOpacity
              style={styles.eyeButton}
              onPress={() => setShowPassword(!showPassword)}
            >
              <Text>{showPassword ? '🙈' : '👁️'}</Text>
            </TouchableOpacity>
          </View>
          {errors.password && <Text style={styles.error}>{errors.password}</Text>}
        </View>

        <View style={styles.field}>
          <Text style={styles.label}>เบอร์โทรศัพท์</Text>
          <TextInput
            ref={phoneRef}
            style={[styles.input, errors.phone && styles.inputError]}
            placeholder="0812345678"
            value={form.phone}
            onChangeText={(text) => setForm(prev => ({ ...prev, phone: text }))}
            returnKeyType="next"
            onSubmitEditing={() => bioRef.current?.focus()}
            keyboardType="phone-pad"
            maxLength={10}
          />
          {errors.phone && <Text style={styles.error}>{errors.phone}</Text>}
        </View>

        <View style={styles.field}>
          <Text style={styles.label}>แนะนำตัว</Text>
          <TextInput
            ref={bioRef}
            style={[styles.input, styles.textArea]}
            placeholder="บอกเล่าเกี่ยวกับตัวเอง..."
            value={form.bio}
            onChangeText={(text) => setForm(prev => ({ ...prev, bio: text }))}
            multiline={true}
            numberOfLines={4}
            maxLength={200}
            textAlignVertical="top"
          />
          <Text style={styles.charCount}>{form.bio.length}/200</Text>
        </View>

        <TouchableOpacity
          style={[styles.button, loading && styles.buttonDisabled]}
          onPress={handleSubmit}
          disabled={loading}
          activeOpacity={0.8}
        >
          <Text style={styles.buttonText}>
            {loading ? 'กำลังสมัครสมาชิก...' : 'สมัครสมาชิก'}
          </Text>
        </TouchableOpacity>
      </ScrollView>
    </KeyboardAvoidingView>
  );
}

const styles = StyleSheet.create({
  container: {
    padding: 20
  },
  title: {
    fontSize: 28,
    fontWeight: '700',
    color: '#1a1a1a',
    marginBottom: 24,
    textAlign: 'center'
  },
  field: {
    marginBottom: 16
  },
  label: {
    fontSize: 14,
    fontWeight: '600',
    color: '#333',
    marginBottom: 6
  },
  input: {
    height: 48,
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 10,
    paddingHorizontal: 14,
    fontSize: 16,
    backgroundColor: '#fff',
    color: '#1a1a1a'
  },
  inputError: {
    borderColor: '#FF3B30'
  },
  textArea: {
    height: 100,
    paddingTop: 12
  },
  passwordContainer: {
    flexDirection: 'row',
    alignItems: 'center'
  },
  passwordInput: {
    flex: 1,
    height: 48,
    borderWidth: 1,
    borderColor: '#ddd',
    borderRadius: 10,
    paddingHorizontal: 14,
    fontSize: 16,
    backgroundColor: '#fff'
  },
  eyeButton: {
    position: 'absolute',
    right: 12
  },
  error: {
    color: '#FF3B30',
    fontSize: 12,
    marginTop: 4
  },
  charCount: {
    textAlign: 'right',
    color: '#999',
    fontSize: 12,
    marginTop: 4
  },
  button: {
    height: 52,
    backgroundColor: '#007AFF',
    borderRadius: 12,
    alignItems: 'center',
    justifyContent: 'center',
    marginTop: 8
  },
  buttonDisabled: {
    opacity: 0.6
  },
  buttonText: {
    color: '#fff',
    fontSize: 16,
    fontWeight: '700'
  }
});
```

---

## Step 1936: React Navigation

### Stack Navigator

```typescript
// App.tsx - Navigation Setup
import React from 'react';
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import { createDrawerNavigator } from '@react-navigation/drawer';

// Types
type RootStackParamList = {
  Home: undefined;
  ProductDetail: { productId: string; productName: string };
  Cart: undefined;
  Checkout: { items: CartItem[] };
};

type CartItem = { id: string; quantity: number };

const Stack = createNativeStackNavigator<RootStackParamList>();
const Tab = createBottomTabNavigator();
const Drawer = createDrawerNavigator();

// Stack Navigator
function ShopStack() {
  return (
    <Stack.Navigator
      initialRouteName="Home"
      screenOptions={{
        headerStyle: { backgroundColor: '#007AFF' },
        headerTintColor: '#fff',
        headerTitleStyle: { fontWeight: '700' },
        animation: 'slide_from_right',
        gestureEnabled: true
      }}
    >
      <Stack.Screen
        name="Home"
        component={HomeScreen}
        options={{
          title: 'หน้าหลัก',
          headerRight: () => (
            <TouchableOpacity onPress={() => navigation.navigate('Cart')}>
              <Text>🛒</Text>
            </TouchableOpacity>
          )
        }}
      />
      <Stack.Screen
        name="ProductDetail"
        component={ProductDetailScreen}
        options={({ route }) => ({
          title: route.params.productName
        })}
      />
      <Stack.Screen
        name="Cart"
        component={CartScreen}
        options={{
          title: 'ตะกร้าสินค้า',
          presentation: 'modal'
        }}
      />
      <Stack.Screen
        name="Checkout"
        component={CheckoutScreen}
        options={{
          title: 'ชำระเงิน',
          headerBackVisible: false
        }}
      />
    </Stack.Navigator>
  );
}

// Tab Navigator
function MainTabs() {
  return (
    <Tab.Navigator
      screenOptions={({ route }) => ({
        tabBarIcon: ({ focused, color, size }) => {
          const icons: Record<string, string> = {
            Shop: '🛍️',
            Search: '🔍',
            Profile: '👤',
            Orders: '📦'
          };
          return <Text style={{ fontSize: size }}>{icons[route.name]}</Text>;
        },
        tabBarActiveTintColor: '#007AFF',
        tabBarInactiveTintColor: '#999',
        tabBarStyle: {
          borderTopColor: '#eee',
          paddingBottom: 4,
          height: 56
        },
        tabBarLabelStyle: {
          fontSize: 12,
          fontWeight: '600'
        }
      })}
    >
      <Tab.Screen name="Shop" component={ShopStack} options={{ title: 'ร้านค้า' }} />
      <Tab.Screen name="Search" component={SearchScreen} options={{ title: 'ค้นหา' }} />
      <Tab.Screen name="Orders" component={OrdersScreen} options={{ title: 'คำสั่งซื้อ' }} />
      <Tab.Screen name="Profile" component={ProfileScreen} options={{ title: 'โปรไฟล์' }} />
    </Tab.Navigator>
  );
}

// Screens
import { useNavigation, useRoute } from '@react-navigation/native';
import type { NativeStackNavigationProp } from '@react-navigation/native-stack';
import type { RouteProp } from '@react-navigation/native';

type HomeNavProp = NativeStackNavigationProp<RootStackParamList, 'Home'>;

function HomeScreen() {
  const navigation = useNavigation<HomeNavProp>();

  return (
    <View style={{ flex: 1, padding: 16 }}>
      <Text style={{ fontSize: 24, fontWeight: '700' }}>สินค้าทั้งหมด</Text>
      <TouchableOpacity
        onPress={() => navigation.navigate('ProductDetail', {
          productId: '123',
          productName: 'สินค้าตัวอย่าง'
        })}
      >
        <Text>ดูสินค้า</Text>
      </TouchableOpacity>
    </View>
  );
}

type ProductDetailRouteProp = RouteProp<RootStackParamList, 'ProductDetail'>;

function ProductDetailScreen() {
  const navigation = useNavigation();
  const route = useRoute<ProductDetailRouteProp>();
  const { productId, productName } = route.params;

  return (
    <View style={{ flex: 1, padding: 16 }}>
      <Text style={{ fontSize: 20 }}>{productName}</Text>
      <Text>ID: {productId}</Text>
      <TouchableOpacity
        onPress={() => navigation.goBack()}
        style={{ marginTop: 16 }}
      >
        <Text>กลับ</Text>
      </TouchableOpacity>
    </View>
  );
}

// App
export default function App() {
  return (
    <NavigationContainer>
      <MainTabs />
    </NavigationContainer>
  );
}
```

---

## Step 1937: State Management

### Zustand สำหรับ State Management

```typescript
// store/index.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

interface Product {
  id: string;
  name: string;
  price: number;
  image: string;
}

interface CartItem extends Product {
  quantity: number;
}

interface CartStore {
  items: CartItem[];
  addToCart: (product: Product) => void;
  removeFromCart: (productId: string) => void;
  updateQuantity: (productId: string, quantity: number) => void;
  clearCart: () => void;
  totalItems: () => number;
  totalPrice: () => number;
}

export const useCartStore = create<CartStore>()(
  persist(
    (set, get) => ({
      items: [],

      addToCart: (product) => {
        const items = get().items;
        const existing = items.find(item => item.id === product.id);

        if (existing) {
          set({
            items: items.map(item =>
              item.id === product.id
                ? { ...item, quantity: item.quantity + 1 }
                : item
            )
          });
        } else {
          set({ items: [...items, { ...product, quantity: 1 }] });
        }
      },

      removeFromCart: (productId) => {
        set({ items: get().items.filter(item => item.id !== productId) });
      },

      updateQuantity: (productId, quantity) => {
        if (quantity <= 0) {
          get().removeFromCart(productId);
          return;
        }
        set({
          items: get().items.map(item =>
            item.id === productId ? { ...item, quantity } : item
          )
        });
      },

      clearCart: () => set({ items: [] }),

      totalItems: () => get().items.reduce((sum, item) => sum + item.quantity, 0),
      totalPrice: () => get().items.reduce(
        (sum, item) => sum + item.price * item.quantity, 0
      )
    }),
    {
      name: 'cart-storage',
      storage: createJSONStorage(() => AsyncStorage)
    }
  )
);

// ใช้งาน
function CartIcon() {
  const totalItems = useCartStore(state => state.totalItems());

  return (
    <View>
      <Text>🛒</Text>
      {totalItems > 0 && (
        <View style={styles.badge}>
          <Text style={styles.badgeText}>{totalItems}</Text>
        </View>
      )}
    </View>
  );
}

function AddToCartButton({ product }: { product: Product }) {
  const addToCart = useCartStore(state => state.addToCart);

  return (
    <TouchableOpacity onPress={() => addToCart(product)}>
      <Text>เพิ่มในตะกร้า</Text>
    </TouchableOpacity>
  );
}
```

---

## Step 1938: AsyncStorage

### Local Storage สำหรับ React Native

```typescript
// hooks/useStorage.ts
import AsyncStorage from '@react-native-async-storage/async-storage';
import { useState, useEffect, useCallback } from 'react';

export function useAsyncStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(initialValue);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadValue();
  }, [key]);

  const loadValue = async () => {
    try {
      const stored = await AsyncStorage.getItem(key);
      if (stored !== null) {
        setValue(JSON.parse(stored));
      }
    } catch (error) {
      console.error('AsyncStorage read error:', error);
    } finally {
      setLoading(false);
    }
  };

  const storeValue = useCallback(async (newValue: T | ((prev: T) => T)) => {
    try {
      const valueToStore = newValue instanceof Function ? newValue(value) : newValue;
      setValue(valueToStore);
      await AsyncStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error('AsyncStorage write error:', error);
    }
  }, [key, value]);

  const removeValue = useCallback(async () => {
    try {
      setValue(initialValue);
      await AsyncStorage.removeItem(key);
    } catch (error) {
      console.error('AsyncStorage remove error:', error);
    }
  }, [key, initialValue]);

  return { value, storeValue, removeValue, loading };
}

// การใช้งาน
function UserPreferences() {
  const { value: theme, storeValue: setTheme } = useAsyncStorage('theme', 'light');
  const { value: language, storeValue: setLanguage } = useAsyncStorage('language', 'th');

  return (
    <View>
      <TouchableOpacity onPress={() => setTheme(prev => prev === 'light' ? 'dark' : 'light')}>
        <Text>Theme: {theme}</Text>
      </TouchableOpacity>

      <TouchableOpacity onPress={() => setLanguage('en')}>
        <Text>Language: {language}</Text>
      </TouchableOpacity>
    </View>
  );
}

// AsyncStorage Utilities
const Storage = {
  async get<T>(key: string): Promise<T | null> {
    const value = await AsyncStorage.getItem(key);
    return value ? JSON.parse(value) : null;
  },

  async set<T>(key: string, value: T): Promise<void> {
    await AsyncStorage.setItem(key, JSON.stringify(value));
  },

  async remove(key: string): Promise<void> {
    await AsyncStorage.removeItem(key);
  },

  async clear(): Promise<void> {
    await AsyncStorage.clear();
  },

  async keys(): Promise<string[]> {
    return await AsyncStorage.getAllKeys();
  },

  async multiGet<T>(keys: string[]): Promise<Record<string, T>> {
    const pairs = await AsyncStorage.multiGet(keys);
    return Object.fromEntries(
      pairs.map(([key, value]) => [key, value ? JSON.parse(value) : null])
    );
  },

  async multiSet(items: Record<string, unknown>): Promise<void> {
    const pairs = Object.entries(items).map(
      ([key, value]) => [key, JSON.stringify(value)] as [string, string]
    );
    await AsyncStorage.multiSet(pairs);
  }
};
```

---

## Step 1939: Camera และ Media

### Expo Camera

```typescript
// screens/CameraScreen.tsx
import React, { useState, useRef, useCallback } from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  Image,
  StyleSheet,
  Alert
} from 'react-native';
import { Camera, CameraType, FlashMode } from 'expo-camera';
import * as MediaLibrary from 'expo-media-library';
import * as ImagePicker from 'expo-image-picker';
import * as ImageManipulator from 'expo-image-manipulator';

export default function CameraScreen() {
  const [hasPermission, setHasPermission] = useState<boolean | null>(null);
  const [type, setType] = useState(CameraType.back);
  const [flash, setFlash] = useState(FlashMode.off);
  const [photo, setPhoto] = useState<string | null>(null);
  const [isRecording, setIsRecording] = useState(false);
  const cameraRef = useRef<Camera>(null);

  React.useEffect(() => {
    requestPermissions();
  }, []);

  const requestPermissions = async () => {
    const { status: cameraStatus } = await Camera.requestCameraPermissionsAsync();
    const { status: micStatus } = await Camera.requestMicrophonePermissionsAsync();
    const { status: mediaStatus } = await MediaLibrary.requestPermissionsAsync();
    setHasPermission(
      cameraStatus === 'granted' &&
      micStatus === 'granted' &&
      mediaStatus === 'granted'
    );
  };

  const takePhoto = useCallback(async () => {
    if (!cameraRef.current) return;

    try {
      const photo = await cameraRef.current.takePictureAsync({
        quality: 0.8,
        base64: false,
        skipProcessing: false,
        exif: true
      });

      // Resize และ compress
      const processed = await ImageManipulator.manipulateAsync(
        photo.uri,
        [
          { resize: { width: 1200 } }
        ],
        {
          compress: 0.7,
          format: ImageManipulator.SaveFormat.JPEG
        }
      );

      setPhoto(processed.uri);
    } catch (error) {
      Alert.alert('ผิดพลาด', 'ไม่สามารถถ่ายรูปได้');
    }
  }, []);

  const savePhoto = useCallback(async () => {
    if (!photo) return;

    try {
      const asset = await MediaLibrary.createAssetAsync(photo);
      await MediaLibrary.createAlbumAsync('My App', asset, false);
      Alert.alert('สำเร็จ', 'บันทึกรูปแล้ว');
    } catch (error) {
      Alert.alert('ผิดพลาด', 'ไม่สามารถบันทึกรูปได้');
    }
  }, [photo]);

  const pickFromGallery = useCallback(async () => {
    const result = await ImagePicker.launchImageLibraryAsync({
      mediaTypes: ImagePicker.MediaTypeOptions.Images,
      allowsEditing: true,
      aspect: [4, 3],
      quality: 0.8,
      allowsMultipleSelection: false
    });

    if (!result.canceled) {
      setPhoto(result.assets[0].uri);
    }
  }, []);

  const startRecording = useCallback(async () => {
    if (!cameraRef.current) return;

    setIsRecording(true);
    const video = await cameraRef.current.recordAsync({
      maxDuration: 60,
      quality: Camera.Constants.VideoQuality['720p']
    });

    setIsRecording(false);
    console.log('Video:', video.uri);
  }, []);

  const stopRecording = useCallback(() => {
    cameraRef.current?.stopRecording();
    setIsRecording(false);
  }, []);

  if (hasPermission === null) {
    return <View style={{ flex: 1 }}><Text>กำลังขอ permission...</Text></View>;
  }

  if (!hasPermission) {
    return (
      <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
        <Text>ไม่มีสิทธิ์ใช้กล้อง</Text>
        <TouchableOpacity onPress={requestPermissions}>
          <Text>ขอ permission ใหม่</Text>
        </TouchableOpacity>
      </View>
    );
  }

  if (photo) {
    return (
      <View style={{ flex: 1 }}>
        <Image source={{ uri: photo }} style={{ flex: 1 }} resizeMode="contain" />
        <View style={styles.photoActions}>
          <TouchableOpacity onPress={() => setPhoto(null)} style={styles.actionButton}>
            <Text style={styles.actionText}>ถ่ายใหม่</Text>
          </TouchableOpacity>
          <TouchableOpacity onPress={savePhoto} style={[styles.actionButton, styles.saveButton]}>
            <Text style={styles.actionText}>บันทึก</Text>
          </TouchableOpacity>
        </View>
      </View>
    );
  }

  return (
    <View style={{ flex: 1 }}>
      <Camera
        ref={cameraRef}
        style={{ flex: 1 }}
        type={type}
        flashMode={flash}
        ratio="16:9"
      >
        {/* Controls */}
        <View style={styles.controls}>
          <TouchableOpacity
            onPress={() => setType(
              type === CameraType.back ? CameraType.front : CameraType.back
            )}
          >
            <Text style={styles.icon}>🔄</Text>
          </TouchableOpacity>

          <TouchableOpacity onPress={() => setFlash(
            flash === FlashMode.off ? FlashMode.on : FlashMode.off
          )}>
            <Text style={styles.icon}>{flash === FlashMode.on ? '⚡' : '🔦'}</Text>
          </TouchableOpacity>
        </View>

        {/* Shutter */}
        <View style={styles.shutterContainer}>
          <TouchableOpacity onPress={pickFromGallery}>
            <Text style={styles.icon}>🖼️</Text>
          </TouchableOpacity>

          <TouchableOpacity
            style={styles.shutterButton}
            onPress={takePhoto}
          >
            <View style={styles.shutterInner} />
          </TouchableOpacity>

          <TouchableOpacity
            onLongPress={startRecording}
            onPressOut={stopRecording}
          >
            <Text style={styles.icon}>{isRecording ? '⏹️' : '🎥'}</Text>
          </TouchableOpacity>
        </View>
      </Camera>
    </View>
  );
}

const styles = StyleSheet.create({
  controls: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    padding: 20
  },
  icon: {
    fontSize: 30
  },
  shutterContainer: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    alignItems: 'center',
    paddingBottom: 40,
    paddingHorizontal: 30,
    marginTop: 'auto'
  },
  shutterButton: {
    width: 80,
    height: 80,
    borderRadius: 40,
    backgroundColor: 'rgba(255,255,255,0.3)',
    borderWidth: 4,
    borderColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center'
  },
  shutterInner: {
    width: 60,
    height: 60,
    borderRadius: 30,
    backgroundColor: '#fff'
  },
  photoActions: {
    flexDirection: 'row',
    padding: 20,
    gap: 12,
    backgroundColor: '#000'
  },
  actionButton: {
    flex: 1,
    height: 48,
    borderRadius: 10,
    backgroundColor: '#333',
    alignItems: 'center',
    justifyContent: 'center'
  },
  saveButton: {
    backgroundColor: '#007AFF'
  },
  actionText: {
    color: '#fff',
    fontWeight: '600',
    fontSize: 16
  }
});
```

---

## Step 1940: Location Services

### Expo Location

```typescript
// hooks/useLocation.ts
import { useState, useEffect, useCallback } from 'react';
import * as Location from 'expo-location';

interface LocationData {
  latitude: number;
  longitude: number;
  accuracy: number | null;
  altitude: number | null;
  heading: number | null;
  speed: number | null;
}

export function useLocation() {
  const [location, setLocation] = useState<LocationData | null>(null);
  const [address, setAddress] = useState<Location.LocationGeocodedAddress | null>(null);
  const [error, setError] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);
  const [watching, setWatching] = useState(false);
  const [subscription, setSubscription] = useState<Location.LocationSubscription | null>(null);

  const requestPermission = useCallback(async () => {
    const { status } = await Location.requestForegroundPermissionsAsync();
    if (status !== 'granted') {
      setError('ไม่ได้รับอนุญาตให้เข้าถึงตำแหน่ง');
      return false;
    }
    return true;
  }, []);

  const getCurrentLocation = useCallback(async () => {
    const hasPermission = await requestPermission();
    if (!hasPermission) return;

    setLoading(true);
    setError(null);

    try {
      const loc = await Location.getCurrentPositionAsync({
        accuracy: Location.Accuracy.High
      });

      const locationData: LocationData = {
        latitude: loc.coords.latitude,
        longitude: loc.coords.longitude,
        accuracy: loc.coords.accuracy,
        altitude: loc.coords.altitude,
        heading: loc.coords.heading,
        speed: loc.coords.speed
      };

      setLocation(locationData);

      // Reverse geocode
      const geocoded = await Location.reverseGeocodeAsync({
        latitude: loc.coords.latitude,
        longitude: loc.coords.longitude
      });

      if (geocoded.length > 0) {
        setAddress(geocoded[0]);
      }
    } catch (err) {
      setError('ไม่สามารถรับตำแหน่งได้');
    } finally {
      setLoading(false);
    }
  }, [requestPermission]);

  const startWatching = useCallback(async () => {
    const hasPermission = await requestPermission();
    if (!hasPermission) return;

    const sub = await Location.watchPositionAsync(
      {
        accuracy: Location.Accuracy.High,
        distanceInterval: 10, // อัปเดตทุก 10 เมตร
        timeInterval: 5000    // หรือทุก 5 วินาที
      },
      (loc) => {
        setLocation({
          latitude: loc.coords.latitude,
          longitude: loc.coords.longitude,
          accuracy: loc.coords.accuracy,
          altitude: loc.coords.altitude,
          heading: loc.coords.heading,
          speed: loc.coords.speed
        });
      }
    );

    setSubscription(sub);
    setWatching(true);
  }, [requestPermission]);

  const stopWatching = useCallback(() => {
    subscription?.remove();
    setSubscription(null);
    setWatching(false);
  }, [subscription]);

  useEffect(() => {
    return () => {
      subscription?.remove();
    };
  }, [subscription]);

  return {
    location,
    address,
    error,
    loading,
    watching,
    getCurrentLocation,
    startWatching,
    stopWatching
  };
}

// Map Component (ใช้ react-native-maps)
import MapView, { Marker, Polyline, PROVIDER_GOOGLE } from 'react-native-maps';

function LocationMap() {
  const { location, getCurrentLocation, startWatching, watching } = useLocation();
  const [route, setRoute] = useState<{ latitude: number; longitude: number }[]>([]);

  useEffect(() => {
    if (location) {
      setRoute(prev => [...prev, {
        latitude: location.latitude,
        longitude: location.longitude
      }]);
    }
  }, [location]);

  return (
    <View style={{ flex: 1 }}>
      {location && (
        <MapView
          provider={PROVIDER_GOOGLE}
          style={{ flex: 1 }}
          region={{
            latitude: location.latitude,
            longitude: location.longitude,
            latitudeDelta: 0.01,
            longitudeDelta: 0.01
          }}
          showsUserLocation={true}
          showsMyLocationButton={true}
        >
          <Marker
            coordinate={{
              latitude: location.latitude,
              longitude: location.longitude
            }}
            title="ตำแหน่งของคุณ"
          />

          {route.length > 1 && (
            <Polyline
              coordinates={route}
              strokeColor="#007AFF"
              strokeWidth={3}
            />
          )}
        </MapView>
      )}

      <View style={styles.controls}>
        <TouchableOpacity onPress={getCurrentLocation} style={styles.button}>
          <Text style={styles.buttonText}>รับตำแหน่ง</Text>
        </TouchableOpacity>

        <TouchableOpacity
          onPress={watching ? stopWatching : startWatching}
          style={[styles.button, watching && styles.activeButton]}
        >
          <Text style={styles.buttonText}>
            {watching ? 'หยุดติดตาม' : 'ติดตามตำแหน่ง'}
          </Text>
        </TouchableOpacity>
      </View>
    </View>
  );
}
```

---

## Step 1941-1950: Push Notifications และ Deployment

### Expo Notifications

```typescript
// hooks/usePushNotifications.ts
import { useState, useEffect, useRef } from 'react';
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';
import { Platform } from 'react-native';

Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: true
  })
});

export function usePushNotifications() {
  const [expoPushToken, setExpoPushToken] = useState<string | null>(null);
  const [notification, setNotification] = useState<Notifications.Notification | null>(null);
  const notificationListener = useRef<Notifications.Subscription>();
  const responseListener = useRef<Notifications.Subscription>();

  useEffect(() => {
    registerForPushNotifications();

    notificationListener.current = Notifications.addNotificationReceivedListener(
      notification => {
        setNotification(notification);
      }
    );

    responseListener.current = Notifications.addNotificationResponseReceivedListener(
      response => {
        const data = response.notification.request.content.data;
        handleNotificationResponse(data);
      }
    );

    return () => {
      if (notificationListener.current) {
        Notifications.removeNotificationSubscription(notificationListener.current);
      }
      if (responseListener.current) {
        Notifications.removeNotificationSubscription(responseListener.current);
      }
    };
  }, []);

  async function registerForPushNotifications() {
    if (!Device.isDevice) {
      console.log('Push notifications ทำงานได้บน physical device เท่านั้น');
      return;
    }

    const { status: existingStatus } = await Notifications.getPermissionsAsync();
    let finalStatus = existingStatus;

    if (existingStatus !== 'granted') {
      const { status } = await Notifications.requestPermissionsAsync();
      finalStatus = status;
    }

    if (finalStatus !== 'granted') {
      console.log('ไม่ได้รับอนุญาตสำหรับ push notifications');
      return;
    }

    if (Platform.OS === 'android') {
      await Notifications.setNotificationChannelAsync('default', {
        name: 'default',
        importance: Notifications.AndroidImportance.MAX,
        vibrationPattern: [0, 250, 250, 250],
        lightColor: '#FF231F7C',
        sound: 'default'
      });
    }

    const token = await Notifications.getExpoPushTokenAsync({
      projectId: 'your-project-id'
    });

    setExpoPushToken(token.data);

    // ส่ง token ไปยัง backend
    await registerTokenOnServer(token.data);
  }

  async function registerTokenOnServer(token: string) {
    try {
      await fetch('https://api.example.com/notifications/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ token })
      });
    } catch (error) {
      console.error('Error registering token:', error);
    }
  }

  function handleNotificationResponse(data: Record<string, unknown>) {
    if (data.screen) {
      // Navigate ไปยัง screen ที่ระบุ
      navigation.navigate(data.screen as string, data.params as Record<string, unknown>);
    }
  }

  // Local Notification
  async function scheduleLocalNotification(
    title: string,
    body: string,
    data?: Record<string, unknown>,
    trigger?: Notifications.NotificationTriggerInput
  ) {
    await Notifications.scheduleNotificationAsync({
      content: {
        title,
        body,
        data: data || {},
        sound: 'default',
        badge: 1
      },
      trigger: trigger || null // null = แสดงทันที
    });
  }

  // Schedule สำหรับเวลาที่ระบุ
  async function scheduleReminder(title: string, body: string, date: Date) {
    await Notifications.scheduleNotificationAsync({
      content: { title, body },
      trigger: {
        date
      }
    });
  }

  // Repeating notification
  async function scheduleDaily(title: string, body: string, hour: number, minute: number) {
    await Notifications.scheduleNotificationAsync({
      content: { title, body },
      trigger: {
        hour,
        minute,
        repeats: true
      }
    });
  }

  // ยกเลิก notifications
  async function cancelAllNotifications() {
    await Notifications.cancelAllScheduledNotificationsAsync();
    await Notifications.setBadgeCountAsync(0);
  }

  return {
    expoPushToken,
    notification,
    scheduleLocalNotification,
    scheduleReminder,
    scheduleDaily,
    cancelAllNotifications
  };
}
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Todo App
สร้าง Todo App ด้วย React Native:
- เพิ่ม/ลบ/แก้ไข todo
- Filter ตามสถานะ
- AsyncStorage
- Local notifications

### แบบฝึกหัดที่ 2: Weather App
สร้าง Weather App:
- ดึงข้อมูลจาก OpenWeatherMap API
- แสดงสภาพอากาศปัจจุบัน
- Weather forecast 7 วัน
- Location-based weather

### แบบฝึกหัดที่ 3: Photo Gallery
สร้าง Photo Gallery:
- เปิดกล้อง
- เลือกจาก Gallery
- Grid view
- Fullscreen view

---

## สรุป

React Native เป็น framework ที่ทรงพลังสำหรับการพัฒนา mobile app ในบทนี้เราเรียนรู้:

1. **React Native** vs **Expo** และเมื่อไหรควรใช้อะไร
2. **Core Components** - View, Text, Image, ScrollView
3. **FlatList** - สำหรับ list ขนาดใหญ่
4. **TextInput** - การจัดการ forms
5. **React Navigation** - Stack, Tab, Drawer navigators
6. **State Management** - Zustand + AsyncStorage
7. **Camera** - expo-camera
8. **Location** - expo-location + Maps
9. **Push Notifications** - expo-notifications
10. **Styling** - StyleSheet, Flexbox
