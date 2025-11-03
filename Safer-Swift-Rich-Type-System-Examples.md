# Examples of using Swift's richer type system to avoid runtime problems

See also the broader [guidelines for Safer Swift](Safer-Swift-Guidelines).

Here are some examples of how runtime problem cases can be prevented at compile time.

### Use enum to tie state to state-dependent data

For example, a C++ state machine might look like this (simplified):

```cpp
enum State {
  First,
  Second
}

class StateMachine {
  State m_state;
  std::string m_string_relevant_only_in_first_state;
  RefPtr<Thingy> m_thingy_relevant_only_in_second_state;
}
```

In Swift, you can eliminate any runtime risk of the state getting out of sync with the per-state data:

```swift
enum StateMachine {
  case First(String)
  case Second(Thingy)
}
```

### Use exclusive access to tie dependent data together

```swift
struct ChildrenData {
  private var children: UnsafeMutablePointer<TreeNode>?
  private var childCount: Int

  // init() omitted for brevity

  mutating func addChild(_ child: TreeNode) {
    // Can modify children and childCount
    // Guaranteed no code can be in the middle of 'iterateChildren'
    // Exclusive access ensures children and childCount stay synchronized
  }

  func iterateChildren() {
    // Non-mutating access - can be called concurrently with other non-mutating methods
    // Can read but not modify children and childCount
  }
}
```

### Use phantom types to force state

```swift
// State types (no runtime storage)
struct Closed {}
struct Open {}

struct FileHandle<State> {
  private let fd: Int32

  private init(fd: Int32) {
    self.fd = fd
  }

  // Can only create closed files
  static func create(path: String) -> FileHandle<Closed> {
    // open file into FD
    return FileHandle<Closed>(fd: fd)
  }
}

extension FileHandle where State == Closed {
  func open() -> FileHandle<Open> {
    // Transition to open state
    return FileHandle<Open>(fd: self.fd)
  }
}

extension FileHandle where State == Open {
  func write(_ data: String) {
    // Can ONLY call this on open files - compile-time guaranteed!
    // ... write the data
  }

  func close() -> FileHandle<Closed> {
    // ... close the file
    return FileHandle<Closed>(fd: -1)
  }
}

// Usage - impossible to write to closed file
let file = FileHandle.create(path: "/tmp/test")
// file.write("data")  // Compile error!
let openFile = file.open()
openFile.write("data")  // OK
let closedFile = openFile.close()
// closedFile.write("more")  // Compile error!
```

You can extend this sort of pattern in all sorts of ways, e.g. encoding capabilities into the type system such that (for example) some database connections have write permissions but others only read permission.

### Use actors for thread safety

```swift
actor Counter {
  private var value = 0

  func increment() {
      value += 1  // Automatically thread-safe
  }

  func get() -> Int {
      return value
  }
}

// Usage
let counter = Counter()

Task {
  await counter.increment()  // Must await - can't forget synchronization
  let val = await counter.get()
}
```

`@MainActor` can ensure that all UI updates are guaranteed to occur on the main thread.

```swift
@MainActor
class ViewController {
  func updateLabel(_ text: String) {
    // Guaranteed to be on main thread
  }
}

func backgroundWork() {
  Task {
    // Background work
    let result = await heavyComputation()

    // Must explicitly hop to main actor
    await MainActor.run {
      viewController.updateLabel(result)
    }

    // Or mark the method call:
    // await viewController.updateLabel(result)  // Automatic main thread dispatch
  }
}
```

### Encode results in an enum to force callers to handle all cases

Swift `switch` statements need to be exhaustive. You can use that to be sure your caller handles all possible states.

```swift
enum ParseError: Error {
  case invalidFormat
  case outOfRange
}

func parseNumber(_ str: String) -> Result<Int, ParseError> {
  // ...
}

func usage() {
  switch parseNumber("invalid") {  // Must handle all possible
                                   // return cases
  case .success(let number):
      print("Got number: \(number)")
  case .failure(.invalidFormat):
      print("Invalid format")
  case .failure(.outOfRange):
      print("Out of range")
  }
}
```

## Use protocols to prove capabilities at compile-time

```swift
protocol Drawable {
  func draw()
}

protocol Transformable {
  func rotate(_ angle: Float)
  func scale(_ factor: Float)
}

struct Image: Drawable {
  func draw() { /* render bitmap */ }
}

struct Shape: Drawable, Transformable {
  func draw() { /* render vector shape */ }
  func rotate(_ angle: Float) { /* apply rotation */ }
  func scale(_ factor: Float) { /* apply scaling */ }
}

// Functions specify exactly what they need
func renderToScreen<T: Drawable>(_ object: T) {
  object.draw()
  // CANNOT accidentally call rotate() - might not be transformable
}

func createAnimation<T: Drawable & Transformable>(_ object: T) {
  object.rotate(45.0)
  object.draw()
}

// Usage
let photo = Image()
let circle = Shape()

renderToScreen(photo)   // OK
renderToScreen(circle)  // OK

createAnimation(circle) // OK
// createAnimation(photo)  // Compile error - Image isn't Transformable
```

### Use structs to encode the meaning of primitives

```swift
struct Meters {
    let value: Double
}

struct Feet {
    let value: Double
}

func calculateArea(width: Meters, height: Meters) -> Double {
    return width.value * height.value
}

// Usage
let widthInMeters = Meters(value: 10.0)
let heightInFeet = Feet(value: 15.0)

calculateArea(width: widthInMeters, height: widthInMeters)  // OK
// calculateArea(width: widthInMeters, height: heightInFeet)  // Compile error!
```
