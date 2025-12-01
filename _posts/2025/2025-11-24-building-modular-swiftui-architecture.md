---
title: "Building Modular SwiftUI Architecture"
date: 2025-11-24T08:00:00 +0200
layout: post
tags: [modular architecture,di,scalable swiftui]
---

## Building Modular SwiftUI Architecture: From Basics to Advanced Patterns

Creating maintainable and **scalable SwiftUI** applications requires more than just understanding views and modifiers. This comprehensive guide explores **SwiftUI architecture** patterns that help you build robust, testable, and maintainable iOS applications. Whether you're transitioning from UIKit or starting fresh with SwiftUI, you'll learn how to structure your apps using **modular architecture** principles and dependency injection to create truly professional-grade applications.

### Prerequisites

- Xcode 14.0 or later installed on your Mac
- Basic understanding of Swift programming language
- Familiarity with SwiftUI fundamentals (Views, State, Binding)
- iOS 15.0+ deployment target knowledge
- Understanding of object-oriented programming concepts

### What You'll Learn

- How to structure a SwiftUI app using **modular architecture** principles
- Implementing dependency injection (DI) in SwiftUI applications
- Creating reusable and testable view components
- Building a scalable navigation system
- Separating business logic from UI using ViewModels
- Managing app-wide state effectively
- Writing unit tests for your architecture components

### A Step-by-Step Guide to Building Your First Modular SwiftUI App

### Step 1: Setting Up Your Project Structure

Before diving into code, let's establish a solid foundation for our **SwiftUI architecture**. A well-organized project structure is crucial for maintainability and team collaboration.

```swift
// Project Structure
MyApp/
├── App/
│   ├── MyApp.swift
│   └── AppDelegate.swift
├── Core/
│   ├── DI/
│   │   ├── Container.swift
│   │   └── Dependencies.swift
│   ├── Navigation/
│   │   └── NavigationCoordinator.swift
│   └── Networking/
│       └── NetworkService.swift
├── Features/
│   ├── Home/
│   │   ├── Views/
│   │   ├── ViewModels/
│   │   └── Models/
│   └── Profile/
│       ├── Views/
│       ├── ViewModels/
│       └── Models/
└── Shared/
    ├── Components/
    ├── Extensions/
    └── Resources/
```

This structure separates concerns into distinct modules. The **Core** folder contains fundamental services, **Features** houses individual app features, and **Shared** includes reusable components. Create these folders in your Xcode project to establish this **modular architecture**.

### Step 2: Implementing Dependency Injection Container

Dependency injection is essential for creating testable and flexible code. Let's build a simple yet powerful **DI** container for our SwiftUI app.

```swift
// Container.swift
import SwiftUI

protocol DIContainer {
    associatedtype Dependencies
    var dependencies: Dependencies { get }
}

struct AppContainer: DIContainer {
    let dependencies: AppDependencies
    
    init() {
        self.dependencies = AppDependencies()
    }
}

// Dependencies.swift
class AppDependencies {
    lazy var networkService: NetworkServiceProtocol = NetworkService()
    lazy var userRepository: UserRepositoryProtocol = UserRepository(networkService: networkService)
    lazy var authService: AuthServiceProtocol = AuthService(networkService: networkService)
    
    // Factory methods for ViewModels
    func makeHomeViewModel() -> HomeViewModel {
        return HomeViewModel(userRepository: userRepository)
    }
    
    func makeProfileViewModel() -> ProfileViewModel {
        return ProfileViewModel(
            userRepository: userRepository,
            authService: authService
        )
    }
}

// Environment injection
struct DIContainerKey: EnvironmentKey {
    static let defaultValue = AppContainer()
}

extension EnvironmentValues {
    var diContainer: AppContainer {
        get { self[DIContainerKey.self] }
        set { self[DIContainerKey.self] = newValue }
    }
}
```

This code creates a centralized dependency container that manages all app dependencies. The `AppDependencies` class uses lazy initialization to create services only when needed. The environment key allows us to inject the container throughout the SwiftUI view hierarchy.

### Step 3: Creating the Network Layer

