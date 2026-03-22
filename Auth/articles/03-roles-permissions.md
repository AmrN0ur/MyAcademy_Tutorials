# دليل الأدوار والصلاحيات في نظام MyAcademy

## مقدمة

في نظام MyAcademy، يعمل نظام الأدوار (Roles) بشكل بسيط ومباشر. هذا الدليل يشرح:
- ما هي الأدوار الموجودة؟
- ما هي صلاحيات كل دور؟
- كيف يتم التحقق من الصلاحيات في الكود؟

---

## الأدوار في النظام (Roles)

يوجد دوران رئيسيان فقط في النظام:

```
┌─────────────────────────────────────────────────────────────┐
│                  Admin (مدير النظام)                        │
│                    👑 أعلى صلاحيات                           │
│                   role_id = 1                               │
└─────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                  User (مستخدم عادي)                         │
│                    👤 صلاحيات محدودة                         │
│                   role_id = 0 (أو null)                     │
└─────────────────────────────────────────────────────────────┘
```

---

## ١. مدير النظام (Admin)

### من هو؟

الشخص المسؤول عن كل شيء في النظام. عادةً هو صاحب المركز التعليمي أو المدير التنفيذي.

### كيف يُعرف في النظام؟

```javascript
// في قاعدة البيانات
role_id = 1

// في الكود (userInfo.js)
isAdmin: (state) => state.user.role_id === 1
```

### الصلاحيات الكاملة

| القسم | الصلاحيات |
|-------|----------|
| **المستخدمين** | يضيف، يحذف، يشوف أي حساب |
| **الطلاب** | يضيف، يعدل، يحذف، يشوف كل الطلاب |
| **المجموعات** | يضيف مجموعات، يعدل، يحذف، يدير الطلاب |
| **الحصص** | يسجل حصص، يعدل، يلغي، يتابع الحضور |
| **الاشتراكات** | يدير الاشتراكات والمدفوعات |
| **الموظفين** | يدير ملفات الموظفين والرواتب |
| **التقارير** | يشوف كل التقارير والإحصائيات |
| **الإعدادات** | يغير إعدادات النظام كله |

### أمثلة على الاستخدام
- إضافة مستخدم جديد للنظام
- إنشاء مجموعة جديدة للطلاب
- تسجيل حضور الطلاب في الحصص
- متابعة الاشتراكات المنتهية
- إصدار تقرير مبيعات الشهر

---

## ٢. المستخدم العادي (User)

### من هو؟

الشخص العادي الذي يعمل في المركز التعليمي (مثل مدرس أو موظف استقبال).

### كيف يُعرف في النظام؟

```javascript
// في قاعدة البيانات
role_id = 0
// أو role_id = null (افتراضي)

// في الكود (userInfo.js)
isAdmin: (state) => state.user.role_id === 1
// إذا لم يكن 1، فهو User عادي
```

### الصلاحيات المحدودة

| القسم | الصلاحيات |
|-------|----------|
| **المستخدمين** | ❌ لا يستطيع إضافة أو حذف |
| **الطلاب** | ✅ يشاهد ويضيف ويعدل |
| **المجموعات** | ✅ يشاهد ويضيف ويعدل |
| **الحصص** | ✅ يسجل الحصص ويتابع الحضور |
| **الاشتراكات** | ✅ يدير الاشتراكات |
| **الموظفين** | ❌ لا يستطيع رؤية إلا نفسه |
| **التقارير** | ✅ يشاهد تقارير محدودة |
| **الإعدادات** | ❌ لا يستطيع تغيير الإعدادات العامة |

### أمثلة على الاستخدام
- تسجيل طالب جديد
- إضافة طالب لمجموعة
- تسجيل حضور في حصة
- تجديد اشتراك طالب
- مشاهدة تقرير الحصص اليوم

---

## جدول مقارنة سريع

