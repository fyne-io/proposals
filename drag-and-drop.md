# Drag-adn-Drop

The proposal to provide native support Drag-and Drop opeerations:
* drag and drop between widgets 
* receive drag and drop URL(s) to application from a desktop or another application
* start drag operation from Fyne application with URL(s) to another desltop application (pending external changes)

## Background

There are few issues opened and related to missing drag and drop functionality:
* Add drag and drop support https://github.com/fyne-io/fyne/issues/142
* Drag and drop to application icon on macOS https://github.com/fyne-io/fyne/issues/497

Existing interface provide a minimum capabilities to programmatically start a drug operation and get notified when it complets.
This functionality doesn't provide an interface to communicate between widgets within the application nor send/receive drag and drop with other applications.

## Architecture / API

Existing interface will not change to preserve compatibility with existing code.
The proposed changes will extend the interface and give user missing capabilities.

The **fyne.CanvasObject** wishing to initiate drag operation must implement **fyne.Draggable** interface. No interface changes.

The **fyne.CanvasObject** wishing to receiver the drop must implement **fyne.DragReceiver** interface.

```
package fyne

type DragReceiver interface {
	DragDrop(Draggable)
}
```

To pass information to receiver the **fyne.Draggable** can also implement **drag.Info** interface.

```
package drag

type Info interface {
	Payload() Payload
}
```

To initiate a drag-and-drop outside the Fyne application the object implement **drag.AppDraggable** interface. This interface will allow the object prepare the **drag.Payload**.

```
package drag

type AppDraggable interface {
	// StartDrag return true when object ready to start drag-and-drop
	StartDrag() bool
	// Payload return Payload to attach to drag-and-drop session
	Payload() Payload
}
```

The initial implementations for **drag.Payload** will include **drag.PayloadString**, **drag.PayloadSliceOfString**. Other types can be added as needed and assuming the platform supports such type.

```
package drag

type Payload interface {}

type PayloadString string
type PayloadSliceOfString []string

```

To receive the drag-and-drop from outside the Fyne application the **fyne.CanvasObject** must implement the **fyne.DragReceiver** interface.

The **fyne.Draggable** can implement **drag.Paintable** interface to provide an image that can be used to paint a cursor during drag-and-drop operation.

```
package drag

type Paintable interface {
	CursorImage() image.Image
}

```

## Implementation

### Drag-and-Drop within the Fyne application

The drag and drop within the Fyne application is implemented by finding the component under the mouse pointer and supporting **fyne.DragReceiver** interface in each of the windows of application sorted by z-index. Only the first window with object of **fyne.DragReceiver** will get the call with fyne.Draggable.

The macOS implementation was tested to confirm this design.

This require for each platform to provide a window position, based on its order from front to back among all visible application windows.

The mobile implementation will be concern only with visible window.

### Drop from outside into Fyne application

The existing go-gl/glfw already provide facilities to receive the drop from another application.

The proposed implementation will utilize this functionality to deliver the drop directly to the object inplementing **fyne.DragReceiver** when found.

### Drag to outside of the Fyne application (pending external changes)

To implement the outside Drag-and-Drop the go-gl/glfw changes will be require as there is no support available for such operation.

The driver will check if object implement **drag.AppDraggable**. When object is ready to start a drag (StartDrag() call), the payload will be collected from the object and this information passed to the OS drag-and-drop mechaism.

#### Darwin

For Darwin platform the **NSDraggingDestination** interface already implemented and we can tap into the code to receive the URL pasteboard (already tested).
However, in order to initiate the drag from Fyne application to another application, the change will require in go-gl/glfw to implement **NSDraggingSource** interface methods in **GLFWContentView** (https://github.com/go-gl/glfw/blob/ea3d685f79fb0206999ffc85ca313ebf6f6b4c51/v3.3/glfw/glfw/src/cocoa_window.m#L629).

#### Windows

TBD - help wanted

#### Linux

TBD - help wanted

#### Mobile

No implementation for Mobile platform.


## Prior art

[Inclusion of any relevant information from other toolkits, libraries etc]