A robust networking layer is fundamental to any modern app. Let's build a protocol-based networking service that's easy to test and extend.

```swift
// NetworkService.swift
import Foundation
import Combine

protocol NetworkServiceProtocol {
    func request<T: Decodable>(_ endpoint: Endpoint) -> AnyPublisher<T, NetworkError>
}

enum NetworkError: Error {
    case invalidURL
    case noData
    case decodingError
    case serverError(statusCode: Int)
}

struct Endpoint {
    let path: String
    let method: HTTPMethod
    let parameters: [String: Any]?
    let headers: [String: String]?
    
    enum HTTPMethod: String {
        case get = "GET"
        case post = "POST"
        case put = "PUT"
        case delete = "DELETE"
    }
}

class NetworkService: NetworkServiceProtocol {
    private let baseURL = "https://api.example.com"
    private let session: URLSession
    
    init(session: URLSession = .shared) {
        self.session = session
    }
    
    func request<T: Decodable>(_ endpoint: Endpoint) -> AnyPublisher<T, NetworkError> {
        guard let url = URL(string: baseURL + endpoint.path) else {
            return Fail(error: NetworkError.invalidURL)
                .eraseToAnyPublisher()
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = endpoint.method.rawValue
        request.allHTTPHeaderFields = endpoint.headers
        
        if let parameters = endpoint.parameters,
           endpoint.method != .get {
            request.httpBody = try? JSONSerialization.data(withJSONObject: parameters)
            request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        }
        
        return session.dataTaskPublisher(for: request)
            .map(\.data)
            .decode(type: T.self, decoder: JSONDecoder())
            .mapError { error in
                if error is DecodingError {
                    return NetworkError.decodingError
                }
                return NetworkError.noData
            }
            .eraseToAnyPublisher()
    }
}
```

This networking layer uses Combine for reactive programming and protocols for testability. The `Endpoint` structure encapsulates all request details, making it easy to define and reuse API endpoints throughout your app.

### Step 4: Building Feature Modules with MVVM

Let's create a complete feature module using the MVVM pattern, demonstrating how to structure **scalable SwiftUI** features.

```swift
// HomeViewModel.swift
import SwiftUI
import Combine

@MainActor
class HomeViewModel: ObservableObject {
    @Published var users: [User] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    private let userRepository: UserRepositoryProtocol
    private var cancellables = Set<AnyCancellable>()
    
    init(userRepository: UserRepositoryProtocol) {
        self.userRepository = userRepository
    }
    
    func loadUsers() {
        isLoading = true
        errorMessage = nil
        
        userRepository.fetchUsers()
            .receive(on: DispatchQueue.main)
            .sink(
                receiveCompletion: { [weak self] completion in
                    self?.isLoading = false
                    if case .failure(let error) = completion {
                        self?.errorMessage = error.localizedDescription
                    }
                },
                receiveValue: { [weak self] users in
                    self?.users = users
                }
            )
            .store(in: &cancellables)
    }
}

// HomeView.swift
struct HomeView: View {
    @StateObject private var viewModel: HomeViewModel
    @Environment(\.diContainer) private var container
    
    init() {
        let container = AppContainer()
        self._viewModel = StateObject(
            wrappedValue: container.dependencies.makeHomeViewModel()
        )
    }
    
    var body: some View {
        NavigationView {
            Group {
                if viewModel.isLoading {
                    ProgressView()
                        .frame(maxWidth: .infinity, maxHeight: .infinity)
                } else if let error = viewModel.errorMessage {
                    ErrorView(message: error) {
                        viewModel.loadUsers()
                    }
                } else {
                    UserListView(users: viewModel.users)
                }
            }
            .navigationTitle("Users")
            .onAppear {
                viewModel.loadUsers()
            }
        }
    }
}

// Reusable Components
struct UserListView: View {
    let users: [User]
    
    var body: some View {
        List(users) { user in
            UserRow(user: user)
        }
    }
}

struct UserRow: View {
    let user: User
    
    var body: some View {
        HStack {
            AsyncImage(url: URL(string: user.avatarURL)) { image in
                image
                    .resizable()
                    .aspectRatio(contentMode: .fit)
            } placeholder: {
                ProgressView()
            }
            .frame(width: 50, height: 50)
            .clipShape(Circle())
            
            VStack(alignment: .leading) {
                Text(user.name)
                    .font(.headline)
                Text(user.email)
                    .font(.caption)
                    .foregroundColor(.secondary)
            }
            
            Spacer()
        }
        .padding(.vertical, 4)
    }
}
```

