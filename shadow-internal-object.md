# Shadow internal state object

A few open issue require to be able to maintain internal state as a copy of the last Refresh call of the exported state. This include fixing race condition (to have a different data being accessed while rendering/checking input), memory leak both with the texture cache and the canvas object cache.

Maintaining an internal state associated with the canvas object would make it easier to keep all the information related to the rendering of an object. Ideally this should not be requiring to expose the painter internal.

## Background

The canvas and painter are insulated from the canvas object through an interface. There is no API for the canvas to attach an internal data to a canvas object or the reverse for a canvas object to find its internal data as seen from the painter. There is no way for the rendering thread or the input thread to access the canvas object data without race condition as there is no synchronization going on between those threads and the user threads modifying those object.

I can see two possible way to implement a solution to this problems. Either the canvas object get internal data attached from the canvas, or the canvas object request the associated internal data from the canvas. The code that call request to the canvas to get the associated internal data, doesn't need to be in the Refresh code of the widget, but at minima it need to be triggered every time canvas.Refresh is called and make sure that the data in the object are the one set on the internal state.

Considering that the data to associate will be different per type of widget and we likely do not want to multiply entry point, we will likely have to rely on interfaces. As resolving interfaces for each Refresh will be in the hot path, it would be best to move that out of the way.

Also it is to be noted that call to canvasObject.Refresh(), even if the documentation says "you must call it" is not enforced and only canvas.Refresh() has been enforced in practice.

## Architecture / API

To avoid the cost of interface resolution to happen all the time, attaching the internal state once with a Register/Unregister pattern does remove the interface translation cost. The CanvasObject on Register can cast the passed interface once, get the right internal type and store it after that cast. With this in mind adding the following function to CanvasObject would solve this needs:

```
// CanvasRegister is called by the canvas when a canvas object is added to a canvas
CanvasRegister(canvas fyne.Canvas, state internal.PainterBase)
// CanvasUnregister is called by the canvas when a canvas object is removed from a canvas
CanvasUnregister()
// CanvasSynchronize is called when an object Refresh function is called and the internal state given during register must be updated
CanvasSynchronize()
// CanvasInternalPainter return the internal painter currently associated with this CanvasObject
CanvasInternalPainter() internal.PainterBase
```

`internal.PainterBase` will be an interface with the following function:
```
type PainterBase interface {
    Draw(pos fyne.Position, frame fyne.Size)
    Canvas() fyne.Canvas
}
```

The `internal.PainterBase` matching the internal state for `canvas.Image`, `canvas.Text`, `canvas.*Gradient`, `canvas.Circle`, `canvas.Line`, `canvas.Raster` and `canvas.Rectangle` should be created by the Painter associated with the Window Canvas when an object is first seen during a WalkTree. As `fyne.Container` and `fyne.Widget` are also used by the rendering thread and the input thread, it is also necessary to synchronize them. Their internal state could be handle at a different level than the Painter, but as it would be nice to think of using PBO to cache `fyne.Widget` that are scrolling, it might be best to add those object in the Painter directly. This open the interesting question of wether the Draw function on a `fyne.Container`/`fyne.Widget` shadow internal state object actually does the walk and draw.

WalkTree and friends should use the internal.PainterBase of `fyne.Container`/`fyne.Widget`. It might be that WalkTree is only useful after this change for input handling and not anymore for rendering purpose.

I would expect the following `type struct` being added to the Painter:
- `painter.Image`
- `painter.Text`
- `painter.LinearGradient`
- `painter.RadialGradient`
- `painter.Circle`
- `painter.Line`
- `painter.Raster`
- `painter.Rectangle`
- `painter.Container`
