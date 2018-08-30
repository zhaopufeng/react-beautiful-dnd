# Changes during a drag

You are welcome to make whatever changes you would like when a drag is not occurring. During a drag this is more limited:

✅ Add and remove no `Draggable` or `Droppable` during a drag

Generally we advise not adding or removing any `Draggable` component during a drag. This is fairly advanced behavior.

## Background

When a drag starts we capture the dimensions of everything relevant to the drag. We use this to create our _virtual dimension model_. When then use this model as the basis for our location calculations. This is what allows us to have beautiful dragging with lots of input types

## Supported changes during a drag

During a drag, `react-beautiful-dnd` supports the addition or removal a `Draggable` in a `Droppable` that is or has a scroll container. We will patch the _virtual dimension model_ with the impact of these changes.

### `DragDropContext > onDragUpdate` behavior

- `onDragUpdate` will be called if the `DragUpdate > source > index` of the dragging item has changed as the result of `Draggables` being added or removed before it.
- `onDragUpdate` will be called if the `DragUpdate > destination` of the dragging item has changed as a result of the addition or removal.

### `DragDropContext > onDragEnd` behavior

`onDragEnd` will be called with values that are adjusted for any additions or removals of `Draggables` during a drag. This can mean that the `DragStart > source > index` can be different from the `DropResult > source > index`.

#### Sample `onDragEnd` flow

> What is important to note is that the `source` property can change during a drag as a result of dynamic changes.

1. A drag starts.

`onDragStart` is called with:

```js
{
  draggableId: 'item-1',,
  type: 'TYPE',
  source: {
    droppableId: 'droppable',
    index: 1,
  },
}
```

2. The first `Draggable` in the list (`item-0`) is removed.

`onDragUpdate` is called with `DragUpdate`:

```diff
{
  draggableId: 'item-1',,
  type: 'TYPE',
  source: {
    droppableId: 'droppable',
+    index: 0,
  },
  // adjusted destination
  destination: null,
}
```

3. The drag ends

`onDragEnd` is called with `DropResult`:

```diff
{
  draggableId: 'item-1',,
  type: 'TYPE',
  source: {
    droppableId: 'droppable',
+    index: 0,
  },
  destination: null,
  reason: 'DROP',
}
```

#### Drag end while we are patching the virtual model

If a drag ends after a `Draggable` has been added or removed, but we have not finished collecting and patching the _virtual dimension model_ then we will delay the drop until the patch is finished. This is usually only a single frame. The `onDropEnd` callback will be called with a `DropResult` that is correct after the patch.

### Limitations

- The addition or removal of a `Draggable` cannot shift the placement of any other `Droppable` component.
- The change cannot impact the size of the `body`

## Unsupported: everything else

For the sake of clarity, here are some behaviors that are not supported during a drag:

- Changing the size of a `Draggable` during a drag
- Adding or removing a `Droppable`
- Adding or removing a `Draggable` to a `Droppable` that does not have a scroll container
- Adding or removing a `Draggable` to a `Droppable` that is not of the same `type` as the dragging item