This implementation showcases a complete feature module with proper separation of concerns. The ViewModel handles business logic and data fetching, while the View focuses solely on presentation. Small, reusable components like `UserRow` promote code reuse across your app.

### Step 5: Implementing Navigation Coordination

Navigation in **modular architecture** requires careful planning. Let's build a coordinator pattern for SwiftUI navigation.

```swift
// NavigationCoordinator.swift
import SwiftUI

enum NavigationDestination: Hashable {
    case home
    case profile(userId: String)
    case settings
    case detail(item: DetailItem)
}

struct DetailItem: Hashable {
    let id: String
    let title: String
}

class NavigationCoordinator: ObservableObject {
    @Published var path = NavigationPath()
    
    func navigate(to destination: NavigationDestination) {
        path.append(destination)
    }
    
    func navigateBack() {
        if !path.isEmpty {
            path.removeLast()
        }
    }
    
    func navigateToRoot() {
        path = NavigationPath()
    }
}

// RootView.swift
struct RootView: View {
    @StateObject private var coordinator = NavigationCoordinator()
    @Environment(\.diContainer) private var container
    
    var body: some View {
        NavigationStack(path: $coordinator.path) {
            HomeView()
                .navigationDestination(for: NavigationDestination.self) { destination in
                    destinationView(for: destination)
                }
        }
        .environmentObject(coordinator)
    }
    
    @ViewBuilder
    private func destinationView(for destination: NavigationDestination) -> some View {
        switch destination {
        case .home:
            HomeView()
        case .profile(let userId):
            ProfileView(userId: userId)
        case .settings:
            SettingsView()
        case .detail(let item):
            DetailView(item: item)
        }
    }
}
```

This navigation coordinator provides centralized navigation logic, making it easy to navigate programmatically from any part of your app. The pattern scales well as your app grows and supports deep linking naturally.

### Step 6: Managing Global State

For app-wide state management, let's implement a store pattern that works seamlessly with SwiftUI's reactive system.

```swift
// AppStore.swift
import SwiftUI
import Combine

@MainActor
class AppStore: ObservableObject {
    @Published var user: User?
    @Published var isAuthenticated = false
    @Published var appTheme: AppTheme = .system
    @Published var notifications: [AppNotification] = []
    
    private let authService: AuthServiceProtocol
    private var cancellables = Set<AnyCancellable>()
    
    init(authService: AuthServiceProtocol) {
        self.authService = authService
        setupBindings()
    }
    
    private func setupBindings() {
        authService.authStatePublisher
            .receive(on: DispatchQueue.main)
            .sink { [weak self] authState in
                self?.isAuthenticated = authState.isAuthenticated
                self?.user = authState.user
            }
            .store(in: &cancellables)
    }
    
    func login(email: String, password: String) async throws {
        let user = try await authService.login(email: email, password: password)
        self.user = user
        self.isAuthenticated = true
    }
    
    func logout() {
        authService.logout()
        user = nil
        isAuthenticated = false
    }
    
    func updateTheme(_ theme: AppTheme) {
        appTheme = theme
        UserDefaults.standard.set(theme.rawValue, forKey: "AppTheme")
    }
}

enum AppTheme: String, CaseIterable {
    case light = "Light"
    case dark = "Dark"
    case system = "System"
    
    var colorScheme: ColorScheme? {
        switch self {
        case .light: return .light
        case .dark: return .dark
        case .system: return nil
        }
    }
}

// App.swift integration
@main
struct MyApp: App {
    @StateObject private var appStore: AppStore
    private let container: AppContainer
    
    init() {
        let container = AppContainer()
        self.container = container
        self._appStore = StateObject(
            wrappedValue: AppStore(authService: container.dependencies.authService)
        )
    }
    
    var body: some Scene {
        WindowGroup {
            RootView()
                .environment(\.diContainer, container)
                .environmentObject(appStore)
                .preferredColorScheme(appStore.appTheme.colorScheme)
        }
    }
}
```

