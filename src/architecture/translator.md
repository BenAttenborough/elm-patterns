# Translator

As described in [Nested TEA](./nested-tea.html) it is sometimes useful to create a nested Elm architecture. When making this we normally would use `Html.map` to route messages back to the nested module.

For example, in the parent container you would have:

```haskell
Sub.view model.subViewModel |> Html.map SubMsg
```

The challenge of using `Html.map` here is that messages produced in the child module always need to map back to itself. We easily produce a message in the child module destined to its parent.

As an application grows it is common to encounter something like that, suddendly you need a UI element in the child module that needs to send the message to the parent.

## Pattern

The solution to this is to make the child module views generic, so instead of `View Msg` they become `View msg`. We then explicitly pass the constructor to route messages.

Say you start with a child module with a view like:

```haskell
view: Model -> Html Msg
view model =
  div []
  [ button [ onClick Clicked ] [ text model.name ]
  ]
```

We want to add another button, but this time it should send a message to the parent.

### Make the children generic

First we need to make this child view generic.

```haskell
view: Model -> (Msg -> msg) -> Html msg
view model toSelf =
  div []
  [ button [ onClick (toSelf Clicked) ] [ text model.name ]
  ]
```

To `toSelf` message is a constructor that wraps the internal message. This replaces `Html.map` in the parent module. Routing the message back to this module.

In the parent container we would call this like:

```haskell
type Msg = SubMsg Sub.Msg

view model =
  ...
  Sub.view model.subViewModel SubMsg
```

### Produce a parent message

With this setup we can produce parent messages from the child module.

```haskell
type alias Args msg =
  { toSelf : Msg -> msg
  , onSave: msg
  }

view: Model -> Args msg -> Html msg
view model args =
  div []
  [ button [ onClick (args.toSelf Clicked) ] [ "Send to self" ]
  , button [ onClick args.onSave ] [ text "Send to parent" ]
  ]
```

The parent would call this like:

```haskell
type Msg
  = OnSave
  | SubMsg Sub.Msg

view model =
  ...
  Sub.view
    model.subViewModel
    { toSelf = SubMsg
    , onSave = OnSave
    }
```