| الإجراء | Admin (role_id=1) | User (role_id=0) |
|---------|:-----------------:|:----------------:|
| إضافة مستخدم | ✅ | ❌ |
| حذف مستخدم | ✅ | ❌ |
| عرض قائمة المستخدمين | ✅ | ❌ |
| إضافة طالب | ✅ | ✅ |
| تعديل طالب | ✅ | ✅ |
| حذف طالب | ✅ | ✅ |
| إضافة مجموعة | ✅ | ✅ |
| إدارة الحصص | ✅ | ✅ |
| الاشتراكات | ✅ | ✅ |
| تقارير مالية | ✅ | محدود |
| تغيير إعدادات النظام | ✅ | ❌ |
| تغيير بياناته الشخصية | ✅ | ✅ |
| تغيير كلمة المرور | ✅ | ✅ |

---

## كيف يتم فحص الصلاحيات في الكود؟

### ١. في المتجر (Store)

ملف: `stores/userInfo.js`

```javascript
export const useUserInfoStore = defineStore('userInfo', {
  state: () => ({
    user: {
      id: null,
      name: '',
      user_name: '',
      email: '',
      role_id: null, // 1: Admin, 0: User
    },
  }),

  getters: {
    // هل المستخدم Admin؟
    isAdmin: (state) => state.user.role_id === 1,
    
    // الحصول على الدور كنص
    getUserRole: (state) => {
      if (state.user.role_id === 1) return 'admin'
      else if (state.user.role_id === 0) return 'user'
      else return 'unknown'
    },
  },
})
```

### ٢. في المكونات (Components)

```vue
<script setup>
import { useUserInfoStore } from '@/stores/userInfo'
import { computed } from 'vue'

const userStore = useUserInfoStore()

// التحقق من Admin
const isAdmin = computed(() => userStore.isAdmin)

// أو مباشرة
const canManageUsers = computed(() => userStore.getUserRoleID === 1)
</script>

<template>
  <!-- إظهار زر إضافة مستخدم فقط للـ Admin -->
  <Button v-if="isAdmin" @click="addUser">إضافة مستخدم</Button>
  
  <!-- أو -->
  <Button v-if="userStore.isAdmin" @click="addUser">إضافة مستخدم</Button>
</template>
```

### ٣. في الباك اند (Backend)

ملف: `backend/app/Models/User.php`

```php
// التحقق من Admin
public function isAdmin(): bool
{
    return $this->role_id === 1;
}
```

ملف: `backend/routes/auth.php`

```php
// Routes تتطلب Admin
Route::get('users', [AuthController::class, 'index'])
    ->name('user.index')
    ->middleware('CheckPermission');
```

---

## Middleware الصلاحيات

### CheckJwtToken

يتحقق من وجود توكن صالح:

```php
// في api.php
Route::middleware('CheckJwtToken')->group(function () {
    // هذه Routes تتطلب تسجيل دخول
    Route::post('logout', [AuthController::class, 'logout']);
    Route::get('users', [AuthController::class, 'index']);
});
```

### CheckPermission

يتحقق من صلاحيات محددة:

```php
// في auth.php
Route::get('users', [AuthController::class, 'index'])
    ->middleware('CheckPermission'); // يتحقق من Admin
```

### guest.jwt

يسمح بالوصول للضيوف فقط (غير مسجلين):

```php
// في guest.php
Route::middleware('guest.jwt')->group(function () {
    Route::post('guest/login', [AuthController::class, 'login']);
});
```

---

## حماية الحسابات

### ١. لا يمكن حذف المدير الرئيسي

لو حاولت حذف المستخدم ذو ID = 1:

```php
// في AuthController.php
public function destroy(User $user)
{
    if ($user->id == 1) {
        return $this->returnRes(false, null, 'لا يمكن حذف المدير الرئيسي', [], 200);
    }
    $user->delete();
    return $this->returnRes(true, null, 'تم حذف المستخدم بنجاح', null, 200);
}
```

