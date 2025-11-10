# 🎯 Advanced Skills & Specializations

<div align="center">

![Laravel](https://img.shields.io/badge/Laravel_Expert-FF2D20?style=for-the-badge&logo=laravel&logoColor=white)
![Vue.js](https://img.shields.io/badge/Vue.js_Master-4FC08D?style=for-the-badge&logo=vue.js&logoColor=white)
![Flutter](https://img.shields.io/badge/Flutter_Pro-02569B?style=for-the-badge&logo=flutter&logoColor=white)

</div>

## 📑 Table of Contents

- [Laravel Deep Dive](#-laravel-deep-dive)
- [Vue.js Mastery](#-vuejs-mastery)
- [Flutter Excellence](#-flutter-excellence)
- [Database Expertise](#-database-expertise)
- [DevOps & Cloud](#-devops--cloud)
- [System Architecture](#-system-architecture)
- [E-Commerce Solutions](#-e-commerce-solutions)
- [API Development](#-api-development)

---

## 🔥 Laravel Deep Dive

### Core Laravel Expertise

<details>
<summary><b>🏗️ Architecture & Design Patterns (Click to expand)</b></summary>

```php
<?php

namespace App\Patterns;

/**
 * Repository Pattern Implementation
 * Used in 15+ projects for clean architecture
 */
interface UserRepositoryInterface
{
    public function find(int $id): ?User;
    public function create(array $data): User;
    public function update(int $id, array $data): bool;
    public function delete(int $id): bool;
}

class UserRepository implements UserRepositoryInterface
{
    public function __construct(
        private User $model
    ) {}
    
    public function find(int $id): ?User
    {
        return $this->model->find($id);
    }
    
    // Additional methods...
}

/**
 * Service Layer Pattern
 * Business logic separation
 */
class UserService
{
    public function __construct(
        private UserRepositoryInterface $userRepository,
        private NotificationService $notificationService
    ) {}
    
    public function registerUser(array $data): User
    {
        DB::beginTransaction();
        
        try {
            $user = $this->userRepository->create($data);
            $this->notificationService->sendWelcomeEmail($user);
            
            DB::commit();
            return $user;
        } catch (\Exception $e) {
            DB::rollBack();
            throw $e;
        }
    }
}

/**
 * Observer Pattern
 * Event-driven architecture
 */
class UserObserver
{
    public function created(User $user): void
    {
        // Send notification
        // Log activity
        // Update related records
    }
    
    public function updated(User $user): void
    {
        // Handle updates
    }
}

/**
 * Factory Pattern
 * Object creation abstraction
 */
class PaymentFactory
{
    public static function create(string $type): PaymentInterface
    {
        return match($type) {
            'stripe' => new StripePayment(),
            'paypal' => new PaypalPayment(),
            'square' => new SquarePayment(),
            default => throw new \InvalidArgumentException("Invalid payment type")
        };
    }
}
```

</details>

<details>
<summary><b>⚡ Advanced Eloquent ORM (Click to expand)</b></summary>

```php
<?php

// Complex Relationships
class User extends Model
{
    // Polymorphic Relations
    public function comments()
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
    
    // Many-to-Many with Pivot
    public function roles()
    {
        return $this->belongsToMany(Role::class)
                    ->withPivot('assigned_at', 'assigned_by')
                    ->withTimestamps();
    }
    
    // Has Many Through
    public function posts()
    {
        return $this->hasManyThrough(Post::class, Blog::class);
    }
    
    // Dynamic Relationships
    public function latestPost()
    {
        return $this->hasOne(Post::class)->latestOfMany();
    }
}

// Global Scopes
class ActiveScope implements Scope
{
    public function apply(Builder $builder, Model $model): void
    {
        $builder->where('status', 'active');
    }
}

// Local Scopes
class Post extends Model
{
    public function scopePublished(Builder $query): void
    {
        $query->where('published', true)
              ->where('published_at', '<=', now());
    }
    
    public function scopePopular(Builder $query): void
    {
        $query->where('views', '>', 1000)
              ->orderBy('views', 'desc');
    }
}

// Query Optimization
class ProductRepository
{
    public function getWithRelations()
    {
        return Product::query()
            ->with([
                'category:id,name',
                'images' => fn($q) => $q->select('id', 'product_id', 'url')
                                         ->limit(5),
                'reviews' => fn($q) => $q->where('approved', true)
                                         ->latest()
            ])
            ->withCount(['reviews', 'sales'])
            ->withAvg('reviews', 'rating')
            ->whereHas('category', fn($q) => $q->where('active', true))
            ->select(['id', 'name', 'price', 'category_id'])
            ->paginate(15);
    }
}

// Custom Collections
class UserCollection extends Collection
{
    public function premium(): self
    {
        return $this->filter(fn($user) => $user->is_premium);
    }
    
    public function totalRevenue(): float
    {
        return $this->sum('total_spent');
    }
}
```

</details>

<details>
<summary><b>🔐 Advanced Authentication & Authorization (Click to expand)</b></summary>

```php
<?php

// Custom Guard
class ApiTokenGuard implements Guard
{
    public function check(): bool
    {
        return !is_null($this->user());
    }
    
    public function user()
    {
        if (!is_null($this->user)) {
            return $this->user;
        }
        
        $token = $this->getTokenFromRequest();
        return $this->user = User::where('api_token', $token)->first();
    }
}

// Policy-based Authorization
class PostPolicy
{
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id 
            || $user->hasRole('admin');
    }
    
    public function delete(User $user, Post $post): bool
    {
        return $user->id === $post->user_id 
            || $user->hasPermission('delete-any-post');
    }
}

// Multi-tenancy Implementation
class TenantMiddleware
{
    public function handle(Request $request, Closure $next)
    {
        $tenant = $this->resolveTenant($request);
        
        app()->instance('tenant', $tenant);
        
        config(['database.connections.tenant.database' => $tenant->database]);
        
        DB::purge('tenant');
        DB::reconnect('tenant');
        
        return $next($request);
    }
}

// Role-Based Access Control
class Role extends Model
{
    public function permissions()
    {
        return $this->belongsToMany(Permission::class);
    }
}

trait HasRoles
{
    public function roles()
    {
        return $this->belongsToMany(Role::class);
    }
    
    public function hasRole(string $role): bool
    {
        return $this->roles()->where('name', $role)->exists();
    }
    
    public function hasPermission(string $permission): bool
    {
        return $this->roles()
            ->whereHas('permissions', fn($q) => 
                $q->where('name', $permission)
            )->exists();
    }
}
```

</details>

<details>
<summary><b>⚙️ Queue System & Background Jobs (Click to expand)</b></summary>

```php
<?php

// Advanced Job Implementation
class ProcessLargeDataset implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
    
    public $tries = 3;
    public $timeout = 120;
    public $backoff = [10, 30, 60];
    
    public function __construct(
        public Collection $data,
        public User $user
    ) {}
    
    public function handle(): void
    {
        $this->data->chunk(1000)->each(function ($chunk) {
            ProcessChunk::dispatch($chunk, $this->user)
                        ->onQueue('high-priority');
        });
    }
    
    public function failed(Throwable $exception): void
    {
        Log::error('Job failed', [
            'job' => self::class,
            'exception' => $exception->getMessage()
        ]);
        
        $this->user->notify(new JobFailedNotification($exception));
    }
}

// Job Chaining
class OrderProcessor
{
    public function process(Order $order): void
    {
        Bus::chain([
            new ValidateOrder($order),
            new ChargePayment($order),
            new UpdateInventory($order),
            new SendConfirmationEmail($order),
            new NotifyShipping($order),
        ])->catch(function (Throwable $e) use ($order) {
            RollbackOrder::dispatch($order);
        })->dispatch();
    }
}

// Event Sourcing with Queue
class UserRegistered
{
    public function __construct(public User $user) {}
}

class SendWelcomeEmail implements ShouldQueue
{
    public function handle(UserRegistered $event): void
    {
        Mail::to($event->user)->send(new WelcomeEmail($event->user));
    }
}

// Scheduled Tasks
class Kernel extends ConsoleKernel
{
    protected function schedule(Schedule $schedule): void
    {
        $schedule->job(new GenerateMonthlyReport)
                 ->monthlyOn(1, '00:00')
                 ->timezone('Asia/Riyadh');
        
        $schedule->call(function () {
            Cache::flush();
        })->daily()->at('03:00');
        
        $schedule->command('backup:run')
                 ->dailyAt('01:00')
                 ->sendOutputTo(storage_path('logs/backup.log'));
    }
}
```

</details>

<details>
<summary><b>🚀 Performance Optimization (Click to expand)</b></summary>

```php
<?php

// Redis Caching Strategy
class ProductService
{
    public function getFeaturedProducts(): Collection
    {
        return Cache::tags(['products', 'featured'])
            ->remember('featured-products', 3600, function () {
                return Product::with('images', 'category')
                    ->where('featured', true)
                    ->orderBy('priority')
                    ->get();
            });
    }
    
    public function clearProductCache(Product $product): void
    {
        Cache::tags(['products'])->flush();
        Cache::forget("product.{$product->id}");
    }
}

// Database Query Optimization
class OptimizedQueries
{
    // Bad: N+1 Problem
    public function getUsersWithPostsBad()
    {
        $users = User::all();
        foreach ($users as $user) {
            $posts = $user->posts; // Causes N queries
        }
    }
    
    // Good: Eager Loading
    public function getUsersWithPostsGood()
    {
        return User::with('posts')->get(); // 2 queries only
    }
    
    // Chunk for Large Datasets
    public function processLargeDataset()
    {
        User::chunk(1000, function ($users) {
            foreach ($users as $user) {
                // Process each user
            }
        });
    }
    
    // Lazy Collections for Memory Efficiency
    public function processWithLazyCollection()
    {
        User::cursor()->each(function ($user) {
            // Process without loading all into memory
        });
    }
}

// Response Caching
Route::get('/api/products', [ProductController::class, 'index'])
    ->middleware('cache.headers:public;max_age=3600');

// Asset Optimization
class AssetOptimization
{
    public function optimizeImage(UploadedFile $file): string
    {
        $image = Image::make($file);
        
        $image->resize(1200, null, function ($constraint) {
            $constraint->aspectRatio();
            $constraint->upsize();
        });
        
        $image->encode('jpg', 85);
        
        $path = Storage::putFileAs(
            'images',
            $file,
            Str::uuid() . '.jpg',
            'public'
        );
        
        return $path;
    }
}
```

</details>

### Laravel Project Statistics

| Metric | Value |
|--------|-------|
| **Total Laravel Projects** | 18+ |
| **Lines of Code** | 300K+ |
| **Packages Integrated** | 50+ |
| **Custom Packages Created** | 5 |
| **Average Project Size** | 15K-20K LOC |
| **Largest Project** | 45K LOC (ERP System) |
| **Testing Coverage** | 75%+ average |

---

## 🎨 Vue.js Mastery

### Vue 3 Composition API Expertise

<details>
<summary><b>⚡ Advanced Composition API Patterns (Click to expand)</b></summary>

```vue
<script setup>
// Composables for Reusable Logic
import { ref, computed, watch, onMounted } from 'vue'
import { useRouter } from 'vue-router'

// Custom Composable: useFetch
export function useFetch(url) {
  const data = ref(null)
  const error = ref(null)
  const loading = ref(false)
  
  const fetchData = async () => {
    loading.value = true
    error.value = null
    
    try {
      const response = await fetch(url.value)
      data.value = await response.json()
    } catch (e) {
      error.value = e
    } finally {
      loading.value = false
    }
  }
  
  watch(url, fetchData, { immediate: true })
  
  return { data, error, loading, refetch: fetchData }
}

// Custom Composable: useAuth
export function useAuth() {
  const router = useRouter()
  const user = ref(null)
  const token = ref(localStorage.getItem('token'))
  
  const login = async (credentials) => {
    const response = await api.post('/login', credentials)
    token.value = response.data.token
    user.value = response.data.user
    localStorage.setItem('token', token.value)
    router.push('/dashboard')
  }
  
  const logout = () => {
    user.value = null
    token.value = null
    localStorage.removeItem('token')
    router.push('/login')
  }
  
  const isAuthenticated = computed(() => !!token.value)
  
  return { user, login, logout, isAuthenticated }
}

// Component Using Composables
const url = ref('/api/users')
const { data: users, loading, error } = useFetch(url)
const { user, isAuthenticated } = useAuth()

// Reactive Computed Properties
const filteredUsers = computed(() => {
  if (!users.value) return []
  return users.value.filter(u => u.active)
})

// Watchers with Side Effects
watch(
  () => route.params.id,
  async (newId) => {
    await fetchUser(newId)
  },
  { immediate: true }
)

// Lifecycle Hooks
onMounted(() => {
  console.log('Component is mounted')
})
</script>

<template>
  <div v-if="loading">Loading...</div>
  <div v-else-if="error">Error: {{ error.message }}</div>
  <div v-else>
    <UserCard
      v-for="user in filteredUsers"
      :key="user.id"
      :user="user"
    />
  </div>
</template>
```

</details>

<details>
<summary><b>🏪 State Management with Pinia (Click to expand)</b></summary>

```javascript
// stores/user.js
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import axios from 'axios'

export const useUserStore = defineStore('user', () => {
  // State
  const users = ref([])
  const currentUser = ref(null)
  const loading = ref(false)
  const error = ref(null)
  
  // Getters (Computed)
  const activeUsers = computed(() => 
    users.value.filter(user => user.status === 'active')
  )
  
  const userCount = computed(() => users.value.length)
  
  const getUserById = computed(() => {
    return (userId) => users.value.find(u => u.id === userId)
  })
  
  // Actions
  async function fetchUsers() {
    loading.value = true
    error.value = null
    
    try {
      const response = await axios.get('/api/users')
      users.value = response.data
    } catch (e) {
      error.value = e.message
    } finally {
      loading.value = false
    }
  }
  
  async function createUser(userData) {
    try {
      const response = await axios.post('/api/users', userData)
      users.value.push(response.data)
      return response.data
    } catch (e) {
      error.value = e.message
      throw e
    }
  }
  
  async function updateUser(userId, userData) {
    try {
      const response = await axios.put(`/api/users/${userId}`, userData)
      const index = users.value.findIndex(u => u.id === userId)
      if (index !== -1) {
        users.value[index] = response.data
      }
      return response.data
    } catch (e) {
      error.value = e.message
      throw e
    }
  }
  
  async function deleteUser(userId) {
    try {
      await axios.delete(`/api/users/${userId}`)
      users.value = users.value.filter(u => u.id !== userId)
    } catch (e) {
      error.value = e.message
      throw e
    }
  }
  
  function setCurrentUser(user) {
    currentUser.value = user
  }
  
  function $reset() {
    users.value = []
    currentUser.value = null
    loading.value = false
    error.value = null
  }
  
  return {
    // State
    users,
    currentUser,
    loading,
    error,
    // Getters
    activeUsers,
    userCount,
    getUserById,
    // Actions
    fetchUsers,
    createUser,
    updateUser,
    deleteUser,
    setCurrentUser,
    $reset
  }
})

// Usage in Component
<script setup>
import { useUserStore } from '@/stores/user'
import { storeToRefs } from 'pinia'

const userStore = useUserStore()

// Reactive refs from store
const { users, loading, activeUsers } = storeToRefs(userStore)

// Actions
const { fetchUsers, createUser } = userStore

onMounted(() => {
  fetchUsers()
})
</script>
```

</details>

<details>
<summary><b>🎯 Advanced Component Patterns (Click to expand)</b></summary>

```vue
<!-- Renderless Component Pattern -->
<script setup>
// MouseTracker.vue - Renderless Component
import { ref, onMounted, onUnmounted } from 'vue'

const x = ref(0)
const y = ref(0)

const updateMouse = (event) => {
  x.value = event.pageX
  y.value = event.pageY
}

onMounted(() => {
  window.addEventListener('mousemove', updateMouse)
})

onUnmounted(() => {
  window.removeEventListener('mousemove', updateMouse)
})

defineExpose({ x, y })
</script>

<template>
  <slot :x="x" :y="y"></slot>
</template>

<!-- Usage -->
<MouseTracker v-slot="{ x, y }">
  <div>Mouse position: {{ x }}, {{ y }}</div>
</MouseTracker>

<!-- Compound Component Pattern -->
<script setup>
// Tabs.vue
import { provide, ref } from 'vue'

const activeTab = ref(0)

provide('tabs', {
  activeTab,
  setActiveTab: (index) => activeTab.value = index
})
</script>

<template>
  <div class="tabs">
    <slot></slot>
  </div>
</template>

<!-- Tab.vue -->
<script setup>
import { inject, computed } from 'vue'

const props = defineProps({
  index: Number,
  label: String
})

const tabs = inject('tabs')
const isActive = computed(() => tabs.activeTab.value === props.index)
</script>

<template>
  <div 
    :class="{ active: isActive }"
    @click="tabs.setActiveTab(index)"
  >
    {{ label }}
    <slot v-if="isActive"></slot>
  </div>
</template>

<!-- Provide/Inject for Deep Props -->
<script setup>
// ThemeProvider.vue
import { provide, ref, readonly } from 'vue'

const theme = ref('light')

const toggleTheme = () => {
  theme.value = theme.value === 'light' ? 'dark' : 'light'
}

provide('theme', {
  theme: readonly(theme),
  toggleTheme
})
</script>

<!-- Any nested component -->
<script setup>
import { inject } from 'vue'

const { theme, toggleTheme } = inject('theme')
</script>

<!-- Teleport for Modals -->
<template>
  <Teleport to="body">
    <div v-if="isOpen" class="modal-overlay" @click="close">
      <div class="modal-content" @click.stop>
        <slot></slot>
      </div>
    </div>
  </Teleport>
</template>

<!-- Suspense for Async Components -->
<template>
  <Suspense>
    <template #default>
      <AsyncDashboard />
    </template>
    <template #fallback>
      <LoadingSpinner />
    </template>
  </Suspense>
</template>
```

</details>

### Vue.js Project Statistics

| Metric | Value |
|--------|-------|
| **Total Vue.js Projects** | 12+ |
| **Components Created** | 200+ |
| **Composables Developed** | 30+ |
| **NPM Packages Used** | 40+ |
| **Largest SPA** | 150+ components |
| **Performance Score** | 90+ (Lighthouse) |

---

## 📱 Flutter Excellence

### Flutter Architecture & Best Practices

<details>
<summary><b>🏗️ Clean Architecture with Bloc (Click to expand)</b></summary>

```dart
// Domain Layer - Entities
class User {
  final String id;
  final String name;
  final String email;
  
  User({required this.id, required this.name, required this.email});
}

// Domain Layer - Repository Interface
abstract class UserRepository {
  Future<Either<Failure, User>> getUser(String id);
  Future<Either<Failure, List<User>>> getUsers();
  Future<Either<Failure, User>> createUser(User user);
}

// Domain Layer - Use Cases
class GetUser {
  final UserRepository repository;
  
  GetUser(this.repository);
  
  Future<Either<Failure, User>> call(String id) async {
    return await repository.getUser(id);
  }
}

// Data Layer - Models
class UserModel extends User {
  UserModel({
    required super.id,
    required super.name,
    required super.email,
  });
  
  factory UserModel.fromJson(Map<String, dynamic> json) {
    return UserModel(
      id: json['id'],
      name: json['name'],
      email: json['email'],
    );
  }
  
  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'name': name,
      'email': email,
    };
  }
}

// Data Layer - Repository Implementation
class UserRepositoryImpl implements UserRepository {
  final UserRemoteDataSource remoteDataSource;
  final UserLocalDataSource localDataSource;
  final NetworkInfo networkInfo;
  
  UserRepositoryImpl({
    required this.remoteDataSource,
    required this.localDataSource,
    required this.networkInfo,
  });
  
  @override
  Future<Either<Failure, User>> getUser(String id) async {
    if (await networkInfo.isConnected) {
      try {
        final user = await remoteDataSource.getUser(id);
        await localDataSource.cacheUser(user);
        return Right(user);
      } on ServerException {
        return Left(ServerFailure());
      }
    } else {
      try {
        final user = await localDataSource.getCachedUser(id);
        return Right(user);
      } on CacheException {
        return Left(CacheFailure());
      }
    }
  }
}

// Presentation Layer - BLoC
class UserBloc extends Bloc<UserEvent, UserState> {
  final GetUser getUser;
  final CreateUser createUser;
  
  UserBloc({
    required this.getUser,
    required this.createUser,
  }) : super(UserInitial()) {
    on<GetUserEvent>(_onGetUser);
    on<CreateUserEvent>(_onCreateUser);
  }
  
  Future<void> _onGetUser(
    GetUserEvent event,
    Emitter<UserState> emit,
  ) async {
    emit(UserLoading());
    
    final result = await getUser(event.userId);
    
    result.fold(
      (failure) => emit(UserError(message: _mapFailureToMessage(failure))),
      (user) => emit(UserLoaded(user: user)),
    );
  }
  
  Future<void> _onCreateUser(
    CreateUserEvent event,
    Emitter<UserState> emit,
  ) async {
    emit(UserLoading());
    
    final result = await createUser(event.user);
    
    result.fold(
      (failure) => emit(UserError(message: _mapFailureToMessage(failure))),
      (user) => emit(UserCreated(user: user)),
    );
  }
}

// Presentation Layer - UI
class UserScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => getIt<UserBloc>()..add(GetUserEvent('123')),
      child: Scaffold(
        appBar: AppBar(title: Text('User Profile')),
        body: BlocBuilder<UserBloc, UserState>(
          builder: (context, state) {
            if (state is UserLoading) {
              return Center(child: CircularProgressIndicator());
            } else if (state is UserLoaded) {
              return UserProfile(user: state.user);
            } else if (state is UserError) {
              return Center(child: Text(state.message));
            }
            return Container();
          },
        ),
      ),
    );
  }
}

// Dependency Injection
final getIt = GetIt.instance;

void setupDependencies() {
  // BLoC
  getIt.registerFactory(() => UserBloc(
    getUser: getIt(),
    createUser: getIt(),
  ));
  
  // Use Cases
  getIt.registerLazySingleton(() => GetUser(getIt()));
  getIt.registerLazySingleton(() => CreateUser(getIt()));
  
  // Repository
  getIt.registerLazySingleton<UserRepository>(
    () => UserRepositoryImpl(
      remoteDataSource: getIt(),
      localDataSource: getIt(),
      networkInfo: getIt(),
    ),
  );
  
  // Data Sources
  getIt.registerLazySingleton<UserRemoteDataSource>(
    () => UserRemoteDataSourceImpl(client: getIt()),
  );
  
  getIt.registerLazySingleton<UserLocalDataSource>(
    () => UserLocalDataSourceImpl(hive: getIt()),
  );
}
```

</details>

<details>
<summary><b>🎨 Advanced UI Patterns (Click to expand)</b></summary>

```dart
// Custom Widgets
class ResponsiveBuilder extends StatelessWidget {
  final Widget Function(BuildContext, BoxConstraints) builder;
  
  const ResponsiveBuilder({required this.builder});
  
  @override
  Widget build(BuildContext context) {
    return LayoutBuilder(
      builder: (context, constraints) {
        return builder(context, constraints);
      },
    );
  }
}

// Responsive Design
class ResponsiveScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ResponsiveBuilder(
      builder: (context, constraints) {
        if (constraints.maxWidth < 600) {
          return MobileLayout();
        } else if (constraints.maxWidth < 1200) {
          return TabletLayout();
        } else {
          return DesktopLayout();
        }
      },
    );
  }
}

// Complex Animations
class AnimatedCard extends StatefulWidget {
  @override
  _AnimatedCardState createState() => _AnimatedCardState();
}

class _AnimatedCardState extends State<AnimatedCard>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _scaleAnimation;
  late Animation<double> _rotationAnimation;
  late Animation<Offset> _slideAnimation;
  
  @override
  void initState() {
    super.initState();
    
    _controller = AnimationController(
      duration: Duration(milliseconds: 800),
      vsync: this,
    );
    
    _scaleAnimation = Tween<double>(begin: 0.8, end: 1.0).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeOutBack),
    );
    
    _rotationAnimation = Tween<double>(begin: -0.1, end: 0.0).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeOut),
    );
    
    _slideAnimation = Tween<Offset>(
      begin: Offset(0, 0.3),
      end: Offset.zero,
    ).animate(
      CurvedAnimation(parent: _controller, curve: Curves.easeOut),
    );
    
    _controller.forward();
  }
  
  @override
  Widget build(BuildContext context) {
    return SlideTransition(
      position: _slideAnimation,
      child: RotationTransition(
        turns: _rotationAnimation,
        child: ScaleTransition(
          scale: _scaleAnimation,
          child: Card(
            child: /* Card Content */,
          ),
        ),
      ),
    );
  }
}

// Custom Painters
class CircularProgressPainter extends CustomPainter {
  final double progress;
  final Color backgroundColor;
  final Color progressColor;
  
  CircularProgressPainter({
    required this.progress,
    required this.backgroundColor,
    required this.progressColor,
  });
  
  @override
  void paint(Canvas canvas, Size size) {
    final center = Offset(size.width / 2, size.height / 2);
    final radius = min(size.width, size.height) / 2;
    
    // Background circle
    final backgroundPaint = Paint()
      ..color = backgroundColor
      ..style = PaintingStyle.stroke
      ..strokeWidth = 8.0;
    
    canvas.drawCircle(center, radius, backgroundPaint);
    
    // Progress arc
    final progressPaint = Paint()
      ..color = progressColor
      ..style = PaintingStyle.stroke
      ..strokeWidth = 8.0
      ..strokeCap = StrokeCap.round;
    
    canvas.drawArc(
      Rect.fromCircle(center: center, radius: radius),
      -pi / 2,
      2 * pi * progress,
      false,
      progressPaint,
    );
  }
  
  @override
  bool shouldRepaint(CircularProgressPainter oldDelegate) {
    return oldDelegate.progress != progress;
  }
}
```

</details>

### Flutter Project Statistics

| Metric | Value |
|--------|-------|
| **Total Flutter Apps** | 5+ |
| **App Downloads** | 15K+ |
| **Custom Widgets** | 80+ |
| **Performance (FPS)** | 60 FPS average |
| **Crash Rate** | < 0.5% |
| **User Rating** | 4.5+ stars |

---

## 💾 Database Expertise

### Advanced SQL Queries

<details>
<summary><b>📊 Complex Query Examples (Click to expand)</b></summary>

```sql
-- Window Functions for Analytics
SELECT 
    user_id,
    order_date,
    total_amount,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY order_date DESC) as order_rank,
    SUM(total_amount) OVER (PARTITION BY user_id) as user_lifetime_value,
    AVG(total_amount) OVER (PARTITION BY user_id) as avg_order_value,
    LAG(total_amount) OVER (PARTITION BY user_id ORDER BY order_date) as previous_order,
    LEAD(total_amount) OVER (PARTITION BY user_id ORDER BY order_date) as next_order
FROM orders
WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR);

-- Recursive CTE for Hierarchical Data
WITH RECURSIVE category_tree AS (
    -- Base case
    SELECT 
        id,
        name,
        parent_id,
        0 as level,
        CAST(name AS CHAR(1000)) as path
    FROM categories
    WHERE parent_id IS NULL
    
    UNION ALL
    
    -- Recursive case
    SELECT 
        c.id,
        c.name,
        c.parent_id,
        ct.level + 1,
        CONCAT(ct.path, ' > ', c.name)
    FROM categories c
    INNER JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT * FROM category_tree ORDER BY path;

-- Complex Join with Aggregations
SELECT 
    u.id as user_id,
    u.name,
    u.email,
    COUNT(DISTINCT o.id) as total_orders,
    SUM(o.total_amount) as lifetime_value,
    AVG(o.total_amount) as avg_order_value,
    MAX(o.created_at) as last_order_date,
    COUNT(DISTINCT oi.product_id) as unique_products_purchased,
    GROUP_CONCAT(DISTINCT c.name ORDER BY c.name SEPARATOR ', ') as purchased_categories
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
LEFT JOIN order_items oi ON o.id = oi.order_id
LEFT JOIN products p ON oi.product_id = p.id
LEFT JOIN categories c ON p.category_id = c.id
WHERE u.created_at >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR)
GROUP BY u.id, u.name, u.email
HAVING total_orders > 0
ORDER BY lifetime_value DESC
LIMIT 100;

-- Pivot Table Simulation
SELECT 
    product_id,
    product_name,
    SUM(CASE WHEN MONTH(order_date) = 1 THEN quantity ELSE 0 END) as Jan,
    SUM(CASE WHEN MONTH(order_date) = 2 THEN quantity ELSE 0 END) as Feb,
    SUM(CASE WHEN MONTH(order_date) = 3 THEN quantity ELSE 0 END) as Mar,
    SUM(CASE WHEN MONTH(order_date) = 4 THEN quantity ELSE 0 END) as Apr,
    SUM(CASE WHEN MONTH(order_date) = 5 THEN quantity ELSE 0 END) as May,
    SUM(CASE WHEN MONTH(order_date) = 6 THEN quantity ELSE 0 END) as Jun,
    SUM(quantity) as Total
FROM order_items oi
JOIN orders o ON oi.order_id = o.id
JOIN products p ON oi.product_id = p.id
WHERE YEAR(o.order_date) = YEAR(CURDATE())
GROUP BY product_id, product_name
ORDER BY Total DESC;

-- Advanced Subqueries
SELECT 
    p.*,
    (SELECT AVG(rating) FROM reviews WHERE product_id = p.id) as avg_rating,
    (SELECT COUNT(*) FROM reviews WHERE product_id = p.id) as review_count,
    (SELECT SUM(quantity) FROM order_items WHERE product_id = p.id) as total_sold,
    (
        SELECT JSON_ARRAYAGG(
            JSON_OBJECT(
                'id', i.id,
                'url', i.url,
                'alt', i.alt_text
            )
        )
        FROM product_images i
        WHERE i.product_id = p.id
        LIMIT 5
    ) as images
FROM products p
WHERE p.active = 1
ORDER BY (SELECT SUM(quantity) FROM order_items WHERE product_id = p.id) DESC;
```

</details>

### Database Optimization Techniques

```sql
-- Index Optimization
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);
CREATE INDEX idx_products_category_active ON products(category_id, active);
CREATE FULLTEXT INDEX idx_products_search ON products(name, description);

-- Composite Index
CREATE INDEX idx_orders_composite ON orders(user_id, status, created_at);

-- Covering Index
CREATE INDEX idx_orders_covering ON orders(user_id, status, total_amount, created_at);

-- Query Optimization
EXPLAIN SELECT * FROM orders WHERE user_id = 123;
EXPLAIN ANALYZE SELECT * FROM products WHERE category_id IN (1, 2, 3);

-- Partitioning
ALTER TABLE orders
PARTITION BY RANGE (YEAR(created_at)) (
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

---

## 📦 More Skills Documentation

[View complete skills documentation](./SKILLS_COMPLETE.md)

---

<div align="center">

**This is a comprehensive breakdown of my technical expertise.**  
**For project examples and more details, check out my [main profile](./README.md)**

[![Back to Main Profile](https://img.shields.io/badge/←_Back_to_Main_Profile-blue?style=for-the-badge)](./README.md)

</div>
