# Rich Text (plus refactoring and RTL/Intl support)

All of our text widgets need work to clean up, make more extensible and support rich text as well as future internationalisation.

## Background

The goal of this work is to develop a text handling solution that will have:

* Rich text support
* Allow developers to create custom widgets incorporating the text capability
* Good testability and performance
* Be more maintainable than the current state of text

The lead-up to this proposal has been discussed in various places

* https://github.com/fyne-io/fyne/wiki/Text-Refactor
* https://github.com/fyne-io/fyne/issues/21

## Architecture / API

* The BiDi/Shaping libraries
  - built in collaboration with Gio team will underpin the drawing elements of this work, but are not described in this proposal.
* RichText widget that encapsulates the parsing, styling and shaping for all our text requirements (see more in Design)
* textPresenter capabilities will be moved to RichText and the textProvider interface moved to configuration calls.
* To continue supporting widgets that expose a Text field we will support passing a *string or *[]rune into RichText, working around the previous need for copying and comparing large strings in Refresh.
* The Label widget will be re-worked to simply wrap a RichText.

Edit mode of rich text will be added at a later date, moving these capabilities out of the `Entry` widget and into the new `RichText`.

The proposed API for `RichText` follows the familiar pattern of standard components (rendered according to theme) that can be extended as required.
Each text segment can provide more than just text by providing a different graphical element from the `CanvasRepresentation` call.

```go
var (
	RichTextStyleInline RichTextStyle
	RichTextStyleParagraph RichTextStyle
	RichTextStyleHeading RichTextStyle
	RichTextStyleSubHeading RichTextStyle
	RichTextStyleQuote RichTextStyle
	RichTextStyleURL RichTextURL
	RichTextStyleEmphasis RichTextStyle
)

type RichText struct {
	BaseWidget

	Segments []RichTextSegment
}

type RichTextSegment interface {
	TextRepresentation() string
	CanvasRepresentation() fyne.CanvasObject
}

type TextSegment struct { // would implement RichTextSegment, handles rendering text
	Text string
	Type RichTextStyle
}

type RichTextStyle struct {
	Alignment fyne.TextAlign
	ColorName ThemeColorName
	Inline    bool
	TextSize  float32
	TextStyle fyne.TextStyle
}

type RichTextURL struct {
	RichTextStyle
	URL url.URL
}

type HorizontalRuleSegment struct { // would implement RichTextSegment, just draws a line
}
```

## Implementation

The updated `canvas.Text` will be ready for RTL and broader language support.
Each node will continue to support a single style, many of these elements will be combined in a complex layout to support the new `RichText` widget.

The heavy lifting will be covered by external libraries that will parse basic bi-directional text and render it accordingly.
We will need to add baseline measurement support into the toolkit (through `canvas.Text` or the `MeasureText` function) to align them accordingly.

The new shaper may not be ready for Fyne 2.1 but we need to build this refactoring so that it can be dropped in when ready (full internationalisation may not be part of Fyne until 3.0).


## Prior art

There is various prior art for the shaping / international components of the text work at [Text Refactor(https://github.com/fyne-io/fyne/wiki/Text-Refactor),
but the details of that work are not part of this proposal - it will be carried out in a [shared shaping library](https://github.com/go-text/shaping), [shared directionality library](https://github.com/go-text/di), and [shared font library](TBD).