### ٢. حماية البيانات

- كلمة المرور: تُخزن مشفرة بـ Hash
- JWT Token: يُخزن في Cookie httpOnly (لا يمكن قراءته من JavaScript)
- الصلاحيات: تُفحص في الباكند ولا تعتمد على الفرونت فقط

---

## استجابات API حسب الصلاحيات

### ١. نجاح (Success)

```json
{
  "success": true,
  "data": { ... },
  "message": "تمت العملية بنجاح"
}
```

### ٢. خطأ في المصادقة (Authentication Error)

```json
{
  "success": false,
  "message": "انتهت صلاحية الجلسة"
}
```

**HTTP Status:** 401 أو 432

### ٣. خطأ في الصلاحيات (Permission Denied)

```json
{
  "success": false,
  "message": "غير مصرح لك بالوصول"
}
```

**HTTP Status:** 403

---

## الأسئلة المتكررة (FAQ)

### س: كيف أجعل مستخدماً Admin؟

**ج:** هذا يتطلب تغيير مباشر في قاعدة البيانات أو من خلال فريق الدعم الفني. قم بتعيين `role_id = 1` للمستخدم.

### س: هل يمكنني إنشاء دور جديد مثل "مدرس"؟

**ج:** النظام حالياً يدعم دورين فقط: Admin و User. لإضافة أدوار جديدة، يتطلب ذلك تعديل في كود النظام.

### س: User يمكنه إضافة طلاب، هل هذا صحيح؟

**ج:** نعم، في نظام MyAcademy، الـ User العادي يمكنه:
- إضافة الطلاب
- إدارة المجموعات
- تسجيل الحصص
- إدارة الاشتراكات

الفرق الرئيسي هو أنه لا يستطيع إضافة مستخدمين جدد أو رؤية قائمة المستخدمين.

### س: ما هي البيانات التي يستلمها المستخدم بعد الدخول؟

**ج:**

```json
{
  "id": 5,
  "name": "محمد علي",
  "user_name": "mohamed.ali",
  "email": "mohamed@academy.com",
  "role_id": 1  // أو 0
}
```

### س: كيف يعرف النظام أني Admin أم User؟

**ج:** من خلال `role_id` في البيانات المُرسلة من الباكند بعد تسجيل الدخول.

---

## ملخص سريع

| الدور | role_id | لمن؟ | السلطة |
|-------|---------|------|--------|
| **Admin** | 1 | صاحب المركز، المدير | الكامل (مع إدارة المستخدمين) |
| **User** | 0 أو null | المدرسين، موظفي الاستقبال | العمليات اليومية (بدون إدارة المستخدمين) |

---

## كود فحص الصلاحيات (Reference)

### Vue 3 (Composition API)

```vue
<script setup lang="ts">
import { useUserInfoStore } from '@/stores/userInfo'
import { storeToRefs } from 'pinia'

const userStore = useUserInfoStore()
const { isAdmin } = storeToRefs(userStore)

// استخدم isAdmin.value للتحقق
</script>
```

### Vue Router (Navigation Guard)

```typescript
// في router/index.ts
router.beforeEach((to, from, next) => {
  const userStore = useUserInfoStore()
  
  // إذا كانت الصفحة تتطلب Admin
  if (to.meta.requiresAdmin && !userStore.isAdmin) {
    next('/access-denied')
    return
  }
  
  next()
})
```

### Laravel (Backend)

```php
// في Controller
public function someAction()
{
    $user = auth('api_user')->user();
    
    if (!$user || $user->role_id !== 1) {
        return response()->json([
            'success' => false,
            'message' => 'غير مصرح لك'
        ], 403);
    }
    
    // أكمل العملية...
}
```

---

**هل تريد تغيير دور أحدهم؟** تواصل مع فريق الدعم الفني.

---

*آخر تحديث: مارس ٢٠٢٦*