This global state management approach provides a single source of truth for app-wide data while maintaining reactivity. The store pattern integrates well with SwiftUI's environment system and remains testable through dependency injection.

### Step 7: Writing Unit Tests

Testing is crucial for maintaining a robust **SwiftUI architecture**. Let's write comprehensive tests for our components.

```swift
// HomeViewModelTests.swift
import XCTest
import Combine
@testable import MyApp

class HomeViewModelTests: XCTestCase {
    var viewModel: HomeViewModel!
    var mockRepository: MockUserRepository!
    var cancellables: Set<AnyCancellable>!
    
    override func setUp() {
        super.setUp()
        mockRepository = MockUserRepository()
        viewModel = HomeViewModel(userRepository: mockRepository)
        cancellables = Set<AnyCancellable>()
    }
    
    override func tearDown() {
        viewModel = nil
        mockRepository = nil
        cancellables = nil
        super.tearDown()
    }
    
    func testLoadUsersSuccess() {
        // Given
        let expectedUsers = [
            User(id: "1", name: "John", email: "john@example.com", avatarURL: ""),
            User(id: "2", name: "Jane", email: "jane@example.com", avatarURL: "")
        ]
        mockRepository.usersToReturn = expectedUsers
        
        let expectation = XCTestExpectation(description: "Users loaded")
        
        // When
        viewModel.$users
            .dropFirst() // Skip initial empty value
            .sink { users in
                // Then
                XCTAssertEqual(users.count, expectedUsers.count)
                XCTAssertEqual(users.first?.name, "John")
                expectation.fulfill()
            }
            .store(in: &cancellables)
        
        viewModel.loadUsers()
        
        wait(for: [expectation], timeout: 2.0)
    }
    
    func testLoadUsersFailure() {
        // Given
        mockRepository.shouldFail = true
        let expectation = XCTestExpectation(description: "Error received")
        
        // When
        viewModel.$errorMessage
            .compactMap { $0 }
            .sink { errorMessage in
                // Then
                XCTAssertFalse(errorMessage.isEmpty)
                expectation.fulfill()
            }
            .store(in: &cancellables)
        
        viewModel.loadUsers()
        
        wait(for: [expectation], timeout: 2.0)
    }
}

// MockUserRepository.swift
class MockUserRepository: UserRepositoryProtocol {
    var usersToReturn: [User] = []
    var shouldFail = false
    
    func fetchUsers() -> AnyPublisher<[User], Error> {
        if shouldFail {
            return Fail(error: NetworkError.noData)
                .eraseToAnyPublisher()
        }
        
        return Just(usersToReturn)
            .setFailureType(to: Error.self)
            .delay(for: .milliseconds(100), scheduler: RunLoop.main)
            .eraseToAnyPublisher()
    }
}
```

These tests demonstrate how **modular architecture** enables easy testing through dependency injection. Mock objects simulate external dependencies, allowing you to test components in isolation.

### Common Errors and How to Fix Them

**Error 1: "Cannot find type 'AppContainer' in scope"**

This error occurs when the dependency injection container isn't properly imported or initialized. Make sure you've created the `Container.swift` file in the correct location and imported it where needed:

```swift
// Solution: Import the module and ensure proper initialization
import SwiftUI

struct ContentView: View {
    @Environment(\.diContainer) private var container // Ensure DIContainerKey is defined
    
    var body: some View {
        // Your view code
    }
}
```

**Error 2: "Publishing changes from background threads is not allowed"**

SwiftUI requires UI updates to happen on the main thread. This error appears when updating `@Published` properties from background queues:

