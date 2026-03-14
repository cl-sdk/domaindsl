# domain dsl

This project provides a DSL for writing your domain types
and generate code for any target language (currently kotlin, swift, javascript).

It doesn't compile complicated or weird features of any language,
but, instead, use the common objects that gives approximately the same meaning
(structures, classes, data types).

## The idea

Using common lisp and its incredible development environment, we can get a lot of control over
what we want to do, like filtering what should be rendered, if I want to render directly to file,
or if I want to do some copy and paste.

Maybe, one day, we can manipulate the AST of each language directly.

![image](https://github.com/domaindsl/domaindsl/blob/development/extras/stateism.png?raw=true)

## Installation

This project uses [ASDF](https://asdf.common-lisp.dev/) for system management and
[Quicklisp](https://www.quicklisp.org/) for dependency management.

Clone the repository into your Quicklisp local projects directory:

```bash
git clone https://github.com/cl-sdk/domaindsl ~/quicklisp/local-projects/domaindsl
```

Then load the desired language systems in your Lisp image:

```lisp
(ql:quickload '(:domaindsl.kotlin :domaindsl.swift :domaindsl.javascript))
```

## Running tests

```bash
make tests
```

## Usage

### Declaring a data type

Use `datatype` to declare an algebraic data type. Each constructor can optionally
receive typed arguments:

```lisp
(defclass another-class () ())

(datatype my-type
  ((constructor-of-my-type-a)
   (constructor-of-my-type-b (:class another-class (:name dependency)))))
```

Each constructor argument is a list of `(kind type-name &optional opts)` where:

- `kind` is `:class` (for a Common Lisp class) or `:datatype` (for a previously declared `datatype`)
- `type-name` is the symbol naming the type
- `opts` is an optional plist that supports:
  - `:name <symbol>` — override the variable name used for the argument (defaults to the type name)
  - `:array t` — indicate the argument is an array/list of the given type

Example using `:array`:

```lisp
(defclass product () ())

(datatype order
  ((order-with-items (:class product (:name items :array t)))))
```

### Registering a standalone class

To make a plain Common Lisp class available as a top-level artifact (e.g. to
generate a corresponding class declaration in a target language), use
`register-class` after defining it with `defclass`:

```lisp
(defclass another-class () ())
(domaindsl.types:register-class 'another-class)
```

### Generating and compiling artifacts

```lisp
;; Generate intermediate artifact objects for a given target language
(domaindsl.artifact:generate-artifact :kotlin my-type-object)

;; Compile an artifact to its final source-code string
(domaindsl.artifact:compile-artifact :kotlin artifact)
```

Supported targets: `:kotlin`, `:swift`, `:javascript`.

### Generated output

Given the `my-type` / `another-class` example above, the following source code
is generated per target language:

**Swift**

```swift
class AnotherClass {}

enum MyType {
  case constructorOfMyTypeA
  case constructorOfMyTypeB(AnotherClass)
}
```

**Kotlin**

```kotlin
class AnotherClass{}

open class MyType

class ConstructorOfMyTypeA: MyType()

data class ConstructorOfMyTypeB(val dependency: AnotherClass): MyType()
```

**JavaScript**

```javascript
export function AnotherClass() {
  if (!(this instanceof AnotherClass)) { return new AnotherClass(); }
}

export function MyType() {}

export const ConstructorOfMyTypeA = new (function ConstructorOfMyTypeA() {})

export function ConstructorOfMyTypeB(dependency) {
  if (!(this instanceof ConstructorOfMyTypeB)) { return new ConstructorOfMyTypeB(dependency); }

  this.dependency = dependency;
}
```

## Example: Door State Machine

See [`examples/door-state-machine.lisp`](examples/door-state-machine.lisp) for a
complete working example. Running it produces the following artifacts for all three
targets:

```lisp
((((ARTIFACT :KOTLIN DOOR-EVENT "open class DoorEvent")
   (ARTIFACT :KOTLIN DOOR-EVENT "class OpenDoorEvent: DoorEvent()")
   (ARTIFACT :KOTLIN DOOR-EVENT "class CloseDoorEvent: DoorEvent()"))
  ((ARTIFACT :KOTLIN DOOR-STATE "open class DoorState")
   (ARTIFACT :KOTLIN DOOR-STATE "class OpenedDoor: DoorState()")
   (ARTIFACT :KOTLIN DOOR-STATE "class ClosedDoor: DoorState()")))
 (((ARTIFACT :JAVASCRIPT DOOR-EVENT "export function DoorEvent() {}")
   (ARTIFACT :JAVASCRIPT DOOR-EVENT
    "export const OpenDoorEvent = new (function OpenDoorEvent() {})")
   (ARTIFACT :JAVASCRIPT DOOR-EVENT
    "export const CloseDoorEvent = new (function CloseDoorEvent() {})"))
  ((ARTIFACT :JAVASCRIPT DOOR-STATE "export function DoorState() {}")
   (ARTIFACT :JAVASCRIPT DOOR-STATE
    "export const OpenedDoor = new (function OpenedDoor() {})")
   (ARTIFACT :JAVASCRIPT DOOR-STATE
    "export const ClosedDoor = new (function ClosedDoor() {})")))
 (((ARTIFACT :SWIFT DOOR-EVENT "enum DoorEvent {
  case openDoorEvent
  case closeDoorEvent
}"))
  ((ARTIFACT :SWIFT DOOR-STATE "enum DoorState {
  case openedDoor
  case closedDoor
}"))))
```
