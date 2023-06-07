# TodoTrek

A trello-like todo board which shows off different dynamic form strategies with Phoenix LiveView.

## Dynamic Forms

The logged-in home page is the main todo dashboard. It contains sortable Lists, which are stream-based and can be re-ordered. Within each list, Todos can be manged and re-ordered. Each todo is implemented as an individual form.

## Dynamic Nested Forms

The new List and edit List pages show examples of traditional nested forms with a dynamic `inputs_for` for List notifications. The notification entries can be prepended, appended, re-ordered, and deleted using regular checkboxes and Ecto's new `sort_param` and `drop_param` options to `cast_assoc` and `cast_embed`.

To start your Phoenix server:

  * Run `mix setup` to install and setup dependencies, and seed data
  * Start Phoenix endpoint with `mix phx.server` or inside IEx with `iex -S mix phx.server`
  * The default user is `user@example.com` with `password password` as the password.
  * Initial dummy data is seeded for the default user

Now you can visit [`localhost:4000`](http://localhost:4000) from your browser.

Ready to run in production? Please [check our deployment guides](https://hexdocs.pm/phoenix/deployment.html).

## Learn more

  * Official website: https://www.phoenixframework.org/
  * Guides: https://hexdocs.pm/phoenix/overview.html
  * Docs: https://hexdocs.pm/phoenix
  * Forum: https://elixirforum.com/c/phoenix-forum
  * Source: https://github.com/phoenixframework/phoenix


## Points of interest

### Control flow for updates
1. User requests the home page (where the todo lists are)
1. An `on_mount` hook attaches a "scope" to the socket (in this case it's just "who's logged in")
1. In `mount/3`, the home LiveView subscribes to a pubsub topic unique to the given scope
1. Mutation functions like `Todos.toggle_complete/2` accept the scope as an argument
1. When the mutation is completed successfully, that action broadcasts an event on the scope's topic
1. If the event pertains to a list, the home LiveView handles that event itself, but if it pertains to a todo inside one of the lists, it forwards the event on to the list component that contains that particular todo

This follows a pretty common high-level pattern, which is... when I need to make a mutation, I don't worry about also true-ing up my own state / display. Instead, I make that change, broadcast it to _everybody_, and I receive it and do the thing then, since I'm part of _everybody_.


### Interesting things
- The TodoListComponent initializes its stream of todos in an `update` call based on the `list` assign
- The stream is made up of todo _forms_ not todo structs or changesets
- Streams are accessed via the magic-ish `@streams` assign
- `stream_insert/4` will automatically update an item if it's already in the stream with matching ids
- `stream_insert/4` also lets you _move_ an existing item by passing one that's already there but with a `for:` option
- `stream_delete/3` similarly lets you remove a thing w/o messing with indexes
- The main `stream/4` function is also super flexible. You use it to initialize the list of items, and then you can automatically add more with another call (e.g. infinite scroll), replace them all with `reset: true`, or set a limit to e.g. only show the last 10 items as more and more are added


#### Dynamic inputs_for
- In `cast_assoc/3` and `cast_embed/3`, you can pass options for `:drop_param` and `:sort_param`. It's murky, but they essentially let you sidestep having to manipulate the changesets in very particular ways to remove and re-order things. Instead you can update these fields and Ecto will take them into account to remove / reorder the children on save.

This app uses this technique in the list form component, along with some client-side JS.


## More references
- [ElixirConf EU Keynote](https://www.youtube.com/watch?v=FADQAnq0RpA)
- [LiveView 0.19 release announcement](https://phoenixframework.org/blog/phoenix-liveview-0.19-released)
- [Stream docs](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#stream/4)
- [Sorting / dropping assoc / embeds](https://hexdocs.pm/ecto/Ecto.Changeset.html#cast_assoc/3-partial-changes-for-many-style-associations)
- [Chris McCord's todo repo](https://github.com/chrismccord/todo_trek)
- [My fork with these notes](https://github.com/grossvogel/todo_trek)