```swift
// Solution: Use @MainActor or receive on main queue
userRepository.fetchUsers()
    .receive(on: DispatchQueue.main) // Add this line
    .sink(receiveCompletion: { completion in
        // Handle completion
    }, receiveValue: { [weak self] users in
        self?.users = users // Now safe to update
    })
    .store(in: &cancellables)
```

**Error 3: "Memory leak detected in View"**

This happens when creating strong reference cycles, especially with closures in ViewModels:

```swift
// Solution: Use [weak self] in closures
somePublisher
    .sink { [weak self] value in // Add [weak self]
        guard let self = self else { return }
        self.updateValue(value)
    }
    .store(in: &cancellables)
```

### Next Steps and Real-World Applications

Now that you've built a solid foundation with **modular architecture**, consider expanding your app with these advanced features:

- **Implement offline caching** using Core Data or SQLite to persist data locally
- **Add deep linking support** to navigate directly to specific screens from URLs
- **Integrate analytics** to track user behavior and app performance
- **Implement feature flags** to gradually roll out new features
- **Add localization** to support multiple languages

Real-world applications using similar architecture patterns include:

- **Banking apps** that require secure, modular feature development
- **E-commerce platforms** with complex navigation and state management
- **Social media apps** that need scalable, maintainable codebases
- **Enterprise applications** requiring testable and maintainable architecture

### Essential Tools and Further Learning

**Official Resources:**
- [Apple's SwiftUI Documentation](https://developer.apple.com/documentation/swiftui)
- [Swift Evolution Proposals](https://github.com/apple/swift-evolution)
- [WWDC SwiftUI Sessions](https://developer.apple.com/videos/swiftui)

**Architecture Libraries:**
- [The Composable Architecture (TCA)](https://github.com/pointfreeco/swift-composable-architecture) - A library for building applications in a consistent and understandable way
- [SwiftUI Navigation](https://github.com/pointfreeco/swiftui-navigation) - Tools for making SwiftUI navigation simpler and more correct
- [Resolver](https://github.com/hmlongco/Resolver) - A lightweight dependency injection framework

**Testing Tools:**
- [ViewInspector](https://github.com/nalexn/ViewInspector) - Runtime introspection and unit testing of SwiftUI views
- [SnapshotTesting](https://github.com/pointfreeco/swift-snapshot-testing) - Delightful Swift snapshot testing

### FAQ

**Q: When should I use MVVM vs MVC in SwiftUI?**

A: MVVM is generally preferred for SwiftUI applications because it aligns naturally with SwiftUI's declarative nature and reactive data flow. The ViewModel serves as an `ObservableObject` that SwiftUI can observe for changes, making state management more predictable. MVC can still work but often leads to massive view controllers and tighter coupling between components.

**Q: How do I handle complex navigation flows in modular SwiftUI apps?**

A: Use a coordinator pattern combined with SwiftUI's `NavigationStack` (iOS 16+) or `NavigationView` for older versions. Create a centralized `NavigationCoordinator` that manages the navigation state and define your destinations as an enum. This approach scales well and supports programmatic navigation, deep linking, and testing.

**Q: Is dependency injection really necessary for small SwiftUI projects?**

A: While not strictly necessary for small projects, implementing basic **DI** from the start pays dividends as your app grows. It makes testing easier, reduces coupling between components, and allows you to swap implementations easily. Even a simple container pattern like the one shown in this tutorial adds minimal complexity while providing significant benefits.

### Conclusion

You've successfully built a professional-grade **SwiftUI architecture** that scales from simple prototypes to complex production applications. By implementing **modular architecture** with proper separation of concerns, dependency injection, and comprehensive testing, you've created a maintainable foundation that will serve your app well as it grows.

The patterns and techniques covered here represent industry best practices used by leading iOS development teams worldwide. Take time to experiment with the code examples, adapt them to your specific needs, and remember that good architecture evolves with your understanding and requirements.

Ready to put these concepts into practice? Start by refactoring an existing SwiftUI project using these patterns, or begin your next app with this solid architectural foundation. For more advanced iOS development techniques and architectural patterns, explore our other technical guides and stay updated with the latest SwiftUI innovations.
