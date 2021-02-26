# Aligned and Expandable BoxLayout

Following the discussion in [#1840](https://github.com/fyne-io/fyne/pull/1840), cross axis alignment would be useful for `fyne.Layouts` to allow them be able to organize widgets with different cross axis sizes. Expandable feature is optional in this proposal but it is maybe useful as you can see later in this text.

## Glossary

MainAxis and CrossAxis:

| Axis      | VBox       | HBox       |
| --------- | ---------- | ---------- |
| MainAxis  | Vertical   | Horizontal |
| CrossAxis | Horizontal | Vertical   |

## Background

Currently, Fyne has two kinds of layout to work with horizontal and vertical alignment:

- `BoxLayouts` => `VBox` and `HBox`
- `GridLayouts` => `GridWithColumns` and `GridWithRows`

`BoxLayouts` resize objects to their min size in the main axis and stretch their cross axis size.

<img width="500" alt="HBox stretch cross axis" src="https://user-images.githubusercontent.com/12239342/106945280-9ce1da80-66f5-11eb-9d0b-f377a0c31d4e.png">
<img width="400" alt="VBox stretch cross axis" src="https://user-images.githubusercontent.com/12239342/106945448-ca2e8880-66f5-11eb-815e-97c11077db70.png">

`GridLayouts` resize objects to have the same size in the main axis and stretch their cross axis size.

<img width="500" alt="" src="https://user-images.githubusercontent.com/12239342/106946172-c51e0900-66f6-11eb-9980-4c5c62cc854b.png">
<img width="600" alt="" src="https://user-images.githubusercontent.com/12239342/106946339-fa2a5b80-66f6-11eb-8da3-d36faf1fff36.png">

---

None of them offers a way to align elements in the cross axis, although that behavior can be accomplished doing some workaround. For example if we want this result:

<img width="202" alt="" src="https://user-images.githubusercontent.com/12239342/106949421-ea147b00-66fa-11eb-9e45-b40d165e1851.png">

We should write something like:

```go
c := container.NewVBox(
	container.NewHBox(
		widget.NewEntry(),
		container.NewVBox(layout.NewSpacer(), createIcon(theme.ContentCopyIcon())),
		container.NewVBox(layout.NewSpacer(), createIcon(theme.ContentPasteIcon())),
		container.NewVBox(layout.NewSpacer(), createIcon(theme.ContentRemoveIcon())),
	),
)
```

Proposed solution would allow us to write:

```go
c := container.NewHBoxAligned(layout.CrossAlignmentEnd,
	widget.NewEntry(),
	createIcon(theme.ContentCopyIcon()),
	createIcon(theme.ContentPasteIcon()),
	createIcon(theme.ContentRemoveIcon()),
)
```

Now if we want to expand the entry to look like this:

<img width="200" alt="" src="https://user-images.githubusercontent.com/12239342/106950360-4b891980-66fc-11eb-90ce-9237f1d932f5.png">

We should write something like:

```go
c := container.NewVBox(
	container.NewBorder(nil, nil, nil,
		container.NewHBox(
			container.NewVBox(layout.NewSpacer(), createIcon(theme.ContentCopyIcon())),
			container.NewVBox(layout.NewSpacer(), createIcon(theme.ContentPasteIcon())),
			container.NewVBox(layout.NewSpacer(), createIcon(theme.ContentRemoveIcon())),
		),
		widget.NewEntry(),
	),
	layout.NewSpacer(),
)
```

This proposal would allow us to write:

```go
c := container.NewHBoxAligned(layout.CrossAlignmentEnd,
	container.NewHBoxExpanded(widget.NewEntry()),
	createIcon(theme.ContentCopyIcon()),
	createIcon(theme.ContentPasteIcon()),
	createIcon(theme.ContentRemoveIcon()),
)
```

It also would allow us to solve cross axis alignment problems for text-based widgets as discussed in [#1701](https://github.com/fyne-io/fyne/issues/1701) using the option `layout.CrossAlignmentBaseline`.

## Architecture / API

### 1. New `layout.CrossAlignment` type

There should be a new type called CrossAlignment (name suggestions are welcome!):

```go
// CrossAlignment defines cross axis alignment type.
type CrossAlignment int

// Cross Axis alignment options.
const (
	CrossAlignmentStart CrossAlignment = iota
	CrossAlignmentEnd
	CrossAlignmentCenter
	CrossAlignmentBaseline
	CrossAlignmentStretch
)
```

This type would help us to specify the cross axis alignment in box layouts.

- `CrossAlignmentStart`: Align the content to the Top (HBox) or Left (VBox).
- `CrossAlignmentEnd`: Align the content to the Bottom (HBox) or Right (VBox).
- `CrossAlignmentCenter`: Center the content vertically (HBox) or horizontally (VBox).
- `CrossAlignmentBaseline`: This option is only useful for HBox, it aligns the content to the text baseline. For VBox, this alignment would behave as CrossAlignmentStart.
- `CrossAlignmentStretch`: Stretch the cross axis size of the children (current behavior).

### 2. New container constructors

#### 2.1. First approach

- `container.NewVBox(objects...)`: It would work just as before (setting crossAlignment to CrossAlignmentStretch).
- `container.NewVBoxAligned(crossAlignment, objects...)`: It would work just like `NewVBox`, but instead of stretching the cross axis by default, user can set the desired cross alignment.
- `container.NewVBoxExpanded(objects...)`: It would resize the objects to have the same size in the vertical axis, and stretching their horizontal axis (setting crossAlignment to CrossAlignmentStretch).
- `container.NewVBoxExpandedAligned(crossAlignment, objects...)`: It would work just like `NewVBoxExpanded`, but instead of stretching the cross axis by default, user can set the desired cross alignment.

- `container.NewHBox(objects...)`: It would work just as before (setting crossAlignment to CrossAlignmentStretch).
- `container.NewHBoxAligned(crossAlignment, objects...)`: It would work just like `NewHBox`, but instead of stretching the cross axis by default, user can set the desired cross alignment.
- `container.NewHBoxExpanded(objects...)`: It would resize the objects to have the same size in the horizontal axis, and stretching their vertical axis (setting crossAlignment to CrossAlignmentStretch).
- `container.NewHBoxExpandedAligned(crossAlignment, objects...)`: It would work just like `NewHBoxExpanded`, but instead of stretching the cross axis by default, user can set the desired cross alignment.

#### 2.2. Second approach

First approach would introduce a lot of new container constructors that would be quite confusing. So a second approach would be to only have:

- `container.NewVBox(objects...)`
- `container.NewVBoxAligned(crossAlignment, objects...)`
- `container.NewHBox(objects...)`
- `container.NewHBoxAligned(crossAlignment, objects...)`

And have a way to specify when we want to expand an object in its main axis. There could be a new container called `ExpandedBox` and internally the boxLayout could detect it and expand its main axis size (just like the `spacers`). However, this new container would only be useful as a child of a VBox/HBox. Its definition could be:

- `container.NewExpandedBox`: It will expand the main axis of its children if its parent is a VBox/HBox, otherwise it will expand on both axis (something like MaxLayout).

#### 2.3. Third approach

Keep the four constructors of the Second approach and ignore the `expandable` feature.

## Implementation

The initial implementation can be found in (while this remains as proposal): https://github.com/fpabl0/fyne/tree/feature/aligned-expanded-boxlayout

This fork is not replacing current VBox and HBox layouts, in order to compare them. It adds a new layout `nboxLayout` to test this implementation (under `cmd/nboxlayout_demo`).

## Prior art

Most of the popular hybrid frameworks/toolkits for app development offer alignment options for elements on the two axis.

### 1. Web based - Ionic

Ionic has a grid layout called [`ion-grid`](https://ionicframework.com/docs/api/grid) that is defined as:

> a powerful mobile-first flexbox system for building custom layouts. It is composed of three units — a grid, row(s) and column(s). Columns will expand to fill the row, and will resize to fit additional columns.

So basically an `ion-grid` is composed by `ion-row`s and `ion-col`s. The alignment on main and cross axis is selected through css classes:

- [`.ion-justify-content-start` ...](https://ionicframework.com/docs/layout/css-utilities#flex-container-properties)
- [`.ion-align-items-start` ...](https://ionicframework.com/docs/layout/css-utilities#flex-container-properties)
- [`.ion-align-self-start` ...](https://ionicframework.com/docs/layout/css-utilities#flex-item-properties)

However, I think those css classes names are really confusing.

### 2. Flutter

Almost every Flutter app layout can be created using two (or three) main components (or widgets as Flutter calls them). These components are: [`Row`](https://api.flutter.dev/flutter/widgets/Row-class.html), [`Column`](https://api.flutter.dev/flutter/widgets/Column-class.html) and [`Expanded`](https://api.flutter.dev/flutter/widgets/Expanded-class.html). As you will notice, this proposal is mainly based on Flutter implementation.

- Row widget example:

```dart
Row(
  mainAxisAlignment: MainAxisAlignment.start,
  crossAxisAlignment: CrossAxisAlignment.center,
  children: [
    const FlutterLogo(),
    const Expanded(
      child: Text("Some text......."),
    ),
    const Icon(Icons.sentiment_very_satisfied),
  ],
)
```

- Column widget example:

```dart
Column(
  mainAxisAlignment: MainAxisAlignment.start,
  crossAxisAlignment: CrossAxisAlignment.start,
  children: [
    Text('We move under cover and we move as one'),
    Text('Through the night, we have one shot to live another day'),
    Text('We cannot let a stray gunshot give us away'),
    Text('We will fight up close, seize the moment and stay in it'),
    Text('It’s either that or meet the business end of a bayonet'),
    Text('The code word is ‘Rochambeau,’ dig me?'),
    Text('Rochambeau!'),
  ],
)
```
