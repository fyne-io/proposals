# Calling OS specific code

[Introduction]

This proposal covers improvements that will make it easier for developers to call into OS specific functionality.

## Background

The Go compiler with CGo functionality allows apps to call into native code on many platforms.
However for some systems it is essential to have an app context to run code (like Android and the JVM).
On other platforms it may be helpful to get the OS level app context (like macOS)

This proposal is a minimal approach to providing a way to execute native code with the appropriate context without leaking OS specific APIs.

## Architecture / API

The core API will provide a single entry point which allows context to be passed in:

```go
driver.RunNative(fn func(ctx interface{}) error)
```

By calling this function the developer can pass their own code (`fn`) which will be provided a context (`ctx`).
This context will then be cast (with a typecheck!) to a OS specific struct in the same package, such as:

```go
if osx, ok := ctx.(driver.MacOSContext); ok {
	C.doSomeLocalThingWith(osx.NSApp)
}
```

In that hypothetical example we see that a `MacOSContext` has a `NSApp` field that will hold an appropriate C type when running on a macOS computer.

## Implementation

It is expected that there will be many challenges during implementation due to pointer types and getting all of the context information encapsulated.
We may also find that some of the platform types could provide helper functions for common tasks (such as string conversion?) wrapping existing C functions.

The aim of this will be to help keep as much code in Go and provide type information to the maximum possible.

To illustrate a possible implementation for android the `driver` package would provide the following implementation inside a `runnative_android.go` file (each platform would provide its own version).

```go
type AndroidContext struct {
	VM, Env, Ctx uintptr
}

func RunNative(fn func(interface{}) error) {
	app.RunOnJVM(func(vm, env, ctx uintptr) error {
		data := &AndroidContext{VM: vm, Env: env, Ctx: ctx}
		return fn(data)
	}
}
```

Here our driver code is calling an internal `app` package to access OS specific calls and wrapping it into the generic function definition described earlier.

Plus the type defined could include things like the following:

```go
func (a *AndroidContext) JString(str string) {
	RunNative(func(ctx interface{}) error {
		cStr := C.CString(str)
		defer C.free(unsafe.Pointer(cStr))

		data := ctx.(AndroidContext)
		C.newJString(data.Env, cStr)
	}
}
```

This example relies on an internal C function `newJString` which would be defined elsewhere and uses the `RunNative` call and `AndroidContext.Env` to load a new java string from the JVM.
